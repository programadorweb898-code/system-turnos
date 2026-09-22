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

Configura servicios, profesionales, horarios, bloqueos y reglas de reserva que afectan la disponibilidad.

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

## 6. Requisitos funcionales

### RF-001 — Consultar disponibilidad

El sistema debe permitir consultar la disponibilidad para un servicio y una fecha determinada.

### RF-002 — Duración del servicio

La duración del servicio debe determinar la duración ocupada por cada turno.

### RF-003 — Horario laboral

Solo deben generarse intervalos que estén dentro del horario laboral aplicable.

### RF-004 — Profesional

Cuando un servicio requiera un profesional específico, la disponibilidad debe considerar sus horarios y bloqueos.

Cuando el cliente pueda elegir entre varios profesionales habilitados, el sistema debe considerar únicamente los profesionales que puedan realizar el servicio.

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

### RF-009 — Intervalos que cruzan límites

Un intervalo de servicio solo puede considerarse disponible si toda su duración cabe dentro de un período permitido.

### RF-010 — Anticipación mínima

No deben ofrecerse horarios cuyo inicio esté a menos tiempo de anticipación que el configurado por el tenant.

Un turno del mismo día puede aparecer si todavía cumple la anticipación mínima.

### RF-011 — Límite diario

Si el tenant ya alcanzó su cantidad máxima de turnos para una fecha, no debe ofrecerse disponibilidad para nuevos turnos en esa fecha.

La comprobación debe utilizar los estados de turno que ocupan capacidad según las reglas del dominio.

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

No deben ofrecerse horarios cuya hora de inicio ya haya pasado respecto del momento actual en la zona horaria del tenant.

### RN-008 — Anticipación mínima

No deben ofrecerse horarios que no cumplan la anticipación mínima configurada por el tenant.

La regla se evalúa comparando el momento actual con el inicio del turno en la zona horaria correspondiente.

### RN-009 — Límite diario

Cuando la cantidad de turnos que ocupan capacidad para una fecha sea igual al límite diario configurado, no deben generarse nuevas opciones para esa fecha.

La disponibilidad no reemplaza la validación transaccional durante la creación del turno.

### RN-010 — Granularidad

La generación de intervalos debe utilizar una granularidad definida por la configuración del sistema.

Para el MVP, la granularidad inicial será de 15 minutos.

### RN-011 — Pertenencia al tenant

Todos los servicios, profesionales, horarios, bloqueos y turnos utilizados para calcular disponibilidad deben pertenecer al tenant correspondiente.

## 8. Concurrencia

La consulta de disponibilidad no reserva el horario.

Entre la consulta y la confirmación puede existir una carrera entre múltiples clientes.

Por lo tanto:

1. el cliente consulta disponibilidad;
2. selecciona un horario;
3. envía la solicitud de creación;
4. el backend vuelve a ejecutar las validaciones necesarias;
5. la base de datos debe proteger la operación frente a concurrencia;
6. el límite diario también debe validarse de forma segura durante la creación.

## 9. API

El contrato conceptual inicial es:

GET /api/v1/availability

Los parámetros definitivos deberán permitir identificar:

- servicio;
- fecha;
- profesional cuando corresponda.

## 10. Rendimiento

La consulta de disponibilidad debe diseñarse para poder utilizarse tanto desde:

- la página pública;
- el widget;
- el panel administrativo cuando sea necesario.

La implementación inicial debe priorizar corrección y claridad.

## 11. Seguridad

El endpoint público de disponibilidad debe exponer únicamente información necesaria para realizar una reserva.

Debe evitarse revelar datos personales de clientes, detalles internos de otros turnos e información de configuración privada.

## 12. Casos límite

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
- límite diario alcanzado;
- límite diario alcanzado mientras existen horarios laborales libres;
- turno del mismo día dentro de la anticipación mínima;
- turno del mismo día fuera de la anticipación mínima.

## 13. Criterios de aceptación

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
11. se respeta la anticipación mínima;
12. no se ofrece disponibilidad cuando se alcanzó el límite diario;
13. existen pruebas unitarias para las reglas principales;
14. la creación posterior vuelve a validar disponibilidad y límite diario.

## 14. Fuera de alcance

Esta especificación no define todavía:

- creación de turnos;
- cancelación;
- reprogramación;
- pagos;
- recordatorios;
- selección automática avanzada de profesionales;
- límites de reservas por cliente;
- autenticación del cliente;
- caché de disponibilidad;
- optimizaciones de rendimiento avanzadas.

## 15. Dependencias

Esta especificación depende de:

- 001-configuracion-tenant.md;
- SDD.md;
- ARCHITECTURE.md;
- API-CONTRACT.md;
- DECISIONS.md.

La especificación de creación de turnos dependerá de esta definición.
