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

## 21. Autenticación únicamente para administración

En el MVP, únicamente el administrador o propietario del negocio requerirá autenticación y autorización.

El cliente podrá reservar un turno sin crear una cuenta.

La identidad administrativa estará asociada a un tenant y el backend deberá garantizar el aislamiento entre tenants.

El mecanismo concreto de autenticación queda para la implementación y su especificación correspondiente.

## 22. Datos mínimos del cliente para reservar

El cliente deberá proporcionar:

- nombre y apellido, obligatorio;
- teléfono, obligatorio;
- información adicional, opcional, con un máximo de 300 caracteres.

No se creará una cuenta de cliente como requisito para reservar.

## 23. Confirmación de turno por WhatsApp

El teléfono proporcionado por el cliente se utilizará para enviar la confirmación del turno mediante WhatsApp.

La confirmación del turno no dependerá de que WhatsApp o n8n hayan procesado correctamente la notificación.

El turno queda confirmado cuando la operación de reserva se persiste correctamente y cumple las reglas del Booking Engine.

## 24. Eventos y automatizaciones desacoplados del Booking Engine

Las automatizaciones externas se iniciarán mediante eventos de dominio posteriores a operaciones confirmadas.

Los eventos iniciales son:

- `appointment.created`;
- `appointment.cancelled`;
- `appointment.rescheduled`.

La implementación deberá contemplar confiabilidad, reintentos e idempotencia cuando sean necesarios.

n8n no será una dependencia síncrona de las operaciones críticas de reserva.

## 25. Publicación del negocio

Un negocio no estará disponible públicamente para nuevas reservas hasta que haya sido publicado explícitamente por su administrador y cumpla la configuración mínima requerida.

La publicación y despublicación son operaciones administrativas.

Despublicar un negocio impide nuevas reservas públicas, pero no elimina el historial existente.

## 26. Acceso público sin exposición de datos administrativos

La página pública y el widget utilizarán el mismo backend y Booking Engine que las operaciones administrativas, pero mediante recursos públicos limitados.

Nunca deberán exponerse credenciales, secretos, información de otros clientes ni datos administrativos innecesarios.

## 27. Operaciones posteriores del cliente

Las operaciones públicas de cancelación o reprogramación por parte del cliente no requerirán una cuenta de cliente en el MVP.

El mecanismo seguro concreto para permitir esas operaciones queda pendiente de una decisión y especificación posterior.

## 28. Alcance actual de notificaciones

La primera notificación obligatoria del flujo público será la confirmación del turno mediante WhatsApp.

Las notificaciones de cancelación, reprogramación y recordatorios podrán utilizar los eventos correspondientes, pero su implementación concreta queda fuera de esta etapa documental inicial.


## 29. Configuración de reglas por el profesional

Después de registrarse en la plataforma, el profesional o administrador responsable deberá completar la configuración de su negocio antes de publicarlo.

La cantidad máxima de turnos diarios y la anticipación mínima para solicitar turnos serán reglas configurables por ese profesional.

La anticipación mínima se expresará en horas.

Estas reglas no serán decididas por el cliente ni por el Booking Engine; el Booking Engine únicamente las aplicará y validará.
