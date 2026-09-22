# Especificación 002 — Disponibilidad

## 1. Objetivo

Definir cómo el sistema calcula los horarios disponibles para que un cliente pueda solicitar un turno.

La disponibilidad debe ser calculada por el backend mediante el Booking Engine y debe considerar la configuración del tenant, el servicio solicitado, el profesional y los períodos que impiden reservar.

## 2. Contexto

La disponibilidad es una operación central del sistema.

El frontend y el widget pueden solicitar horarios disponibles, pero el resultado mostrado al cliente es informativo hasta que el backend vuelve a validar las condiciones durante la creación del turno.

El Booking Engine es la fuente de verdad para esta lógica.

## 3. Actores

### Cliente

Consulta horarios disponibles para un servicio y, cuando corresponda, un profesional.

### Administrador del tenant

Configura servicios, profesionales, horarios y bloqueos que afectan la disponibilidad.

### Booking Engine

Calcula los intervalos disponibles aplicando las reglas del dominio.

## 4. Entrada mínima

Una consulta de disponibilidad debe permitir determinar:

- tenant;
- servicio;
- fecha o período consultado;
- profesional, cuando corresponda.

El backend debe obtener el contexto del tenant de forma segura y no confiar únicamente en un identificador enviado por el cliente.

## 5. Resultado

La respuesta debe representar los horarios que pueden utilizarse para iniciar un turno.

Cada opción debe contener, como mínimo:

- fecha;
- hora de inicio;
- hora de finalización;
- identificador del profesional cuando corresponda.

La representación exacta del contrato HTTP será definida al implementar el endpoint.

## 6. Requisitos funcionales

### RF-001 — Consultar disponibilidad

El sistema debe permitir consultar la disponibilidad para un servicio y una fecha determinada.

### RF-002 — Duración del servicio

La duración del servicio debe determinar la duración ocupada por cada turno.

Un servicio de 30 minutos no puede generar un intervalo de 15 minutos.

### RF-003 — Horario laboral

Solo deben generarse intervalos que estén dentro del horario laboral aplicable.

### RF-004 — Profesional

Cuando un servicio requiera un profesional específico, la disponibilidad debe considerar sus horarios y bloqueos.

Cuando el cliente pueda elegir entre varios profesionales habilitados, el sistema debe considerar únicamente los profesionales que puedan realizar el servicio.

La regla exacta de asignación automática de profesional se definirá en una especificación posterior.

### RF-005 — Turnos existentes

Los períodos ocupados por turnos existentes deben excluirse de la disponibilidad.

### RF-006 — Bloqueos

Los períodos bloqueados deben excluirse de la disponibilidad.

### RF-007 — Estado de las entidades

No debe ofrecerse disponibilidad para:

- tenants inactivos;
- servicios inactivos;
- profesionales inactivos.

### RF-008 — Zona horaria

Los cálculos deben realizarse respetando la zona horaria configurada para el tenant.

No se debe interpretar una hora local del negocio como UTC sin realizar la conversión correspondiente.

### RF-009 — Intervalos que cruzan límites

Un intervalo de servicio solo puede considerarse disponible si toda su duración cabe dentro de un período permitido.

Ejemplo:

Horario laboral: 09:00–17:00  
Servicio: 60 minutos  
Inicio: 16:30

El horario 16:30 no es válido porque el servicio finalizaría a las 17:30.

## 7. Reglas de negocio

### RN-001 — Fuente de verdad

El Booking Engine es la fuente de verdad para determinar disponibilidad.

### RN-002 — Disponibilidad completa

Un horario es disponible únicamente cuando todo el intervalo requerido por el servicio está libre y permitido.

### RN-003 — Conflicto con turnos

Si el intervalo solicitado se superpone con un turno existente incompatible, el intervalo no debe ofrecerse.

### RN-004 — Conflicto con bloqueos

Si el intervalo solicitado se superpone con un bloqueo incompatible, el intervalo no debe ofrecerse.

### RN-005 — Límite del horario laboral

El turno debe comenzar y terminar dentro del período laboral aplicable.

### RN-006 — Fecha y hora

Las fechas y horas deben interpretarse según la zona horaria del tenant.

### RN-007 — Tiempo pasado

Por defecto, no deben ofrecerse horarios cuya hora de inicio ya haya pasado respecto del momento actual en la zona horaria del tenant.

La política exacta sobre reservas con poca anticipación queda pendiente de una especificación de reglas de reserva.

### RN-008 — Granularidad

La generación de intervalos debe utilizar una granularidad definida por la configuración del sistema.

Para el MVP, la granularidad inicial será de 15 minutos, salvo que una especificación posterior establezca otra regla.

