# Decisiones arquitectónicas

Este documento registra decisiones estructurales ya acordadas para evitar cambios accidentales o inconsistentes durante el desarrollo.

## 1. Producto genérico y multi-tenant

El sistema no estará diseñado exclusivamente para peluquerías.

Debe permitir configurar sistemas de turnos para distintos tipos de profesionales y negocios mediante un modelo multi-tenant.

Cada negocio representa un tenant y sus datos deben permanecer aislados.

## 2. Un backend genérico

No se generará un backend independiente para cada negocio.

Todos los tenants utilizarán el mismo backend y la misma arquitectura de aplicación, con aislamiento lógico de los datos.

## 3. Booking Engine como fuente de verdad

La lógica de disponibilidad y reserva estará centralizada en un Booking Engine.

El Booking Engine será responsable de:

- validar servicios;
- validar profesionales;
- calcular disponibilidad;
- comprobar horarios laborales;
- comprobar bloqueos;
- aplicar reglas de reserva;
- detectar conflictos;
- proteger la creación de turnos frente a concurrencia.

El frontend nunca será la autoridad final sobre la disponibilidad.

## 4. Frontend y backend separados

El frontend y el backend serán proyectos/repositorios independientes.

Repositorios previstos:

- `system-turnos`: especificaciones, arquitectura y coordinación;
- `system-turnos-backend`: API y lógica de negocio;
- `system-turnos-frontend`: interfaz y experiencia de usuario.

## 5. Tres agentes especializados

El desarrollo se organizará con tres agentes:

- Orchestrator Agent
- Backend Agent
- Frontend Agent

El Orchestrator coordina requisitos, dependencias, contratos y consistencia general.

El Backend Agent implementa la lógica del servidor.

El Frontend Agent implementa la interfaz y las integraciones del cliente.

## 6. SDD como metodología

El proyecto utilizará Specification-Driven Development.

No se debe comenzar una implementación importante sin una especificación suficientemente clara.

La especificación precede al plan y a la implementación.

## 7. Integraciones iniciales

El producto contempla dos formas principales de uso:

1. página pública de turnos proporcionada por la plataforma;
2. widget integrable en un sitio web existente.

La integración mediante modificación profunda del código fuente de sitios externos queda fuera del alcance inicial.

## 8. IA como interfaz de configuración

La IA podrá ayudar al profesional a expresar su configuración en lenguaje natural.

Ejemplo:

> Trabajo de lunes a sábado de 9 a 17 y hago cortes de 30 minutos.

La IA podrá transformar esa intención en una configuración estructurada.

La configuración deberá ser validada por el backend antes de persistirse.

La IA no será la fuente de verdad de:

- disponibilidad;
- reservas;
- reglas críticas;
- integridad de datos.

## 9. n8n fuera del núcleo de reservas

n8n se utilizará para automatizaciones y notificaciones, por ejemplo:

- WhatsApp;
- email;
- recordatorios;
- notificaciones.

La creación y gestión de turnos no dependerá de que n8n esté disponible.

Los eventos de dominio podrán utilizarse para desacoplar las automatizaciones del núcleo de reservas.

## 10. Persistencia

La base de datos principal será PostgreSQL, utilizando Neon como infraestructura inicial.

TypeORM será el ORM del backend.

## 11. Stack inicial

Backend:

- Node.js
- TypeScript
- Express
- PostgreSQL
- Neon
- TypeORM

Frontend:

- Next.js
- TypeScript

Infraestructura inicial:

- Vercel para frontend;
- Render para backend;
- Neon para PostgreSQL;
- n8n para automatizaciones.

## 12. API REST versionada

La API utilizará REST sobre HTTP/HTTPS.

La versión inicial utilizará el prefijo:

`/api/v1`

Los contratos entre frontend y backend estarán documentados y los cambios coordinados.

## 13. Concurrencia en reservas

La disponibilidad mostrada al usuario no garantiza que el turno siga disponible al confirmar.

El backend deberá volver a validar las condiciones en el momento de crear la reserva y utilizar mecanismos de base de datos apropiados para evitar doble reserva bajo concurrencia.

## 14. Zona horaria explícita

La zona horaria será un dato explícito de la configuración del tenant.

La configuración inicial del proyecto utilizará:

`America/Argentina/Buenos_Aires`

La arquitectura no debe asumir que todos los tenants utilizarán necesariamente esa zona horaria en el futuro.

## 15. Sin microservicios prematuros

El sistema comenzará como una aplicación backend modular.

No se introducirán microservicios únicamente por anticipar una futura escala.

La separación en servicios podrá evaluarse posteriormente cuando exista una necesidad técnica concreta.

## 16. Rama de desarrollo

La rama `main` representa el estado estable.

La rama `development` es la línea principal de desarrollo actual.

Las funcionalidades nuevas deberán desarrollarse sobre `development` o ramas derivadas de ella, según corresponda.

## 17. Commits en español

Los mensajes de commit del proyecto estarán escritos en español y deberán describir claramente el cambio realizado.

## 18. Fuente de verdad documental

Las decisiones y reglas del proyecto se distribuyen de la siguiente manera:

- `SDD.md`: metodología;
- `AGENTS.md`: responsabilidades y reglas de los agentes;
- `ARCHITECTURE.md`: arquitectura;
- `API-CONTRACT.md`: contrato frontend/backend;
- `DECISIONS.md`: decisiones arquitectónicas;
- `specs/`: especificaciones funcionales concretas.

Cuando exista una contradicción, debe resolverse antes de implementar.

## 19. Evitar sobreingeniería

Las decisiones técnicas deben resolver necesidades reales del producto.

No se incorporarán tecnologías, abstracciones o componentes complejos solamente por ser técnicamente posibles.

## 20. Alcance de esta etapa

El objetivo actual es establecer correctamente la base arquitectónica y documental antes de construir el backend y frontend.

Las decisiones futuras deberán registrarse en este documento cuando tengan impacto arquitectónico significativo.
