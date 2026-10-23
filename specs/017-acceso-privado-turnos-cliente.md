# Especificación 017 — Acceso privado del cliente a su turno

## 1. Objetivo y alcance

Permitir que un cliente sin cuenta consulte, cancele o reprograme únicamente su propio turno. Esta especificación complementa 004, 005 y 009. No habilita esas rutas por sí sola: su implementación requiere backend, contrato, pruebas y una forma de entregar el acceso al cliente.

## 2. Decisión de acceso

El identificador UUID del turno, el nombre y el teléfono son datos de referencia, no credenciales. El acceso se concede por posesión de un token opaco y aleatorio, exclusivo de un turno. No se reutiliza el JWT administrativo.

El backend genera al confirmar la reserva 32 bytes con un generador criptográficamente seguro; codifica el token en base64url. Persiste solamente SHA-256(token), asociado a `tenant_id` y `appointment_id`, con índice único. La creación del turno y del hash debe ser atómica. El token en claro se entrega una sola vez en la respuesta exitosa de la reserva y, cuando exista el flujo de notificación, mediante un enlace privado al teléfono de confirmación. Nunca se incluye en eventos de dominio, logs, métricas, trazas o respuestas administrativas. Para poder reintentar la entrega por WhatsApp, el material del enlace podrá persistirse temporalmente cifrado en una cola transaccional de notificaciones; jamás en claro y con borrado al entregarse o vencer. La clave de cifrado vive fuera del repositorio y del payload de eventos.

Una reserva creada antes de introducir este mecanismo no obtiene acceso público automáticamente. No habrá un endpoint anónimo que permita emitir o recuperar tokens usando teléfono, nombre o UUID. Su gestión continuará a través del administrador hasta definir una recuperación segura.

## 3. Vigencia y revocación

El token se acepta hasta 24 horas después del inicio programado. Reprogramar actualiza la vigencia en la misma transacción que el turno. Cancelar reduce su vencimiento, si fuera posterior, a 24 horas después de cancelar. Esto conserva un breve plazo para consultar el resultado o repetir una cancelación sin duplicarla. El backend puede revocarlo por incidente; una revocación no altera el turno.

El vencimiento del token no concede permiso para cambiar turnos pasados: cancelar y reprogramar siguen sujetos a las reglas temporales y de estado de 004 y 005. Una política comercial de cancelación configurable queda fuera del MVP.

## 4. Entrega y uso

El cliente recibe `managementToken` únicamente en el `201` de creación y debe poder conservar un enlace privado para volver después. El flujo completo debe enviar el enlace al teléfono suministrado al reservar por WhatsApp y notificar también cada reprogramación y cancelación confirmada. Un fallo de entrega no revierte una reserva; tampoco debe inducir a emitir un segundo token por reintento.

El enlace del frontend llevará el token en el fragmento (`#token=...`), no en la ruta ni en el query string. El fragmento no viaja en la petición HTTP. El frontend lo lee, lo elimina de la barra de dirección y llama a la API con `Authorization: Bearer <managementToken>`. Debe usar HTTPS, `Referrer-Policy: no-referrer`, evitar analytics sobre esa pantalla, no guardarlo en `localStorage` y no incluirlo en mensajes de error. Si se pierde, no se puede recuperar con los datos de contacto.

El acceso por token es una credencial de capacidad: cualquiera que obtenga el enlace puede gestionar ese turno. Esta limitación debe explicarse al cliente; el frontend no debe tratar el enlace como público.

## 5. Operaciones públicas

Las rutas se definen en `API-CONTRACT.md`. Todas exigen token válido y correspondencia exacta entre token, tenant y turno. La consulta pública devuelve solo los datos necesarios para reconocer el turno: referencia, negocio, servicio, profesional, inicio, fin y estado. No devuelve teléfono, notas, otros turnos ni configuración administrativa.

