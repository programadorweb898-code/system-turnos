# Especificación 014 — Modelo de Appointment

## 1. Objetivo

Definir el modelo persistente de un turno y su relación con el tenant, servicio y profesional.

Esta especificación precede a la implementación de la entidad y tabla `appointments`.

## 2. Concepto

Un `Appointment` representa una reserva concreta realizada por un cliente:

- para un tenant;
- para un servicio;
- con un profesional concreto;
- en un intervalo concreto;
- con un estado dentro del ciclo de vida del turno.

El cliente no es una entidad autenticada en el MVP. Sus datos de contacto forman parte del turno.

## 3. Modelo persistente

La tabla `appointments` tendrá inicialmente:

| Campo | Tipo conceptual | Requerido | Descripción |
| --- | --- | --- | --- |
| `id` | UUID | Sí | Identificador del turno |
| `tenant_id` | UUID | Sí | Tenant al que pertenece |
| `customer_name` | varchar(120) | Sí | Nombre y apellido del cliente |
| `customer_phone` | varchar(40) | Sí | Teléfono del cliente |
| `customer_notes` | varchar(300) | No | Información adicional |
| `service_id` | UUID | Sí | Servicio reservado |
| `professional_id` | UUID | Sí | Profesional que atenderá el turno |
| `start_at` | timestamptz | Sí | Inicio real del turno |
| `end_at` | timestamptz | Sí | Fin real del turno |
| `status` | enum lógico | Sí | Estado del turno |
| `created_at` | timestamp | Sí | Fecha de creación |
| `updated_at` | timestamp | Sí | Última modificación |

## 4. Profesional concreto

Aunque el cliente pueda seleccionar inicialmente `Cualquier profesional`, una reserva persistida siempre debe tener un `professional_id` concreto.

Flujo:

```text
Cliente
  ↓
Cualquier profesional
  ↓
Backend encuentra profesional elegible
  ↓
Appointment.professional_id = profesional seleccionado
```

Esto permite que el turno represente exactamente quién atenderá al cliente.

## 5. Relaciones

```text
Tenant
  │
  └── Appointment
       ├── Service
       └── Professional
```

Las tres relaciones deben pertenecer al mismo tenant.

El backend debe validar esta pertenencia antes de crear o modificar un turno.

## 6. Estado

Los estados del dominio serán centralizados y no se representarán mediante strings arbitrarios repartidos por el código.

Estados iniciales:

```text
PENDING
CONFIRMED
CANCELLED
COMPLETED
NO_SHOW
```

Para el flujo público definido actualmente, una creación exitosa produce `CONFIRMED`.

Un turno `CANCELLED` no debe bloquear disponibilidad.

## 7. Intervalos

`start_at` y `end_at` representan el intervalo realmente ocupado por el turno.

`end_at` debe derivarse de:

```text
start_at + duración del servicio
```

El cliente no controla libremente `end_at`.

Los intervalos utilizan la zona horaria del tenant para la lógica de negocio y se almacenan como timestamps con zona horaria.

Los intervalos se consideran semiabiertos:

```text
[start_at, end_at)
```

Por lo tanto:

```text
10:00–10:30
10:30–11:00
```

pueden coexistir.

## 8. Integridad

La tabla debe tener claves foráneas para:

- `tenant_id → tenants.id`
- `service_id → services.id`
- `professional_id → employees.id`

La creación de un turno debe verificar además:

- tenant activo;
- servicio activo y perteneciente al tenant;
- profesional activo y perteneciente al tenant;
- asignación profesional-servicio;
- horario laboral;
- bloqueos;
- reglas de reserva;
- ausencia de solapamiento.

## 9. Concurrencia

La base de datos debe impedir que dos turnos activos para el mismo profesional ocupen intervalos incompatibles.

La protección debe funcionar con múltiples instancias del backend.

La estrategia concreta de PostgreSQL se definirá en la migración de `appointments`, considerando los estados que deben bloquear disponibilidad.

## 10. Cancelación

La cancelación no elimina físicamente el registro.

El estado pasa a `CANCELLED` y el turno deja de bloquear disponibilidad.

Los campos adicionales de auditoría de cancelación y el actor que la realizó se definirán al implementar la operación de cancelación, sin modificar la identidad del turno.

## 11. Datos del cliente

El modelo no crea una tabla de clientes en el MVP.

Los datos:

- nombre y apellido;
- teléfono;
- información adicional;

quedan asociados directamente al turno.

No deben utilizarse como mecanismo de autenticación.

## 12. Idempotencia

La creación pública debe soportar solicitudes repetidas.

El mecanismo concreto de idempotencia se definirá en el contrato de creación de turnos y en su implementación. No se debe asumir que `appointment.id` por sí solo resuelve la idempotencia.

## 13. Eventos

El modelo de `Appointment` podrá producir:

- `appointment.created`
- `appointment.cancelled`
- `appointment.rescheduled`

Los eventos no forman parte de la decisión de disponibilidad.

n8n no participa en la transacción crítica de creación.

## 14. Fuera de alcance

Esta especificación no define todavía:

- tabla de clientes;
- autenticación de clientes;
- pagos;
- reservas recurrentes;
- lista de espera;
- historial completo de cambios;
- estrategia definitiva de idempotencia;
- implementación de Outbox;
- selección avanzada o balanceo entre profesionales.

## 15. Criterios de aceptación

1. Existe una entidad y tabla `appointments`.
2. Cada turno pertenece a un tenant.
3. Cada turno referencia un servicio.
4. Cada turno persistido referencia un profesional concreto.
5. El intervalo completo del turno queda almacenado.
6. La duración determina `end_at`.
7. Los estados están centralizados.
8. Un turno cancelado no bloquea disponibilidad.
9. Las relaciones pertenecen al mismo tenant.
10. La base de datos participa en la protección contra solapamientos.
11. Los datos del cliente se conservan dentro del turno sin crear una cuenta de cliente.
