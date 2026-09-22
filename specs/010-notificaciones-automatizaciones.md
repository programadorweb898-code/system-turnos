# Especificación 010 — Notificaciones y automatizaciones

## 1. Objetivo

Definir cómo el sistema emitirá eventos de dominio relacionados con los turnos y cómo esos eventos podrán ser procesados por servicios externos de automatización, inicialmente n8n y WhatsApp.

La automatización debe complementar al sistema de turnos sin formar parte de la lógica crítica del Booking Engine.

## 2. Contexto

El sistema confirma un turno cuando la operación de creación se persiste correctamente en PostgreSQL.

Después de esa confirmación pueden ejecutarse automatizaciones como:

- enviar una confirmación por WhatsApp;
- notificar una cancelación;
- notificar una reprogramación;
- enviar recordatorios futuros.

n8n actúa como sistema externo de automatización. No debe decidir si un turno está disponible ni si un turno queda confirmado.

## 3. Actores

- Administrador del negocio.
- Cliente.
- Backend.
- Booking Engine.
- Sistema de automatización n8n.
- Proveedor de WhatsApp.

## 4. Eventos de dominio

Los eventos relevantes inicialmente son:

- `appointment.created`
- `appointment.cancelled`
- `appointment.rescheduled`

Cada evento debe representar una operación de negocio que haya sido confirmada por el backend.

### 4.1 appointment.created

Se emite después de que la creación del turno haya sido confirmada correctamente.

Uso inicial:

- enviar confirmación del turno por WhatsApp.

### 4.2 appointment.cancelled

Se emite después de que una cancelación haya sido confirmada correctamente.

Uso inicial:

- notificar la cancelación al cliente cuando corresponda.

### 4.3 appointment.rescheduled

Se emite después de que una reprogramación haya sido confirmada correctamente.

Uso inicial:

- notificar al cliente la nueva fecha y horario cuando corresponda.

## 5. Reglas de negocio

1. La creación del turno no depende de n8n.
2. La creación del turno no depende de WhatsApp.
3. Si n8n está caído, el turno igualmente puede quedar confirmado.
4. Si WhatsApp falla, el turno no debe cancelarse automáticamente.
5. Un fallo de automatización debe poder registrarse para su posterior análisis o reintento.
6. Los eventos deben representar únicamente operaciones confirmadas.
7. No debe emitirse un evento de creación para un turno cuya transacción haya fallado.
8. Los eventos deben incluir el contexto del tenant correspondiente.
9. Un evento no debe permitir acceder a información privada de otros tenants.
10. Los consumidores deben poder procesar un mismo evento más de una vez sin generar efectos duplicados cuando la operación lo requiera.

## 6. Datos del evento

El contrato definitivo del evento deberá definirse junto con la implementación de backend.

Como mínimo, un evento debe permitir identificar:

- identificador único del evento;
- tipo de evento;
- fecha/hora de generación;
- tenant;
- turno;
- datos necesarios para la automatización;
- versión del contrato del evento.

Para las notificaciones al cliente podrán ser necesarios:

- nombre y apellido;
- teléfono;
- servicio;
- profesional, cuando corresponda;
- fecha y hora;
- información adicional únicamente si resulta necesaria para la automatización.

No deben incluirse datos personales que no sean necesarios para el procesamiento.

## 7. Confiabilidad de eventos

La implementación debe evitar que un evento confirmado se pierda simplemente porque el sistema externo no estaba disponible en el momento de la operación.

La solución concreta queda para la implementación de backend. Puede utilizarse un patrón Outbox cuando resulte necesario para garantizar la publicación confiable de eventos.

Conceptualmente:

```text
Transacción PostgreSQL
        |
        +--> Appointment
        |
        +--> Event / Outbox
                  |
                  v
             Procesamiento
                  |
                  v
                 n8n
                  |
                  v
              WhatsApp
```

La disponibilidad de n8n no debe formar parte de la transacción crítica de creación del turno.

## 8. Reintentos e idempotencia

Los eventos pueden ser reintentados cuando exista un fallo temporal.

El procesamiento debe contemplar:

- identificador único del evento;
- detección de eventos ya procesados;
- reintentos controlados;
- registro de errores;
- posibilidad de inspeccionar eventos que no pudieron procesarse.

No se debe asumir que un evento será entregado exactamente una sola vez.

## 9. Seguridad

La comunicación entre el backend y n8n debe estar protegida.

La implementación deberá contemplar, según el mecanismo elegido:

- autenticación;
- autorización;
- secretos fuera del código fuente;
- validación del origen;
- protección contra solicitudes no autorizadas;
- protección de datos personales.

n8n no debe obtener acceso directo e innecesario a la base de datos del sistema.

La integración debe utilizar una interfaz explícita de eventos o API.

## 10. WhatsApp

WhatsApp será utilizado inicialmente para enviar la confirmación del turno al número proporcionado por el cliente.

El proveedor concreto de WhatsApp no forma parte de esta especificación.

La automatización deberá recibir los datos necesarios para construir el mensaje, pero la confirmación del turno seguirá siendo responsabilidad del backend.

Un fallo de envío debe poder distinguirse de un fallo de creación del turno.

## 11. Recordatorios

Los recordatorios pueden implementarse posteriormente mediante automatizaciones programadas.

Ejemplos:

- recordatorio previo al turno;
- segundo recordatorio opcional;
- notificación posterior.

