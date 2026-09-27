# API-CONTRACT.md — Contrato entre Frontend y Backend

## 1. Objetivo

Este documento define las reglas y contratos que utilizarán el frontend y el backend de System Turnos para comunicarse.

El objetivo es evitar que cada agente interprete de manera diferente los endpoints, datos, errores o reglas de comunicación.

Las definiciones detalladas de cada funcionalidad se establecerán en las especificaciones correspondientes dentro de `specs/`.

## 2. Principios

1. El backend es la fuente de verdad de las reglas de negocio.
2. El frontend consume la API y no implementa reglas críticas de negocio.
3. Los contratos deben ser explícitos.
4. Los cambios incompatibles deben documentarse y coordinarse.
5. Los errores deben tener una estructura consistente.
6. Las fechas y horarios deben manejarse explícitamente.
7. El contexto del tenant debe respetarse en todas las operaciones protegidas.

## 3. Tecnología

La API será REST sobre HTTP/HTTPS.

Tecnologías objetivo del backend:

- Node.js.
- TypeScript.
- Express.

El frontend utilizará:

- Next.js.
- TypeScript.

## 4. Prefijo de API

El backend utilizará inicialmente:

`/api/v1`

Ejemplos conceptuales:

`GET /api/v1/tenants/{tenantId}`

`GET /api/v1/availability`

Las rutas definitivas se establecerán en las especificaciones correspondientes.

## 5. Formato de datos

Las solicitudes y respuestas de la API utilizarán JSON cuando corresponda.

Header esperado:

`Content-Type: application/json`

Las respuestas deben mantener una estructura consistente.

## 6. Identificadores

Los recursos tendrán identificadores únicos.

El formato definitivo de los identificadores será establecido durante el diseño del backend y deberá mantenerse consistente entre API, base de datos y frontend.

El frontend no debe asumir un formato específico que no esté documentado.

## 7. Fechas y horarios

Las fechas y horarios son especialmente importantes para un sistema de reservas.

La API debe distinguir entre:

- Fecha.
- Hora local.
- Fecha y hora.
- Zona horaria.

El tenant tendrá una zona horaria explícita.

Para el contexto inicial:

`America/Argentina/Buenos_Aires`

No se deben enviar ni interpretar fechas de forma ambigua.

Las reglas definitivas de serialización y almacenamiento serán establecidas antes de implementar funcionalidades sensibles al tiempo.

## 8. Multi-tenancy

Las operaciones protegidas deben ejecutarse dentro del contexto de un tenant.

El frontend no debe poder seleccionar arbitrariamente un tenant para acceder a información a la que el usuario no tiene autorización.

El backend debe determinar y validar el tenant según el mecanismo de autenticación/autorización establecido.

Las APIs públicas que utilizan un slug u otro identificador público también deben validar que el recurso solicitado pertenezca al contexto correcto.

## 9. Autenticación

La autenticación administrativa será definida en una especificación específica.

Las rutas públicas de reserva no requerirán una cuenta de cliente en el MVP.

Las rutas administrativas requerirán autenticación.

La API debe diferenciar claramente:

- Recursos públicos.
- Recursos autenticados.
- Recursos autorizados para un tenant.

## 10. Autorización

Autenticación y autorización son conceptos separados.

Estar autenticado no implica tener acceso a todos los tenants o recursos.

El backend debe verificar permisos antes de permitir operaciones administrativas.

## 11. Respuestas HTTP

La API debe utilizar códigos HTTP semánticamente apropiados.

Como referencia:

- `200 OK`: operación exitosa.
- `201 Created`: recurso creado.
- `204 No Content`: operación exitosa sin contenido.
- `400 Bad Request`: solicitud inválida.
- `401 Unauthorized`: autenticación requerida o inválida.
- `403 Forbidden`: usuario autenticado sin permisos suficientes.
- `404 Not Found`: recurso inexistente o no accesible.
- `409 Conflict`: conflicto de estado o concurrencia.
- `422 Unprocessable Entity`: datos válidos sintácticamente pero inválidos según reglas de negocio, cuando corresponda.
- `429 Too Many Requests`: límite de solicitudes excedido.
- `500 Internal Server Error`: error inesperado del servidor.

