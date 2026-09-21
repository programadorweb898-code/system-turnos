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

## 19. Recursos públicos

La página pública y el widget necesitarán acceder a información pública del tenant.

Ejemplos:

- Nombre del negocio.
- Servicios activos.
- Profesionales públicos.
- Horarios disponibles.
- Configuración visual pública.

La API debe exponer únicamente la información necesaria.

Nunca debe devolver información administrativa o sensible a clientes públicos.

## 20. Idempotencia

Las operaciones críticas de creación de reservas deben considerar el riesgo de solicitudes duplicadas.

La estrategia concreta de idempotencia será definida durante la especificación de creación de reservas.

El objetivo es evitar que reintentos de red o solicitudes duplicadas generen reservas duplicadas.

## 21. Paginación

Los endpoints que puedan devolver grandes cantidades de recursos deberán definir una estrategia de paginación.

No se debe devolver una cantidad ilimitada de registros.

La estrategia concreta de paginación será establecida cuando se definan los endpoints administrativos correspondientes.

## 22. Versionado

La API comenzará utilizando:

`/api/v1`

Los cambios incompatibles deberán utilizar una estrategia de versionado o migración explícita.

No se deben introducir cambios incompatibles silenciosamente.

## 23. Compatibilidad frontend/backend

Antes de modificar un contrato utilizado por el frontend:

1. Identificar consumidores.
2. Actualizar la documentación.
3. Actualizar la especificación correspondiente.
4. Implementar el cambio.
5. Actualizar frontend y backend.
6. Ejecutar pruebas de integración.

## 24. Eventos

Los eventos internos relacionados con reservas utilizarán nombres consistentes.

Eventos iniciales previstos:

```
appointment.created
appointment.updated
appointment.cancelled
```

Estos eventos no constituyen automáticamente endpoints HTTP públicos.

Su implementación y transporte se definirá según la arquitectura y las especificaciones.

## 25. Seguridad de la API

La API debe contemplar:

- HTTPS en producción.
- Validación de entrada.
- Autenticación.
- Autorización.
- CORS configurado explícitamente.
- Rate limiting cuando corresponda.
- Protección contra inyección.
- No exposición de secretos.
- No exposición de información sensible en errores.
- Aislamiento entre tenants.

## 26. Reglas para los agentes

### Backend Agent

Debe:

- Implementar exactamente los contratos documentados.
- Actualizar el contrato cuando una especificación lo requiera.
- Mantener consistencia en errores.
- Validar las reglas en backend.

### Frontend Agent

Debe:

- Consumir únicamente contratos existentes.
- No inventar endpoints.
- No asumir estructuras de respuesta no documentadas.
- Manejar códigos de error.
- Adaptarse a cambios coordinados.

### Orchestrator Agent

Debe:

- Resolver inconsistencias entre agentes.
- Mantener actualizado este documento.
- Coordinar cambios incompatibles.
- Verificar que backend y frontend trabajen con el mismo contrato.

## 27. Contratos detallados

Este documento define reglas generales.

Los contratos detallados de cada funcionalidad deben establecerse en las especificaciones correspondientes.

Ejemplos futuros:

```
specs/
├── booking/
│   ├── availability.md
│   ├── create-appointment.md
│   ├── cancel-appointment.md
│   └── reschedule-appointment.md
├── tenants/
│   └── tenant-creation.md
└── widget/
    └── widget-integration.md
```

Cuando una especificación defina un contrato más específico, dicha definición será la referencia operativa para esa funcionalidad.

## 28. Regla principal

El contrato API es un acuerdo entre backend y frontend.

Ningún agente debe modificar unilateralmente el comportamiento esperado de una API compartida.

Los cambios deben ser explícitos, documentados, coordinados y probados.
