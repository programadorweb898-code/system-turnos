# Especificación 003 — Creación de turno

## 1. Objetivo

Definir cómo el sistema confirma un turno y garantiza que dos solicitudes concurrentes no puedan reservar de forma válida el mismo recurso en un período incompatible.

## 2. Contexto

La disponibilidad consultada por el cliente no constituye una reserva. Entre la consulta y la confirmación puede ocurrir que otro cliente reserve el horario, un administrador lo bloquee o cambie la configuración.

Por este motivo, el backend debe volver a validar las condiciones relevantes al crear el turno.

## 3. Flujo de creación

1. Recibir la solicitud.
2. Autenticar cuando corresponda.
3. Determinar el tenant de forma segura.
4. Validar los datos de entrada.
5. Validar servicio y profesional.
6. Calcular la hora de finalización a partir de la duración del servicio.
7. Validar horario laboral y bloqueos.
8. Verificar conflictos con otros turnos.
9. Ejecutar la operación con un mecanismo transaccional apropiado.
10. Aplicar la protección contra concurrencia.
11. Persistir el turno.
12. Confirmar la operación.
13. Registrar o publicar el evento correspondiente cuando corresponda.

## 4. Regla principal de concurrencia

Si dos clientes intentan reservar simultáneamente un mismo recurso para períodos incompatibles, solo una solicitud puede confirmarse correctamente.

Ejemplo:

Cliente A → Juan → 10:00–10:30
Cliente B → Juan → 10:00–10:30

Resultado válido:

Cliente A → confirmado
Cliente B → conflicto

El sistema no debe depender del orden en que el frontend recibió o mostró la disponibilidad.

## 5. Regla de solapamiento

No debe permitirse crear dos turnos incompatibles para el mismo profesional.

Ejemplo:

Turno existente: 10:00–11:00

10:30–11:00 → rechazado
11:00–11:30 → permitido
09:30–10:00 → permitido
09:30–10:30 → rechazado

Los intervalos contiguos pueden coexistir; los intervalos que se intersectan no.

## 6. Duración

La hora de finalización no debe ser una fuente de verdad proporcionada libremente por el cliente.

Debe derivarse del servicio:

inicio + duración del servicio = fin

## 7. Validaciones críticas

Antes de confirmar deben comprobarse nuevamente:

- tenant activo;
- servicio activo y perteneciente al tenant;
- profesional activo y perteneciente al tenant, cuando corresponda;
- compatibilidad del profesional con el servicio;
- horario laboral;
- bloqueos;
- conflictos con otros turnos;
- zona horaria;
- reglas de reserva vigentes.

## 8. Protección contra concurrencia

La implementación debe utilizar mecanismos de PostgreSQL adecuados para garantizar integridad bajo concurrencia.

Se deben preferir garantías de integridad en la base de datos sobre mecanismos exclusivamente aplicados en memoria dentro de Node.js.

No se debe resolver el problema únicamente mediante variables globales, locks en memoria o comprobaciones previas sin protección transaccional.

Las garantías deben mantenerse aunque el backend funcione en múltiples instancias.

## 9. Restricción de integridad

La base de datos debe disponer de una estrategia que impida confirmar dos intervalos incompatibles para el mismo profesional.

La implementación concreta podrá utilizar mecanismos nativos de PostgreSQL, restricciones apropiadas, transacciones y/o bloqueos según el diseño final.

## 10. Transacción

La creación del turno debe formar parte de una operación atómica. Si una condición crítica falla, la operación debe abortarse sin dejar un turno parcialmente creado.

## 11. Conflicto

Cuando otro turno ya haya ocupado el intervalo, la solicitud debe rechazarse como conflicto de dominio.

Respuesta conceptual:

{
  "error": {
    "code": "APPOINTMENT_CONFLICT",
    "message": "El horario seleccionado ya no está disponible."
  }
}

El conflicto debe utilizar HTTP 409.

## 12. Idempotencia

La creación debe contemplar idempotencia para evitar que reintentos de una misma solicitud produzcan reservas duplicadas.

El mecanismo concreto, por ejemplo una clave de idempotencia, deberá definirse en el contrato definitivo de la API.

## 13. Eventos

Una creación confirmada podrá producir el evento de dominio appointment.created.

El procesamiento de notificaciones o automatizaciones no debe formar parte de la transacción crítica de reserva.

## 14. Casos de concurrencia que deben probarse

### Caso A — Mismo horario
Dos clientes intentan reservar Juan de 10:00 a 10:30. Resultado: una reserva confirmada y una rechazada por conflicto.

### Caso B — Solapamiento parcial
A: 10:00–11:00 y B: 10:30–11:30. Solo uno puede confirmarse.

### Caso C — Límites contiguos
A: 10:00–11:00 y B: 11:00–11:30. Ambos pueden confirmarse si el resto de las reglas lo permite.

### Caso D — Distinta duración
El conflicto debe determinarse por el intervalo real ocupado, no solamente por la hora de inicio.

### Caso E — Bloqueo concurrente
Un administrador bloquea un período mientras una reserva intenta utilizarlo. El resultado final no puede violar las reglas de disponibilidad.

### Caso F — Múltiples instancias
Las garantías deben mantenerse con dos o más instancias del backend procesando solicitudes simultáneamente.

## 15. Seguridad

El backend debe validar autenticación y autorización cuando corresponda, aislar el tenant, validar pertenencia de servicios y profesionales y no confiar exclusivamente en datos enviados por el cliente.

## 16. API

Endpoint conceptual:

POST /api/v1/appointments

La API debe utilizar JSON, los códigos HTTP definidos en API-CONTRACT.md, errores estructurados y un mecanismo de idempotencia cuando sea definido.

## 17. Criterios de aceptación

1. Se puede crear un turno válido.
2. La duración se obtiene del servicio.
3. Las reglas se vuelven a validar al crear.
4. No se permiten turnos incompatibles para el mismo profesional.
5. Dos solicitudes concurrentes para el mismo horario producen como máximo una reserva confirmada.
6. Se detectan solapamientos parciales.
7. Los intervalos contiguos permitidos pueden coexistir.
8. Las garantías funcionan con múltiples instancias.
9. La operación es atómica.
10. Los conflictos se devuelven como error de dominio HTTP 409.
11. Existe una estrategia de idempotencia.
12. Existen pruebas de concurrencia.
13. Un fallo no deja datos inconsistentes.
14. La creación confirmada no depende de que n8n esté disponible.

## 18. Fuera de alcance

- cancelación;
- reprogramación;
- pagos;
- autenticación detallada del cliente;
- límites de reservas por cliente;
- reservas recurrentes;
- lista de espera;
- recordatorios;
- políticas comerciales avanzadas;
- selección automática avanzada de profesionales.

## 19. Dependencias

- specs/001-configuracion-tenant.md
- specs/002-disponibilidad.md
- SDD.md
- ARCHITECTURE.md
- API-CONTRACT.md
- DECISIONS.md