Las especificaciones concretas podrán definir excepciones justificadas.

## 12. Estructura de errores

Las respuestas de error deben utilizar una estructura consistente.

Formato conceptual:

```json
{
  "error": {
    "code": "APPOINTMENT_CONFLICT",
    "message": "El horario seleccionado ya no está disponible."
  }
}
```

Los códigos de error deben ser estables para permitir que el frontend pueda reaccionar correctamente.

El texto de `message` está destinado a proporcionar información legible y no debe utilizarse como identificador lógico.

## 13. Validación

La validación debe realizarse en el backend aunque el frontend también valide los datos.

El frontend puede mejorar la experiencia de usuario mediante validación anticipada.

El backend debe considerar todas las solicitudes externas como no confiables.

## 14. Disponibilidad

La consulta de disponibilidad debe permitir obtener horarios posibles para un contexto determinado.

Conceptualmente:

```
GET /api/v1/availability
```

Los parámetros definitivos serán establecidos por la especificación de disponibilidad.

La consulta deberá contemplar, cuando corresponda:

- Tenant.
- Servicio.
- Profesional.
- Fecha.
- Zona horaria.
- Reglas de disponibilidad.

La respuesta debe indicar claramente los horarios disponibles y la información necesaria para mostrarlos.

## 15. Reserva

La creación de una reserva será responsabilidad del backend.

Conceptualmente:

```
POST /api/v1/appointments
```

La solicitud deberá contener la información necesaria para identificar:

- Tenant.
- Servicio.
- Profesional cuando corresponda.
- Fecha/hora solicitada.
- Datos requeridos del cliente.

El backend debe volver a comprobar la disponibilidad antes de persistir la reserva.

La disponibilidad obtenida previamente por el frontend no constituye una garantía.

## 16. Concurrencia

Una reserva puede entrar en conflicto con otra reserva creada simultáneamente.

En ese caso, el backend debe rechazar la operación de forma controlada.

El conflicto debe representarse mediante un error apropiado, inicialmente `409 Conflict` cuando corresponda.

El frontend debe poder informar al usuario que el horario dejó de estar disponible y permitirle seleccionar otro.

## 17. Cancelación

Conceptualmente:

```
POST /api/v1/appointments/{appointmentId}/cancel
```

La ruta definitiva y las reglas de cancelación serán definidas en la especificación correspondiente.

El backend debe validar que:

- La reserva existe.
- La operación está permitida.
- El actor tiene autorización.
- Se cumplen las reglas de cancelación.

## 18. Modificación y reprogramación

La modificación o reprogramación de una reserva debe considerarse una operación de negocio y no simplemente una edición arbitraria de campos.

Debe volver a validarse:

- Disponibilidad.
- Duración.
- Profesional.
- Reglas del negocio.
- Permisos.
- Concurrencia.

Las rutas definitivas se establecerán en la especificación correspondiente.

## 19. Recursos administrativos

El administrador autenticado puede consultar la configuración del tenant asociado a su identidad mediante:

```http
GET /api/v1/admin/tenant
```

La ruta no recibe `tenant_id`. El backend obtiene el tenant desde el contexto autenticado y no debe permitir que el cliente sustituya ese contexto mediante parámetros o campos de la solicitud.

La respuesta incluye únicamente la configuración necesaria para la administración del tenant:

- `id`
- `name`
- `slug`
- `timezone`
- `status`
- `maxDailyAppointments`
- `minimumBookingNoticeHours`

Una solicitud sin autenticación debe recibir `401 Unauthorized`.

## 20. Configuración administrativa de servicios

Los servicios del tenant autenticado se administran mediante:

```http
GET /api/v1/admin/configuration/services
POST /api/v1/admin/configuration/services
```

Ambas rutas requieren autenticación administrativa.

El backend obtiene el `tenant_id` exclusivamente desde el contexto autenticado. El cliente no puede seleccionar otro tenant mediante el body, query string o parámetros de la ruta.

### GET /api/v1/admin/configuration/services

