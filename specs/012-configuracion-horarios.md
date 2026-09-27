# Especificación 012 — Configuración de horarios de atención

## 1. Objetivo

Definir cómo el administrador configura los horarios de atención del tenant.

En el MVP, los horarios de atención aplican al tenant en su conjunto. No se implementan todavía horarios individuales por profesional.

## 2. Actor

### Administrador del tenant

Puede consultar y crear horarios de atención del tenant autenticado.

Todas las operaciones requieren autenticación y aislamiento por tenant.

## 3. Modelo

Se utilizará la entidad existente `BusinessHour`:

- `id`
- `tenantId`
- `dayOfWeek`
- `startTime`
- `endTime`
- `createdAt`
- `updatedAt`

## 4. Reglas

- `dayOfWeek` debe representar un día válido de la semana, de 0 a 6.
- `startTime` y `endTime` deben utilizar formato `HH:mm`.
- `endTime` debe ser posterior a `startTime`.
- El horario no puede pertenecer a otro tenant.
- El `tenantId` se obtiene del contexto autenticado y no del body.
- Los horarios se interpretan como hora local del tenant.
- Un día puede tener más de un intervalo para permitir futuras pausas dentro de una jornada.
- No se implementa todavía edición ni eliminación de horarios.
- No se implementan horarios individuales por profesional.

## 5. API

### GET /api/v1/admin/configuration/business-hours

Devuelve los horarios del tenant autenticado.

Respuesta exitosa: `200 OK`.

Cada elemento contiene:

- `id`
- `dayOfWeek`
- `startTime`
- `endTime`

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

Respuesta exitosa: `201 Created`.

Datos inválidos: `400 Bad Request`.

Si el cliente envía `tenantId`, debe ignorarse.

## 6. Casos límite

- día menor que 0;
- día mayor que 6;
- hora inválida;
- inicio igual a fin;
- fin anterior al inicio;
- solicitud sin autenticación;
- intento de consultar o modificar datos de otro tenant mediante valores enviados por el cliente.

## 7. Criterios de aceptación

1. El administrador puede consultar sus horarios.
2. El administrador puede crear un horario válido.
3. El backend valida día y horas.
4. El backend utiliza el tenant del contexto autenticado.
5. Un `tenantId` enviado por el cliente no cambia el tenant afectado.
6. Los horarios quedan ordenados por día y hora de inicio.
7. Existen tests para validación, aislamiento y endpoints principales.

## 8. Fuera de alcance

- edición de horarios;
- eliminación de horarios;
- horarios por profesional;
- vacaciones;
- feriados;
- excepciones de horario;
- disponibilidad pública;
- cálculo de disponibilidad.

## 9. Dependencias

- `specs/001-configuracion-tenant.md`
- `specs/002-disponibilidad.md`
- `specs/006-autenticacion-autorizacion.md`
- `API-CONTRACT.md`
- `ARCHITECTURE.md`
- `DECISIONS.md`
