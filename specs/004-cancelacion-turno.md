# Especificación 004 — Cancelación de turno

## 1. Objetivo

Definir cómo se cancela un turno existente, quién puede hacerlo y qué condiciones deben cumplirse para mantener la integridad del sistema.

## 2. Contexto

La cancelación modifica el estado de un turno previamente creado.

La operación no debe eliminar físicamente el registro, porque el historial del turno puede ser necesario para auditoría, métricas, trazabilidad y futuras funcionalidades.

## 3. Actores

### Cliente

Puede cancelar un turno cuando las reglas del negocio lo permitan.

### Administrador del tenant

Puede cancelar turnos pertenecientes a su tenant de acuerdo con sus permisos.

## 4. Estados

Como mínimo, el dominio debe distinguir un turno activo de uno cancelado.

Los estados definitivos del ciclo de vida podrán ampliarse posteriormente, por ejemplo para diferenciar completado, no asistió o pendiente.

Un turno cancelado no debe volver a considerarse una reserva activa para el cálculo de disponibilidad.

## 5. Requisitos funcionales

### RF-001 — Cancelar turno

El sistema debe permitir cancelar un turno existente cuando el actor tenga autorización y se cumplan las reglas de cancelación.

### RF-002 — No eliminar físicamente

La cancelación debe conservar el registro del turno.

### RF-003 — Motivo opcional

El sistema podrá registrar un motivo de cancelación.

El modelo definitivo del motivo se definirá durante el diseño del backend.

### RF-004 — Fecha de cancelación

Debe poder registrarse cuándo se produjo la cancelación.

### RF-005 — Actor de cancelación

Cuando corresponda, debe poder identificarse quién realizó la cancelación.

## 6. Reglas de negocio

### RN-001 — Pertenencia al tenant

Un usuario administrativo solo puede cancelar turnos pertenecientes a su tenant.

### RN-002 — Turno inexistente

Si el turno no existe, debe devolverse un error apropiado.

### RN-003 — Turno ya cancelado

Cancelar nuevamente un turno ya cancelado no debe generar una segunda transición de estado ni efectos duplicados.

La operación puede tratarse como idempotente según el contrato definitivo.

### RN-004 — Turno no cancelable

Un turno que ya haya finalizado o se encuentre en un estado que no permita cancelación debe ser rechazado.

Los estados exactos y las reglas temporales deberán quedar definidos antes de implementar estados adicionales.

### RN-005 — Política de anticipación

El sistema deberá contemplar una política que determine hasta cuánto tiempo antes del turno puede cancelarse.

Para el MVP, si todavía no existe una política comercial configurable, el backend debe mantener esta regla en un punto centralizado y documentado en lugar de dispersarla por la aplicación.

### RN-006 — Disponibilidad

Una vez cancelado el turno, el intervalo podrá volver a estar disponible si no existe otra restricción.

El Booking Engine debe considerar el nuevo estado en consultas posteriores.

## 7. Concurrencia

La cancelación también debe ser segura frente a solicitudes concurrentes.

Ejemplo:

Cliente A cancela un turno mientras un administrador intenta cancelarlo al mismo tiempo.

El resultado final debe ser consistente y no producir efectos duplicados.

También debe contemplarse la carrera entre cancelación y otras operaciones del turno, como reprogramación.

La implementación debe utilizar transacciones y condiciones de actualización apropiadas cuando sea necesario.

## 8. Eventos

Una cancelación confirmada podrá producir un evento:

`appointment.cancelled`

Las notificaciones, mensajes y automatizaciones deben ejecutarse fuera de la operación crítica de cancelación.

Si se utiliza Outbox, el evento y el cambio de estado deben mantener la atomicidad definida por el sistema.

## 9. API

El contrato conceptual inicial será:

`POST /api/v1/appointments/{appointmentId}/cancel`

Se utiliza una operación explícita de dominio en lugar de permitir que el cliente modifique libremente el campo de estado mediante un PATCH genérico.

Esto permite centralizar las reglas de cancelación.

## 10. Respuestas

### Cancelación exitosa

Debe devolverse el turno actualizado o una representación equivalente definida por el contrato.

### Turno inexistente

HTTP 404.

### Sin autorización

HTTP 403.

### Turno no cancelable

Debe utilizarse el código HTTP correspondiente definido en `API-CONTRACT.md`, según la causa concreta.

### Conflicto de estado

Si la operación entra en conflicto con una modificación concurrente, debe devolverse un error de dominio consistente.

## 11. Seguridad

El backend debe verificar:

- autenticación cuando corresponda;
- autorización;
- pertenencia del turno al tenant;
- permisos del actor;
- que no se pueda cancelar un turno perteneciente a otro tenant;
- que los datos privados del cliente no se expongan innecesariamente.

El identificador del turno por sí solo nunca debe ser suficiente para saltarse el aislamiento multi-tenant.

## 12. Auditoría

El sistema debe conservar información suficiente para saber que el turno fue cancelado.

Como mínimo debe poder determinarse:

- turno afectado;
- momento de cancelación;
- actor cuando corresponda;
- estado anterior;
- nuevo estado.

El nivel completo de auditoría se podrá ampliar posteriormente.

## 13. Casos límite

Deben contemplarse al menos:

- turno inexistente;
- turno de otro tenant;
- turno ya cancelado;
- turno ya finalizado;
- turno que no permite cancelación por política temporal;
- cancelación simultánea por dos actores;
- cancelación simultánea con reprogramación;
- cancelación de un turno cuya hora ya está muy próxima;
- cliente intentando cancelar un turno que no le pertenece;
- administrador sin permisos suficientes;
- tenant inactivo.

## 14. Criterios de aceptación

La especificación se considera implementada cuando:

1. un actor autorizado puede cancelar un turno válido;
2. el turno no se elimina físicamente;
3. la cancelación cambia el estado de forma consistente;
4. un turno cancelado deja de bloquear disponibilidad;
5. un turno de otro tenant no puede cancelarse;
6. un actor sin autorización no puede cancelarlo;
7. repetir la cancelación no produce efectos duplicados;
8. la operación es segura frente a concurrencia;
9. se registra la información básica de cancelación;
10. la cancelación confirmada puede generar `appointment.cancelled`;
11. las automatizaciones externas no son necesarias para completar la cancelación;
12. existen pruebas unitarias e integración para las reglas principales.

## 15. Fuera de alcance

- reprogramación;
- lista de espera;
- reembolsos;
- pagos;
- políticas comerciales avanzadas por tenant;
- notificaciones específicas por canal;
- auditoría avanzada;
- historial de cambios completo.

## 16. Dependencias

- `specs/001-configuracion-tenant.md`
- `specs/002-disponibilidad.md`
- `specs/003-creacion-turno.md`
- `SDD.md`
- `ARCHITECTURE.md`
- `API-CONTRACT.md`
- `DECISIONS.md`