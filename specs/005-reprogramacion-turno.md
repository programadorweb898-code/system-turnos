# Especificación 005 — Reprogramación de turno

## 1. Objetivo

Definir cómo se cambia la fecha, hora o profesional de un turno existente sin romper las reglas de disponibilidad ni las garantías de concurrencia.

## 2. Contexto

La reprogramación no debe tratarse como una simple edición de campos.

Es una operación de dominio que combina conceptualmente:

- liberación del intervalo anterior;
- validación del nuevo intervalo;
- confirmación del nuevo intervalo;
- conservación de la identidad e historial del turno.

El cambio debe ejecutarse de forma atómica para evitar que un turno quede en un estado intermedio.

## 3. Actores

### Cliente

Puede reprogramar sus propios turnos cuando las reglas del negocio lo permitan.

### Administrador del tenant

Puede reprogramar turnos pertenecientes a su tenant según sus permisos.

## 4. Requisitos funcionales

### RF-001 — Reprogramar

El sistema debe permitir cambiar la fecha y/u hora de un turno válido.

### RF-002 — Cambiar profesional

El sistema podrá permitir cambiar el profesional cuando el servicio y las reglas del tenant lo permitan.

Si el profesional cambia, el nuevo profesional debe cumplir las mismas validaciones de disponibilidad y compatibilidad.

### RF-003 — Mantener el servicio

Por defecto, una reprogramación no cambia el servicio.

El cambio de servicio durante una reprogramación queda fuera del MVP y deberá definirse como operación separada si posteriormente se necesita.

### RF-004 — Recalcular finalización

La hora de finalización debe recalcularse a partir de la duración del servicio.

### RF-005 — Mantener identidad

Una reprogramación debe conservar el identificador lógico del turno, salvo que una decisión posterior establezca explícitamente otro modelo.

## 5. Flujo

El flujo conceptual es:

1. identificar el turno;
2. validar autenticación y autorización;
3. validar pertenencia al tenant;
4. validar que el turno sea reprogramable;
5. obtener el servicio vigente;
6. calcular el nuevo intervalo;
7. validar horario laboral;
8. validar profesional;
9. validar bloqueos;
10. comprobar conflictos en el nuevo intervalo;
11. ejecutar el cambio dentro de una operación transaccional;
12. confirmar el nuevo intervalo;
13. registrar el cambio;
14. generar el evento correspondiente.

## 6. Regla de disponibilidad

El nuevo intervalo debe cumplir todas las reglas que tendría una reserva nueva.

El turno que se está reprogramando no debe considerarse un conflicto consigo mismo.

Ejemplo:

Turno actual: 10:00–10:30
Nuevo horario: 11:00–11:30

El intervalo actual no debe impedir la propia reprogramación.

## 7. Concurrencia

La reprogramación debe estar protegida frente a operaciones simultáneas.

Ejemplo:

Cliente A intenta mover su turno a 11:00.
Cliente B intenta reservar 11:00 simultáneamente.

El sistema debe garantizar que no se confirmen dos intervalos incompatibles para el mismo profesional.

También debe contemplarse:

- dos solicitudes de reprogramación sobre el mismo turno;
- reprogramación simultánea con cancelación;
- reprogramación simultánea con una nueva reserva;
- reprogramación mientras un administrador modifica un bloqueo.

Las garantías deben mantenerse con múltiples instancias del backend.

## 8. Atomicidad

El sistema no debe dejar el turno sin horario válido entre la liberación del intervalo anterior y la confirmación del nuevo.

Conceptualmente:

BEGIN

validar estado actual
validar nuevo intervalo
aplicar cambio
registrar evento

COMMIT

Si alguna condición crítica falla, debe revertirse toda la operación.

## 9. Conflicto

Si el nuevo horario deja de estar disponible antes de confirmar, la reprogramación debe rechazarse.

Respuesta conceptual:

{
  "error": {
    "code": "APPOINTMENT_CONFLICT",
    "message": "El nuevo horario seleccionado ya no está disponible."
  }
}

Debe utilizarse HTTP 409 para conflictos de disponibilidad.

El turno original debe permanecer sin cambios cuando la reprogramación falle.

## 10. Idempotencia

La operación debe contemplar solicitudes repetidas para evitar resultados inconsistentes ante reintentos de red.

Si se utiliza una clave de idempotencia, la misma solicitud lógica debe producir un único resultado.

El mecanismo definitivo se documentará en el contrato de API.

## 11. Historial

La reprogramación debe conservar información suficiente para conocer el cambio realizado.

Como mínimo, cuando corresponda:

- fecha y hora anteriores;
- fecha y hora nuevas;
- profesional anterior;
- profesional nuevo;
- momento del cambio;
- actor que realizó la operación.

El nivel completo de auditoría podrá ampliarse posteriormente.

## 12. Eventos

Una reprogramación confirmada podrá producir:

`appointment.rescheduled`

Las notificaciones y automatizaciones externas deben ejecutarse fuera de la operación crítica.

## 13. API

El endpoint conceptual será:

`POST /api/v1/appointments/{appointmentId}/reschedule`

Se utiliza una operación explícita de dominio en lugar de permitir modificaciones arbitrarias mediante PATCH.

Esto centraliza las reglas de reprogramación.

## 14. Seguridad

El backend debe verificar:

- autenticación cuando corresponda;
- autorización;
- pertenencia del turno al tenant;
- permisos del actor;
- pertenencia del nuevo profesional al tenant;
- compatibilidad del profesional con el servicio;
- que no se expongan datos privados innecesarios.

## 15. Casos límite

Deben contemplarse al menos:

- turno inexistente;
- turno de otro tenant;
- turno cancelado;
- turno finalizado;
- turno no reprogramable por política temporal;
- nuevo horario inexistente;
- nuevo horario fuera del horario laboral;
- nuevo horario bloqueado;
- nuevo horario ocupado;
- nuevo profesional incompatible;
- nuevo profesional inactivo;
- dos reprogramaciones simultáneas;
- reprogramación simultánea con una nueva reserva;
- reprogramación simultánea con cancelación;
- reintento de la misma solicitud.

## 16. Criterios de aceptación

La especificación se considera implementada cuando:

1. un actor autorizado puede reprogramar un turno válido;
2. el turno conserva su identidad;
3. la duración se obtiene del servicio;
4. el nuevo intervalo se valida como una reserva nueva;
5. el propio turno actual no genera falso conflicto;
6. un horario ocupado provoca HTTP 409;
7. si falla la reprogramación, el turno original permanece intacto;
8. la operación es atómica;
9. se mantienen las garantías bajo concurrencia;
10. funciona correctamente con múltiples instancias;
11. se contempla idempotencia;
12. se registra información básica del cambio;
13. puede generarse `appointment.rescheduled`;
14. las automatizaciones externas no son necesarias para completar la operación;
15. existen pruebas unitarias, integración y concurrencia para las reglas principales.

## 17. Fuera de alcance

- cambio de servicio durante la reprogramación;
- pagos o diferencias de precio;
- reembolsos;
- lista de espera;
- reservas recurrentes;
- políticas comerciales avanzadas;
- notificaciones por canales específicos;
- auditoría avanzada.

## 18. Dependencias

- `specs/001-configuracion-tenant.md`
- `specs/002-disponibilidad.md`
- `specs/003-creacion-turno.md`
- `specs/004-cancelacion-turno.md`
- `SDD.md`
- `ARCHITECTURE.md`
- `API-CONTRACT.md`
- `DECISIONS.md`