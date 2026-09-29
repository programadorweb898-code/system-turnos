# Especificación 016 — Bloqueos de agenda

## 1. Objetivo

Definir cómo el administrador bloquea períodos de la agenda y cómo se relaciona un bloqueo con los turnos ya confirmados.

Esta especificación cierra un hueco: los bloqueos se usan en el cálculo de disponibilidad desde `specs/002-disponibilidad.md`, pero no existía una API administrativa para gestionarlos.

## 2. Actor

### Administrador del tenant

Puede consultar y crear bloqueos del tenant autenticado.

Todas las operaciones requieren autenticación y aislamiento por tenant.

## 3. Contexto

Un bloqueo excluye un período de la disponibilidad. La creación de un bloqueo puede chocar con turnos que los clientes ya confirmaron, y ese choque necesita una regla definida.

## 4. Modelo de datos

Se utilizará la entidad existente `BlockedTime`:

- `id`
- `tenantId`
- `professionalId` (opcional; `null` significa que el bloqueo aplica a todo el tenant)
- `startsAt`
- `endsAt`
- `reason` (opcional)
- `createdAt`
- `updatedAt`

## 5. Reglas de negocio

### RN-001 — Origen del tenant

El `tenantId` se obtiene del contexto autenticado y nunca del body. Si el cliente envía `tenantId`, debe ignorarse.

### RN-002 — Intervalo válido

`startsAt` debe ser anterior a `endsAt`. Un intervalo vacío o invertido se rechaza.

### RN-003 — Interpretación temporal

Los instantes se almacenan y comparan en UTC. El cliente envía instantes completos con zona horaria explícita.

### RN-004 — Alcance del bloqueo

Si `professionalId` es `null`, el bloqueo aplica a todos los profesionales del tenant. Si se informa, aplica solo a ese profesional.

### RN-005 — Conflicto con turnos existentes

Al crear un bloqueo, el backend debe verificar si existen turnos en estado `PENDING` o `CONFIRMED` que se superpongan con el intervalo.

Si existen superposiciones, el backend **no crea el bloqueo** y responde `409 Conflict`.

La verificación se realiza siempre, tanto para bloqueos de tenant completo como para bloqueos dirigidos a un profesional. En el segundo caso solo se consideran los turnos de ese profesional.

Motivar la verificación en ambos casos: si el administrador no fuera avisado al crear un bloqueo sobre un turno ya confirmado, el bloqueo ocultaría un turno existente en el panel y en la disponibilidad, y el conflicto aparecería más tarde y en un lugar menos útil.

Los intervalos se comparan como semiabiertos `[inicio, fin)`. Por lo tanto, un turno que termina exactamente al inicio del bloqueo, o que empieza exactamente al final, **no** constituye superposición.

### RN-006 — El turno confirmado no se modifica

Cuando existe un conflicto, el backend nunca cancela, reprograma ni altera el turno existente. El turno confirmado es el compromiso del sistema con el cliente y no puede romperse por una acción administrativa.

La resolución del conflicto corresponde al administrador, que debe decidir si reprograma o cancela el turno por los canales habituales.

### RN-007 — Motivo opcional

`reason` es texto libre y opcional. Si se informa, se persiste recortado.

## 6. API

### GET /api/v1/admin/configuration/blocked-times

Devuelve los bloqueos del tenant autenticado.

Respuesta exitosa: `200 OK`.

Cada elemento contiene:

- `id`
- `professionalId` (puede ser `null`)
- `startsAt`
- `endsAt`
- `reason` (puede ser `null`)

Los bloqueos se devuelven ordenados por `startsAt` ascendente.

### POST /api/v1/admin/configuration/blocked-times

Crea un bloqueo para el tenant autenticado.

Request:

```json
{
  "startsAt": "2026-03-10T14:00:00.000Z",
  "endsAt": "2026-03-10T16:00:00.000Z",
  "reason": "Capacitación"
}
```

El campo `professionalId` es opcional. Si se omite, el bloqueo aplica a todo el tenant.

