# Especificación 013 — Asignación de servicios a profesionales

## 1. Objetivo

Definir cómo un tenant determina qué profesionales están habilitados para prestar cada servicio.

La relación debe permitir que un servicio sea atendido por uno o varios profesionales y que un profesional pueda atender uno o varios servicios.

Esta relación es necesaria para calcular correctamente la disponibilidad y para permitir que el cliente elija un profesional cuando exista más de una opción.

## 2. Contexto

System Turnos es un sistema multi-tenant y no está limitado a peluquerías.

Un tenant puede representar, por ejemplo:
- una peluquería;
- una clínica;
- un centro médico;
- un centro de estética;
- otro negocio basado en reservas.

Por lo tanto, el modelo utiliza los conceptos genéricos de Service y Professional.

La relación entre ambos recursos es muchos a muchos.

Conceptualmente:

Professional
      ↕
ProfessionalService
      ↕
Service

## 3. Actor

### Administrador

Puede:
- consultar los servicios asignados a un profesional;
- asignar servicios a un profesional;
- reemplazar las asignaciones de servicios de un profesional.

Todas las operaciones requieren autenticación y autorización administrativa.

## 4. Modelo de datos

Se debe incorporar una relación persistente equivalente a:

### ProfessionalService

- professionalId
- serviceId

La combinación debe ser única.

Ambos recursos deben pertenecer al mismo tenant.

La implementación concreta puede utilizar otro nombre interno, pero debe representar esta relación muchos a muchos.

## 5. Reglas de negocio

### RN-01 — Asignación explícita

Un profesional solo puede atender un servicio si existe una asignación entre ambos.

No se debe asumir que todos los profesionales pueden realizar todos los servicios.

### RN-02 — Mismo tenant

No se puede crear una relación entre un profesional y un servicio pertenecientes a tenants diferentes.

### RN-03 — Profesional activo

Un profesional inactivo no puede utilizarse para nuevas reservas aunque conserve sus asignaciones históricas.

### RN-04 — Servicio activo

Un servicio inactivo no puede utilizarse para nuevas reservas aunque conserve sus asignaciones.

### RN-05 — Sin duplicados

No puede existir más de una asignación para la misma combinación profesional-servicio.

### RN-06 — Sin eliminación histórica

Desactivar un profesional o servicio no requiere eliminar las asignaciones existentes.

### RN-07 — Cantidad de profesionales

El sistema no debe almacenar un campo redundante como employeeCount para representar la cantidad de profesionales.

La cantidad se obtiene de los profesionales registrados.

## 6. Selección del profesional por parte del cliente

La selección del profesional es opcional.

### Caso A — Un solo profesional habilitado

Si existe exactamente un profesional activo asignado al servicio:
- ese profesional queda determinado automáticamente;
- el cliente no debe poder cambiarlo;
- no se debe mostrar una opción modificable de Cualquier profesional.

### Caso B — Más de un profesional habilitado

Si existen dos o más profesionales activos asignados al servicio:
- el cliente puede seleccionar un profesional específico;
- también debe existir la opción Cualquier profesional;
- Cualquier profesional es la opción seleccionada por defecto.

### Caso C — Ningún profesional habilitado

El servicio no puede ofrecerse para nuevas reservas.

## 7. Regla de disponibilidad según la selección

La selección del profesional forma parte del contexto de disponibilidad.

### Cualquier profesional

Un horario es disponible cuando existe al menos un profesional activo y habilitado para el servicio que pueda atender el intervalo completo.

### Profesional específico

Un horario es disponible únicamente si ese profesional puede atender el intervalo completo.

La disponibilidad de otros profesionales no debe influir en ese resultado.

## 8. Recalculo de fechas y horarios

La disponibilidad debe recalcularse cuando cambie cualquiera de estos datos:
- servicio;
- profesional seleccionado;
- fecha.

El frontend no debe reutilizar horarios obtenidos para una selección diferente.

El backend debe volver a validar todas las condiciones al crear el turno.

## 9. Reserva con Cualquier profesional

Cuando el cliente selecciona Cualquier profesional, el cliente no determina qué profesional recibirá el turno.

Durante la creación, el backend debe:
1. verificar que el servicio esté activo;
2. obtener los profesionales activos habilitados para el servicio;
3. determinar cuáles pueden atender el intervalo solicitado;
4. seleccionar uno de los profesionales elegibles;
5. asociar ese profesional al turno;
6. realizar la operación bajo las garantías de concurrencia correspondientes.

La reserva confirmada debe almacenar el profesional finalmente asignado.

Si ningún profesional puede atender el intervalo al momento de confirmar, la reserva debe rechazarse con 409 Conflict.

## 10. Reserva con profesional específico