La cancelación es idempotente mientras el token sea válido: un segundo intento devuelve el estado cancelado sin nuevo cambio ni evento. La reprogramación conserva ID y servicio; usa el mismo motor de validación que la reserva y la ruta administrativa: estado activo/futuro, anticipación mínima, horario, servicio/profesional habilitados, bloqueos, límite diario y exclusión de intervalos. Repetir la misma reprogramación debe ser un no-op. Los cambios y el historial se confirman en una transacción con los mismos bloqueos de concurrencia que reservas y bloqueos de agenda.

El historial debe distinguir `CLIENT` de `ADMIN`. Para un cambio del cliente, `actor_user_id` será nulo y se registrará el tipo de actor; no se inventará un usuario administrativo. El backend ajustará la migración de auditoría antes de habilitar estas rutas.

## 6. Errores y privacidad

Formato de error: `{ "error": { "code": "...", "message": "..." } }`.

- `400 INVALID_REQUEST`: formato de token, identificador o body inválido.
- `401 AUTHENTICATION_REQUIRED`: falta el bearer token.
- `404 APPOINTMENT_NOT_FOUND`: turno inexistente, token inválido, vencido, revocado o correspondiente a otro turno/tenant. La respuesta no debe revelar cuál de esas causas ocurrió.
- `409 APPOINTMENT_NOT_MODIFIABLE`: turno accesible, pero estado/fecha impiden modificarlo.
- `409 APPOINTMENT_CONFLICT`: nueva disponibilidad, profesional o regla de reserva no satisfecha.
- `429 RATE_LIMIT_EXCEEDED`: límite por IP en rutas de acceso público.

Las respuestas privadas, y la respuesta de creación que contiene `managementToken`, llevan `Cache-Control: no-store`. El backend valida la credencial antes de consultar o exponer datos del turno. Las comparaciones de hash deben evitar diferencias observables útiles. Se aplica un límite de solicitudes, adecuado para la experiencia pública, tanto a lectura como a cambios. Nunca se permite enumerar turnos por teléfono o nombre. No se registra el header `Authorization` ni el enlace.

## 7. Dependencias y secuencia de implementación

1. Actualizar el contrato compartido y la decisión arquitectónica.
2. Agregar migración del hash/vigencia del token y adaptar auditoría para actor cliente.
3. Crear el token atómicamente con la reserva y entregarlo una vez en `201`.
4. Implementar autorización por token, lectura mínima y cancelación/reprogramación usando reglas existentes.
5. Probar aislamiento, token inválido/vencido/revocado, filtrado de datos, idempotencia, disponibilidad y carreras.
6. Integrar frontend, cola de notificaciones y envío por WhatsApp para reserva, reprogramación y cancelación. La reserva y los cambios no deben depender de la disponibilidad del proveedor; si falla, el aviso queda pendiente y se reintenta.

No se considerará completo el acceso posterior del cliente hasta que exista al menos un método de entrega y conservación del enlace. No se desplegarán rutas públicas de modificación sin validar la seguridad de extremo a extremo.

## 8. Criterios de aceptación

- El ID, nombre o teléfono solos nunca autorizan operaciones.
- Un token permite acceder a un solo turno del tenant correcto.
- La base no contiene tokens en claro.
- Una reserva y su token se crean juntos o ninguno se crea.
- El cliente ve únicamente datos mínimos de su turno.
- Cancelar dos veces produce una sola transición e historial.
- Reprogramar respeta disponibilidad y conserva el turno original si falla.
- Carreras entre cliente, administrador, nueva reserva y bloqueo no producen estados inconsistentes.
- Token vencido o revocado no permite lectura ni cambios.
- Se registra el actor `CLIENT` sin simular un usuario administrativo.
- Los tres cambios de estado generan un aviso WhatsApp pendiente de manera atómica y sin duplicados por reintento.
- Un fallo del proveedor no revierte el turno y deja el aviso reintentable.
- Existen pruebas de integración y contrato para todas las condiciones anteriores.
