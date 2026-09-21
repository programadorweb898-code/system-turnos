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
- estado.

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

El modelo detallado de bloqueos será definido en una especificación posterior si requiere reglas adicionales.

## 5. Requisitos funcionales

### RF-001 — Crear tenant

El sistema debe permitir crear un tenant con los datos mínimos requeridos.

### RF-002 — Identificación pública

Cada tenant debe disponer de un slug único que permita identificar su página pública.

Ejemplo:

`turnos.tuplataforma.com/luis-peluqueria`

El dominio definitivo es una decisión de infraestructura y puede cambiar posteriormente.

### RF-003 — Zona horaria

Cada tenant debe tener una zona horaria explícita.

La configuración inicial del proyecto utilizará:

`America/Argentina/Buenos_Aires`

No debe asumirse que será la única zona horaria soportada.

### RF-004 — Crear servicios

El administrador debe poder registrar los servicios que ofrece el negocio.

Cada servicio debe tener una duración válida.

### RF-005 — Crear profesionales

El administrador debe poder registrar uno o más profesionales.

### RF-006 — Configurar horarios

El administrador debe poder definir los días y horarios en los que el negocio o un profesional puede atender.

### RF-007 — Configurar estado

El tenant, los servicios y los profesionales deben disponer de un estado que permita habilitarlos o deshabilitarlos sin eliminar necesariamente sus datos.

## 6. Reglas de negocio

### RN-001 — Aislamiento por tenant

Los datos pertenecientes a un tenant no deben ser accesibles ni modificables desde el contexto de otro tenant.

### RN-002 — Slug único

El slug público debe ser único dentro de la plataforma.

### RN-003 — Duración positiva

La duración de un servicio debe ser mayor que cero.

### RN-004 — Horario válido

La hora de finalización debe ser posterior a la hora de inicio dentro de una jornada normal.

El soporte para horarios que atraviesen medianoche deberá definirse explícitamente antes de implementarse.

### RN-005 — Estados inactivos

Un servicio o profesional inactivo no debe poder utilizarse para nuevas reservas.

La eliminación física de datos no debe ser requisito para desactivar una entidad.

### RN-006 — Configuración incompleta

Un tenant que todavía no tenga la configuración mínima necesaria no debe poder publicar una agenda operativa.

Los criterios exactos de publicación serán definidos en una especificación posterior.

## 7. Validaciones

El backend debe validar todos los datos recibidos.

El frontend puede realizar validaciones anticipadas para mejorar la experiencia, pero no reemplaza las validaciones del backend.

Como mínimo deben validarse:

- campos obligatorios;
- formatos;
- duración;
- horarios;
- identificadores;
- pertenencia al tenant;
- unicidad del slug.

## 8. Seguridad

Todas las operaciones administrativas deben requerir autenticación.

Además de autenticación, el backend debe verificar autorización y pertenencia al tenant correspondiente.

No se debe confiar en un tenant_id enviado libremente por el cliente para determinar el contexto de autorización.

## 9. API

Los endpoints concretos se definirán en la implementación contractual correspondiente, pero deberán respetar:

- REST;
- JSON;
- prefijo `/api/v1`;
- códigos HTTP definidos en `API-CONTRACT.md`;
- formato de errores consistente.

La API administrativa no debe exponer datos de otros tenants.

## 10. Persistencia

La implementación utilizará:

- PostgreSQL;
- Neon;
- TypeORM.

Las entidades y relaciones definitivas se diseñarán en el repositorio backend a partir de esta especificación.

## 11. Criterios de aceptación

La especificación se considera implementada cuando:

1. se puede crear un tenant válido;
2. el tenant obtiene un identificador público único;
3. se puede establecer su zona horaria;
4. se pueden crear servicios válidos;
5. se pueden crear profesionales;
6. se pueden configurar horarios válidos;
7. las entidades pueden deshabilitarse;
8. el backend impide acceder a datos de otro tenant;
9. las validaciones del backend funcionan independientemente de las del frontend;
10. existen pruebas para las reglas de negocio principales.

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
- facturación del SaaS.

Estos temas tendrán especificaciones propias.

## 14. Dependencias

Esta especificación depende de:

- `SDD.md`;
- `AGENTS.md`;
- `ARCHITECTURE.md`;
- `API-CONTRACT.md`;
- `DECISIONS.md`.

Las especificaciones posteriores de disponibilidad y reservas dependerán de esta configuración.
