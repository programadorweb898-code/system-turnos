# Especificación 001 — Configuración inicial del tenant

## 1. Objetivo

Definir la configuración mínima que permite crear y preparar un negocio dentro de la plataforma para utilizar el sistema de turnos.

La configuración debe ser genérica y no depender de una profesión específica.

## 2. Contexto

La plataforma es multi-tenant.

Cada negocio debe disponer de una configuración propia antes de poder publicar su sistema de reservas.

Esta especificación define la base funcional sobre la que posteriormente se apoyarán la disponibilidad, la página pública y el widget.

## 3. Actores

### Administrador del tenant

Persona responsable de configurar el negocio.

Puede:
- crear o completar los datos del negocio;
- configurar servicios;
- configurar profesionales;
- definir horarios laborales;
- definir reglas básicas de reserva.

### Cliente

No configura el tenant.

Utilizará posteriormente la configuración para consultar disponibilidad y solicitar turnos.

## 4. Datos principales

La configuración inicial debe contemplar como mínimo:

### Tenant
- identificador;
- nombre comercial;
- slug público;
- zona horaria;
- estado;
- cantidad máxima de turnos por día;
- anticipación mínima para solicitar un turno.

### Servicio
- identificador;
- nombre;
- descripción opcional;
- duración;
- estado.

### Profesional
- identificador;
- nombre;
- estado.

### Horario laboral
- día de la semana;
- hora de inicio;
- hora de finalización;
- profesional o ámbito al que aplica.

### Bloqueo de agenda

Debe poder representar períodos en los que no se pueden asignar turnos.

## 5. Requisitos funcionales

### RF-001 — Crear tenant

El sistema debe permitir crear un tenant con los datos mínimos requeridos.

### RF-002 — Identificación pública

Cada tenant debe disponer de un slug único que permita identificar su página pública.

### RF-003 — Zona horaria

Cada tenant debe tener una zona horaria explícita.

La configuración inicial del proyecto utilizará America/Argentina/Buenos_Aires.

### RF-004 — Crear servicios

El administrador debe poder registrar los servicios que ofrece el negocio.

Cada servicio debe tener una duración válida.

### RF-005 — Crear profesionales

El administrador debe poder registrar uno o más profesionales.

### RF-006 — Configurar horarios

El administrador debe poder definir los días y horarios en los que el negocio o un profesional puede atender.

### RF-007 — Configurar estado

El tenant, los servicios y los profesionales deben disponer de un estado que permita habilitarlos o deshabilitarlos sin eliminar necesariamente sus datos.

### RF-008 — Configurar límite diario de turnos

El administrador debe poder establecer una cantidad máxima de turnos que el negocio puede aceptar por día.

El límite se aplicará inicialmente al tenant en su conjunto.

### RF-009 — Configurar anticipación mínima

El administrador debe poder establecer cuánto tiempo de anticipación mínima se requiere para solicitar un turno.

La regla se evaluará respecto de la hora actual en la zona horaria del tenant y del inicio del turno.

Si la anticipación mínima es de 2 horas, un turno para hoy sigue siendo válido si comienza al menos 2 horas después del momento actual y cumple las demás reglas de disponibilidad.

## 6. Reglas de negocio

### RN-001 — Aislamiento por tenant

Los datos pertenecientes a un tenant no deben ser accesibles ni modificables desde el contexto de otro tenant.

### RN-002 — Slug único

El slug público debe ser único dentro de la plataforma.

### RN-003 — Duración positiva

La duración de un servicio debe ser mayor que cero.

### RN-004 — Horario válido

La hora de finalización debe ser posterior a la hora de inicio dentro de una jornada normal.

### RN-005 — Estados inactivos

Un servicio o profesional inactivo no debe poder utilizarse para nuevas reservas.

### RN-006 — Configuración incompleta

Un tenant que todavía no tenga la configuración mínima necesaria no debe poder publicar una agenda operativa.

### RN-007 — Límite diario

No se debe permitir la confirmación de nuevos turnos cuando el tenant ya haya alcanzado su cantidad máxima de turnos para ese día.