El backend debe verificar nuevamente que:
- pertenece al tenant;
- está activo;
- está asignado al servicio;
- puede atender el intervalo;
- no existe un conflicto;
- se cumplen las demás reglas de reserva.

## 11. API administrativa

Las rutas definitivas serán:

GET /api/v1/admin/configuration/professionals/:professionalId/services

PUT /api/v1/admin/configuration/professionals/:professionalId/services

Ambas requieren autenticación administrativa.

GET devuelve los servicios asignados al profesional dentro del tenant autenticado.

PUT reemplaza el conjunto de servicios asignados al profesional.

Request conceptual:

{
  "serviceIds": ["service-uuid-1", "service-uuid-2"]
}

El backend debe verificar el profesional, los servicios, el tenant y los IDs, evitando duplicados y realizando el reemplazo de forma atómica.

## 12. API pública de disponibilidad

La consulta debe permitir indicar opcionalmente un profesional.

GET /api/v1/availability?serviceId=<id>&date=YYYY-MM-DD

GET /api/v1/availability?serviceId=<id>&professionalId=<id>&date=YYYY-MM-DD

Cuando no se envía professionalId, se interpreta como Cualquier profesional.

Si existe un único profesional habilitado, el contexto queda determinado por ese profesional. Si existen varios, el frontend puede ofrecer Cualquier profesional y los profesionales específicos.

## 13. API de reserva

La creación de un turno deberá aceptar un profesional opcional.

professionalId puede omitirse para representar Cualquier profesional.

Cuando se omite, el backend debe seleccionar un profesional elegible durante la creación.

Cuando se informa, el backend debe validar que el profesional sea elegible para el servicio y el intervalo solicitado.

## 14. Seguridad y aislamiento

Todas las operaciones administrativas deben utilizar el tenant del contexto autenticado.

Nunca debe utilizarse un tenantId enviado por el cliente para seleccionar el tenant.

En operaciones públicas, el backend debe comprobar que el servicio, profesional y relación profesional-servicio pertenezcan al mismo contexto.

## 15. Casos límite

Deben contemplarse:
- profesional inexistente;
- servicio inexistente;
- profesional de otro tenant;
- servicio de otro tenant;
- profesional inactivo;
- servicio inactivo;
- profesional sin servicios asignados;
- servicio sin profesionales asignados;
- asignación duplicada;
- selección de profesional no asignado al servicio;
- selección de profesional inactivo;
- un único profesional habilitado;
- varios profesionales habilitados;
- ningún profesional habilitado;
- disponibilidad diferente entre profesionales;
- cambio de Cualquier profesional a un profesional específico;
- cambio de profesional específico a Cualquier profesional;
- profesional elegido que deja de estar disponible antes de confirmar;
- concurrencia entre reservas para diferentes profesionales;
- concurrencia entre reservas cuando se selecciona Cualquier profesional.

## 16. Criterios de aceptación

1. Un servicio puede asignarse a varios profesionales.
2. Un profesional puede realizar varios servicios.
3. No existen asignaciones duplicadas.
4. No se permiten relaciones entre recursos de tenants diferentes.
5. Los profesionales inactivos no aparecen como opciones para nuevas reservas.
6. Los servicios inactivos no pueden reservarse.
7. Un servicio sin profesionales activos asignados no puede reservarse.
8. Con un único profesional habilitado, la selección queda determinada automáticamente.
9. Con varios profesionales habilitados, Cualquier profesional aparece y es la opción por defecto.
10. Al seleccionar un profesional específico, la disponibilidad se calcula únicamente para ese profesional.
11. Al seleccionar Cualquier profesional, un horario aparece si al menos un profesional elegible puede atenderlo.
12. Cambiar profesional o servicio provoca una nueva consulta o recalculación de disponibilidad.
13. Una reserva con Cualquier profesional termina asociada a un profesional concreto.
14. Una reserva con profesional específico valida nuevamente su elegibilidad.
15. La creación mantiene las garantías de concurrencia definidas en la especificación de creación de turnos.
16. El frontend no puede utilizar tenantId para acceder o modificar asignaciones de otro tenant.

## 17. Fuera de alcance

- horarios individuales por profesional;
- vacaciones individuales;
- selección avanzada o balanceo automático de profesionales;
- preferencias del cliente por profesional;
- límites de turnos por profesional;
- cuentas de usuario para profesionales;
- pagos;
- notificaciones;
- lista de espera.

## 18. Dependencias

- SDD.md
- AGENTS.md
- ARCHITECTURE.md
- API-CONTRACT.md
- DECISIONS.md
- specs/001-configuracion-tenant.md
- specs/002-disponibilidad.md
- specs/003-creacion-turno.md
- specs/011-configuracion-profesionales.md