Los recordatorios no forman parte del flujo crítico de creación del turno.

## 12. Separación de responsabilidades

### Backend

Responsable de:

- ejecutar las operaciones de negocio;
- confirmar o rechazar turnos;
- persistir los cambios;
- generar eventos correspondientes;
- mantener la integridad y seguridad de los datos.

### Booking Engine

Responsable de:

- disponibilidad;
- reglas de reserva;
- conflictos;
- concurrencia.

No es responsable de:

- enviar WhatsApp;
- ejecutar workflows de n8n;
- administrar plantillas de mensajes.

### n8n

Responsable de:

- recibir eventos;
- ejecutar automatizaciones;
- coordinar notificaciones;
- gestionar reintentos propios de los workflows;
- integrar proveedores externos.

No es responsable de:

- confirmar turnos;
- calcular disponibilidad;
- modificar directamente la base de datos como parte normal del flujo.

### WhatsApp

Responsable únicamente de transportar el mensaje al cliente mediante el proveedor seleccionado.

## 13. Errores

Los errores de automatización deben diferenciarse de los errores de negocio.

Ejemplos:

- `APPOINTMENT_CONFLICT`: conflicto al crear un turno.
- `NOTIFICATION_DELIVERY_FAILED`: fallo en la entrega de una notificación.
- `AUTOMATION_PROCESSING_FAILED`: fallo procesando un evento.

Un fallo de notificación no debe transformarse en un error de creación de turno después de que el turno haya sido confirmado.

## 14. Observabilidad

La implementación debe permitir investigar:

- qué evento se generó;
- cuándo se generó;
- para qué tenant;
- para qué turno;
- si fue procesado;
- cuántos intentos tuvo;
- si falló;
- motivo del último fallo.

Los logs no deben exponer innecesariamente información personal del cliente.

## 15. Cambios de contrato

Los contratos de eventos deben versionarse cuando exista una modificación incompatible.

Un cambio coordinado entre backend y automatizaciones debe actualizar:

- especificación;
- contrato correspondiente;
- implementación;
- pruebas.

No deben introducirse cambios incompatibles silenciosamente.

## 16. Casos principales

### Caso 1 — Creación exitosa

1. Cliente solicita un turno.
2. Backend valida los datos.
3. Booking Engine valida disponibilidad.
4. Backend confirma el turno.
5. Se registra `appointment.created`.
6. n8n procesa el evento.
7. Se solicita el envío de confirmación por WhatsApp.

### Caso 2 — n8n no disponible

1. Cliente solicita un turno.
2. Backend confirma el turno.
3. El evento queda registrado para procesamiento.
4. n8n no está disponible.
5. El turno continúa confirmado.
6. El evento queda pendiente o se reintenta según la implementación.

### Caso 3 — WhatsApp falla

1. El turno ya fue confirmado.
2. n8n intenta enviar la confirmación.
3. El proveedor de WhatsApp devuelve un error.
4. El fallo se registra.
5. El turno continúa confirmado.
6. El sistema puede reintentar posteriormente.

### Caso 4 — Cancelación

1. Se valida la cancelación.
2. Backend cambia el estado del turno.
3. Se registra `appointment.cancelled`.
4. n8n procesa la notificación correspondiente.

### Caso 5 — Reprogramación

1. Se valida la nueva disponibilidad.
2. Backend confirma la reprogramación.
3. Se registra `appointment.rescheduled`.
4. n8n procesa la notificación correspondiente.

## 17. Criterios de aceptación

- [ ] Los eventos se generan únicamente después de operaciones de negocio confirmadas.
- [ ] `appointment.created` puede utilizarse para iniciar la confirmación por WhatsApp.
- [ ] `appointment.cancelled` representa cancelaciones confirmadas.
- [ ] `appointment.rescheduled` representa reprogramaciones confirmadas.
- [ ] Un fallo de n8n no cancela un turno confirmado.
- [ ] Un fallo de WhatsApp no cancela un turno confirmado.
- [ ] Los eventos pueden reintentarse de forma segura.
- [ ] El procesamiento contempla duplicados.
- [ ] Los eventos mantienen aislamiento entre tenants.
- [ ] No se exponen datos personales innecesarios.
- [ ] La integración externa está protegida.
- [ ] Existen mecanismos para investigar fallos de procesamiento.
- [ ] Las automatizaciones no son responsables de la disponibilidad ni de la confirmación del turno.

## 18. Pruebas requeridas

Como mínimo deberán contemplarse pruebas para:

- generación de `appointment.created`;
- generación de `appointment.cancelled`;
- generación de `appointment.rescheduled`;
- evento no generado cuando la operación principal falla;
- persistencia/confiabilidad del evento;
- reintentos;
- procesamiento duplicado;
- aislamiento entre tenants;
- fallo de n8n sin afectar el turno;
- fallo de WhatsApp sin afectar el turno;
- validación de autenticación de la integración;
- ausencia de datos personales innecesarios en el payload.

## 19. Fuera de alcance

Esta especificación no define:

- proveedor concreto de WhatsApp;
- plantillas definitivas de mensajes;
- implementación concreta de workflows n8n;
- pagos;
- cuentas de clientes;
- IA para generación de mensajes;
- infraestructura de mensajería;
- panel avanzado de métricas de notificaciones.

Estos aspectos podrán definirse mediante especificaciones posteriores si son necesarios.