Devuelve los servicios pertenecientes al tenant autenticado.

Cada servicio incluye:

- `id`
- `name`
- `description`
- `duration`
- `status`

Una solicitud sin autenticación debe recibir `401 Unauthorized`.

### POST /api/v1/admin/configuration/services

Crea un servicio para el tenant autenticado.

Request:

```json
{
  "name": "Corte",
  "description": "Corte clásico",
  "duration": 30
}
```

`description` es opcional.

`duration` debe ser un entero mayor que cero.

Una creación exitosa devuelve `201 Created` con el servicio creado.

Datos inválidos deben devolver `400 Bad Request`.

El campo `tenantId`, si fuera enviado por el cliente, debe ser ignorado y nunca utilizarse para seleccionar el tenant de autorización.

## 21. Recursos públicos

## 21. Configuración administrativa de profesionales

Los profesionales del tenant autenticado se administran mediante:

```http
GET /api/v1/admin/configuration/professionals
POST /api/v1/admin/configuration/professionals
PATCH /api/v1/admin/configuration/professionals/:id/status
```

Todas las rutas requieren autenticación administrativa.

El backend obtiene el `tenant_id` exclusivamente desde el contexto autenticado. El cliente no puede seleccionar otro tenant mediante body, query string o parámetros de ruta.

### GET /api/v1/admin/configuration/professionals

Devuelve los profesionales pertenecientes al tenant autenticado.

Cada profesional incluye:

- `id`
- `name`
- `status`

Respuesta exitosa:

`200 OK`

### POST /api/v1/admin/configuration/professionals

Crea un profesional para el tenant autenticado.

Request:

```json
{
  "name": "Juan"
}
```

`name` es obligatorio y debe contener al menos un carácter después de eliminar espacios externos.

El profesional se crea inicialmente con estado `active`.

Una creación exitosa devuelve:

`201 Created`

Datos inválidos deben devolver `400 Bad Request`.

Si el cliente envía `tenantId`, ese valor debe ignorarse y nunca utilizarse para seleccionar el tenant de autorización.

### PATCH /api/v1/admin/configuration/professionals/:id/status

Modifica el estado de un profesional perteneciente al tenant autenticado.

Request:

```json
{
  "status": "inactive"
}
```

Valores permitidos:

- `active`
- `inactive`

Una modificación exitosa devuelve `200 OK`.

Si el profesional no existe dentro del tenant autenticado, devuelve `404 Not Found`.

Un estado inválido devuelve `400 Bad Request`.

Un profesional inactivo no debe considerarse disponible para nuevas reservas.


## 22. Configuración administrativa de horarios de atención

Los horarios de atención del tenant autenticado se administran mediante:

```http
GET /api/v1/admin/configuration/business-hours
POST /api/v1/admin/configuration/business-hours
```

Ambas rutas requieren autenticación administrativa.

En el MVP los horarios aplican al tenant en su conjunto. No se definen todavía horarios individuales por profesional.

El backend obtiene el `tenant_id` exclusivamente desde el contexto autenticado. El cliente no puede seleccionar otro tenant mediante el body, query string o parámetros de la ruta.

### GET /api/v1/admin/configuration/business-hours

Devuelve los horarios pertenecientes al tenant autenticado.

Cada horario incluye:

- `id`
- `dayOfWeek`
- `startTime`
- `endTime`

La respuesta es `200 OK` y los resultados se ordenan por día y hora de inicio.

### POST /api/v1/admin/configuration/business-hours

Crea un intervalo de atención para el tenant autenticado.

Request:

```json
{
  "dayOfWeek": 1,
  "startTime": "09:00",
  "endTime": "17:00"
}
```

`dayOfWeek` debe estar entre 0 y 6.

`startTime` y `endTime` deben utilizar formato `HH:mm`, y la hora de finalización debe ser posterior a la de inicio.

Una creación exitosa devuelve `201 Created`.

Datos inválidos deben devolver `400 Bad Request`.

Si el cliente envía `tenantId`, ese valor debe ignorarse y nunca utilizarse para seleccionar el tenant de autorización.