Respuesta exitosa: `201 Created`.

Datos inválidos: `400 Bad Request`.

### 6.1 Respuesta ante conflicto

Cuando existen turnos que se superponen con el intervalo solicitado, el backend responde `409 Conflict` con el código `BLOCKED_TIME_CONFLICTS` y un campo `details` con una entrada por cada turno en conflicto.

```json
{
  "error": {
    "code": "BLOCKED_TIME_CONFLICTS",
    "message": "No se puede crear el bloqueo porque hay turnos confirmados en ese intervalo.",
    "details": [
      "Turno del 2026-03-10T14:30:00.000Z al 2026-03-10T15:00:00.000Z (Ana Gómez)."
    ]
  }
}
```

El campo `details` es opcional según `API-CONTRACT.md` y aquí cumple la función de indicar qué condiciones concretas impiden completar la operación.

## 7. Seguridad

- Todas las operaciones requieren autenticación.
- El `tenantId` se toma del contexto autenticado.
- Un administrador no puede leer ni crear bloqueos de otro tenant.
- Un `professionalId` informado debe pertenecer al tenant autenticado; en caso contrario se responde `404 Not Found`.
- No se registran datos personales de clientes en los logs.

## 8. Errores

- `400 Bad Request`: intervalo inválido, fechas mal formadas o `reason` demasiado largo.
- `401 Unauthorized`: falta autenticación.
- `404 Not Found`: el `professionalId` informado no pertenece al tenant.
- `409 Conflict`: existen turnos que se superponen con el bloqueo solicitado.

## 9. Casos límite

- `startsAt` igual a `endsAt`;
- `endsAt` anterior a `startsAt`;
- fechas que no son instantes válidos;
- bloqueo que coincide exactamente con los bordes de un turno existente (no debe considerarse superposición, porque los intervalos son semiabiertos `[inicio, fin)`);
- bloqueo sin `reason`;
- bloqueo con `professionalId` de otro tenant;
- bloqueo que cubre varios turnos;
- bloqueo cuyo intervalo contiene únicamente turnos cancelados o completados (no debe entrar en conflicto);
- solicitud sin autenticación;
- intento de crear un bloqueo para otro tenant mediante valores enviados por el cliente.

## 10. Criterios de aceptación

1. El administrador puede consultar sus bloqueos.
2. El administrador puede crear un bloqueo válido.
3. El backend valida que el intervalo sea correcto.
4. El backend utiliza el tenant del contexto autenticado.
5. Un `tenantId` enviado por el cliente no cambia el tenant afectado.
6. Crear un bloqueo que se superpone con turnos `PENDING` o `CONFIRMED` devuelve `409` con `BLOCKED_TIME_CONFLICTS`.
7. El intento fallido no crea el bloqueo.
8. El turno en conflicto no se modifica en ningún caso.
9. Los turnos `CANCELLED` y `COMPLETED` no generan conflicto.
10. Existen tests para validación, aislamiento, conflictos y endpoints principales.

## 11. Pruebas requeridas

- validación de intervalo y formato de fechas;
- creación correcta con y sin `reason`;
- aislamiento por tenant;
- rechazo de `professionalId` ajeno al tenant;
- superposición con turno confirmado devuelve `409` y `details` con una entrada por conflicto;
- superposición con turno cancelado no devuelve `409`;
- los turnos no se modifican tras un `409`.

## 12. Dependencias

- `specs/002-disponibilidad.md`
- `specs/003-creacion-turno.md`
- `specs/006-autenticacion-autorizacion.md`
- `specs/012-configuracion-horarios.md`
- `API-CONTRACT.md`
- `ARCHITECTURE.md`
- `DECISIONS.md`

## 13. Fuera de alcance

- edición de bloqueos;
- eliminación de bloqueos;
- bloqueos recurrentes o semanales;
- vacaciones y feriados;
- bloqueos creados por el cliente;
- notificaciones al cliente cuando un turno entra en conflicto con un bloqueo.