La validación debe realizarse nuevamente durante la creación del turno para evitar superar el límite por concurrencia.

### RN-008 — Anticipación mínima

No se debe permitir un turno cuyo inicio esté a menos tiempo de anticipación que el configurado por el tenant.

La regla se calcula utilizando la zona horaria del tenant.

La anticipación mínima no implica prohibir todos los turnos del mismo día: si todavía se cumple el tiempo mínimo requerido, el turno del mismo día puede reservarse.

### RN-009 — Valores de reglas válidos

La cantidad máxima diaria no puede ser negativa.

La anticipación mínima no puede ser negativa.

## 7. Validaciones

El backend debe validar todos los datos recibidos.

El frontend puede realizar validaciones anticipadas, pero no reemplaza las validaciones del backend.

Como mínimo deben validarse:
- campos obligatorios;
- formatos;
- duración;
- horarios;
- identificadores;
- pertenencia al tenant;
- unicidad del slug;
- cantidad máxima diaria;
- anticipación mínima.

## 8. Seguridad

Todas las operaciones administrativas deben requerir autenticación.

Además de autenticación, el backend debe verificar autorización y pertenencia al tenant correspondiente.

No se debe confiar en un tenant_id enviado libremente por el cliente para determinar el contexto de autorización.

## 9. API

Los endpoints concretos se definirán en la implementación contractual correspondiente y deberán respetar REST, JSON, prefijo /api/v1, códigos HTTP definidos en API-CONTRACT.md y formato de errores consistente.

## 10. Persistencia

La implementación utilizará PostgreSQL, Neon y TypeORM.

Las entidades y relaciones definitivas se diseñarán en el repositorio backend a partir de esta especificación.

## 11. Criterios de aceptación

La especificación se considera implementada cuando:
1. se puede crear un tenant válido;
2. el tenant obtiene un identificador público único;
3. se puede establecer su zona horaria;
4. se pueden configurar el límite diario y la anticipación mínima;
5. se pueden crear servicios válidos;
6. se pueden crear profesionales;
7. se pueden configurar horarios válidos;
8. las entidades pueden deshabilitarse;
9. el backend impide acceder a datos de otro tenant;
10. las validaciones del backend funcionan independientemente de las del frontend;
11. existen pruebas para las reglas de negocio principales;
12. el sistema impide confirmar turnos por encima del límite diario;
13. el sistema impide confirmar turnos que no cumplan la anticipación mínima.

## 12. Casos límite

Deben contemplarse al menos:
- slug duplicado;
- servicio con duración cero;
- servicio con duración negativa;
- horario con inicio igual a fin;
- horario con fin anterior al inicio;
- profesional perteneciente a otro tenant;
- servicio perteneciente a otro tenant;
- tenant inexistente;
- tenant deshabilitado;
- servicio deshabilitado;
- profesional deshabilitado;
- cantidad máxima diaria igual a cero;
- cantidad máxima diaria negativa;
- límite diario alcanzado;
- dos solicitudes concurrentes que podrían superar el límite diario;
- anticipación mínima igual a cero;
- anticipación mínima negativa;
- turno del mismo día dentro del margen permitido;
- turno del mismo día fuera del margen permitido;
- solicitud que cruza el límite de anticipación por pocos segundos;
- solicitudes administrativas sin autenticación;
- solicitudes autenticadas sin autorización suficiente.

## 13. Fuera de alcance

Esta especificación no define todavía:
- algoritmo completo de disponibilidad;
- creación de turnos;
- cancelación de turnos;
- reprogramación;
- pagos;
- notificaciones;
- integración con WhatsApp;
- widget;
- página pública completa;
- configuración mediante IA;
- autenticación detallada;
- facturación del SaaS;
- límites de reservas por cliente;
- límites diarios específicos por profesional.

## 14. Dependencias

Esta especificación depende de:
- SDD.md;
- AGENTS.md;
- ARCHITECTURE.md;
- API-CONTRACT.md;
- DECISIONS.md.

Las especificaciones posteriores de disponibilidad y reservas dependerán de esta configuración.