### RN-009 — Pertenencia al tenant

Todos los servicios, profesionales, horarios, bloqueos y turnos utilizados para calcular disponibilidad deben pertenecer al tenant correspondiente.

## 8. Ejemplo conceptual

Configuración:

- horario laboral: 09:00–17:00;
- servicio: 60 minutos;
- granularidad: 15 minutos;
- turno existente: 11:00–12:00.

El Booking Engine puede producir:

- 09:00–10:00;
- 09:15–10:15;
- 09:30–10:30;
- 09:45–10:45;
- 10:00–11:00;
- 12:00–13:00;
- etc.

No debe producir intervalos que se superpongan con 11:00–12:00 ni aquellos que terminen después de las 17:00.

Este ejemplo es conceptual; la política definitiva sobre solapamientos y granularidad será parte del contrato de disponibilidad.

## 9. Concurrencia

La consulta de disponibilidad no reserva el horario.

Entre la consulta y la confirmación puede existir una carrera entre múltiples clientes.

Por lo tanto:

1. el cliente consulta disponibilidad;
2. selecciona un horario;
3. envía la solicitud de creación;
4. el backend vuelve a ejecutar las validaciones necesarias;
5. la base de datos debe proteger la operación frente a concurrencia.

El mecanismo concreto de protección se definirá en la especificación de creación de turnos.

## 10. API

El contrato conceptual inicial es:

`GET /api/v1/availability`

Los parámetros definitivos deberán permitir identificar:

- servicio;
- fecha;
- profesional cuando corresponda.

La respuesta debe utilizar el formato de errores establecido en `API-CONTRACT.md`.

No debe incluir información privada de otros clientes ni datos innecesarios para reservar.

## 11. Rendimiento

La consulta de disponibilidad debe diseñarse para poder utilizarse tanto desde:

- la página pública;
- el widget;
- el panel administrativo cuando sea necesario.

La implementación inicial debe priorizar corrección y claridad.

Se podrán introducir índices, caché u otras optimizaciones cuando existan mediciones o necesidades concretas.

La caché no debe convertirse en la fuente de verdad de la disponibilidad.

## 12. Seguridad

El endpoint público de disponibilidad debe exponer únicamente información necesaria para realizar una reserva.

Debe evitarse revelar:

- datos personales de clientes;
- detalles internos de otros turnos;
- información de configuración privada.

Las operaciones administrativas relacionadas con horarios y bloqueos requieren autenticación y autorización.

## 13. Casos límite

Deben contemplarse al menos:

- servicio inexistente;
- servicio perteneciente a otro tenant;
- servicio inactivo;
- tenant inexistente;
- tenant inactivo;
- profesional inexistente;
- profesional de otro tenant;
- profesional inactivo;
- fecha inválida;
- fecha en el pasado;
- día sin horario laboral;
- horario completamente bloqueado;
- servicio más largo que el horario laboral;
- turno existente parcialmente superpuesto;
- bloqueo parcialmente superpuesto;
- múltiples profesionales disponibles;
- ningún profesional disponible;
- cambio de zona horaria;
- horario de verano o cambios de offset cuando sean relevantes para la zona configurada.

## 14. Criterios de aceptación

La especificación se considera implementada cuando:

1. se puede consultar disponibilidad para un servicio y fecha;
2. los intervalos respetan la duración del servicio;
3. los intervalos respetan el horario laboral;
4. los turnos existentes se excluyen;
5. los bloqueos se excluyen;
6. se respetan estados de tenant, servicio y profesional;
7. los cálculos respetan la zona horaria del tenant;
8. no se ofrecen intervalos que excedan el horario laboral;
9. se respeta la granularidad definida;
10. se verifica correctamente la pertenencia al tenant;
11. existen pruebas unitarias para las reglas principales;
12. existen pruebas para casos de solapamiento;
13. la consulta no se utiliza como garantía de reserva;
14. la creación posterior vuelve a validar la disponibilidad.

## 15. Fuera de alcance

Esta especificación no define todavía:

- creación de turnos;
- cancelación;
- reprogramación;
- pagos;
- recordatorios;
- selección automática avanzada de profesionales;
- reglas de anticipación mínima o máxima;
- límites de reservas por cliente;
- autenticación del cliente;
- caché de disponibilidad;
- optimizaciones de rendimiento avanzadas.

## 16. Dependencias

Esta especificación depende de:

- `001-configuracion-tenant.md`;
- `SDD.md`;
- `ARCHITECTURE.md`;
- `API-CONTRACT.md`;
- `DECISIONS.md`.

La especificación de creación de turnos dependerá de esta definición.
