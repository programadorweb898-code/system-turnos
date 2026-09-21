# ARCHITECTURE.md — Arquitectura del sistema

## 1. Objetivo

Este documento define la arquitectura técnica y conceptual de System Turnos.

La plataforma debe permitir crear y operar sistemas de reservas para distintos tipos de negocios y profesionales utilizando un único motor de reservas multi-tenant.

La arquitectura debe separar claramente:

- Configuración del negocio.
- Motor de reservas.
- API.
- Interfaz pública.
- Panel administrativo.
- Integración mediante widget.
- Automatizaciones y notificaciones.

## 2. Principios arquitectónicos

La arquitectura seguirá estos principios:

1. Multi-tenancy desde el diseño inicial.
2. Backend como fuente de verdad para las reglas de negocio.
3. Frontend desacoplado del backend.
4. Contratos API explícitos.
5. Motor de reservas determinístico.
6. Separación entre configuración y operación.
7. Integraciones desacopladas.
8. Seguridad por defecto.
9. Persistencia consistente ante concurrencia.
10. Evitar sobreingeniería.
11. Mantener la arquitectura extensible sin implementar complejidad innecesaria.

## 3. Arquitectura de alto nivel

La arquitectura conceptual es:

    SYSTEM TURNOS
           |
    +------+------+------+
    |             |      |
    v             v      v
Configuración  Booking  Integraciones
    |           Engine       |
    v             |          +--> Widget
Config Engine     v          +--> n8n
Wizard / IA   PostgreSQL
                  / Neon
                    |
                    v
              Eventos / Outbox

La interfaz pública y el widget deben utilizar el mismo Booking Engine.

## 4. Repositorios

El sistema se divide en tres repositorios.

### 4.1 Repositorio de coordinación

system-turnos

Responsabilidad:

- Especificaciones.
- Arquitectura.
- Contratos.
- Decisiones.
- Reglas de agentes.
- Coordinación.

No contiene la implementación principal del backend ni del frontend.

### 4.2 Repositorio backend

system-turnos-backend

Responsabilidad:

- API REST.
- Motor de reservas.
- Persistencia.
- Multi-tenancy.
- Autenticación y autorización.
- Validaciones.
- Eventos.
- Integraciones backend.
- Pruebas backend.

Tecnologías objetivo:

- Node.js.
- TypeScript.
- Express.
- TypeORM.
- PostgreSQL.
- Neon.

### 4.3 Repositorio frontend

system-turnos-frontend

Responsabilidad:

- Interfaz pública.
- Panel administrativo.
- Configuración.
- Selección de fechas.
- Selección de horarios.
- Widget.
- Experiencia responsive.
- Accesibilidad.
- Pruebas frontend.

Tecnologías objetivo:

- Next.js.
- TypeScript.

## 5. Separación frontend/backend

Frontend y backend deben ser proyectos independientes.

El frontend no debe acceder directamente a PostgreSQL.

El backend es responsable de:

- Validar solicitudes.
- Aplicar reglas de negocio.
- Consultar disponibilidad.
- Crear reservas.
- Validar concurrencia.
- Aplicar autorización.
- Persistir cambios.

El frontend es responsable de presentar y consumir los datos proporcionados por la API.

## 6. Multi-tenancy

El concepto central del dominio es el Tenant.

Un tenant representa un negocio o profesional que utiliza la plataforma.

Ejemplo conceptual:

    Tenant
    ├── Usuarios
    ├── Profesionales
    ├── Servicios
    ├── Horarios
    ├── Bloqueos
    └── Reservas

Cada entidad perteneciente a un negocio debe poder asociarse inequívocamente con su tenant cuando corresponda.

### Regla fundamental

Una operación realizada en nombre de un tenant nunca debe poder acceder accidentalmente a datos pertenecientes a otro tenant.

El aislamiento debe aplicarse en:

- API.
- Servicios.
- Consultas.
- Autorización.
- Operaciones administrativas.

## 7. Modelo conceptual del dominio

Las entidades principales previstas son:

    Tenant
    User
    Employee
    Service
    BusinessHour
    BlockedTime
    Appointment

Podrán agregarse nuevas entidades cuando una especificación lo requiera.

### Tenant

Representa el negocio.

Información conceptual:

- id.
- nombre.
- tipo de negocio.
- slug.
- configuración.
- zona horaria.
- estado.
- timestamps.

### User

Representa una cuenta autenticada de la plataforma.

Puede estar asociada a uno o más tenants según las reglas de autorización que se definan.

### Employee

Representa a un profesional que presta servicios dentro de un tenant.

Ejemplos:

- Peluquero.
- Psicólogo.
- Dentista.
- Abogado.
- Entrenador.
- Técnico.

### Service

Representa un servicio reservable.

Debe incluir, como mínimo conceptualmente:

- Nombre.
- Duración.
- Estado.
- Tenant.
- Precio cuando corresponda.

### BusinessHour

Representa los horarios de atención del tenant o de un profesional, según las reglas que posteriormente se definan.

### BlockedTime

Representa períodos durante los cuales no se pueden realizar reservas.

Ejemplos:

- Vacaciones.
- Reuniones.
- Mantenimiento.
- Ausencias.
- Bloqueos manuales.

### Appointment

Representa una reserva.

Debe mantener la información necesaria para determinar:

- Tenant.
- Servicio.
- Profesional cuando corresponda.
- Fecha.
- Hora de inicio.
- Hora de finalización.
- Datos mínimos del cliente.
- Estado.
- Timestamps.

## 8. Motor de reservas

El Booking Engine es el núcleo funcional del sistema.

Su responsabilidad es determinar qué horarios pueden reservarse y garantizar que una reserva válida pueda persistirse correctamente.

El motor debe considerar:

- Tenant.
- Servicio.
- Duración.
- Profesional.
- Horarios de atención.
- Reservas existentes.
- Bloqueos.
- Zona horaria.
- Reglas de reserva.

## 9. Disponibilidad

La disponibilidad no debe almacenarse como una lista permanente de slots disponibles.

Debe calcularse a partir de las reglas y datos actuales.

Conceptualmente:

    Disponibilidad =
    Horarios de atención
    - Reservas existentes
    - Bloqueos
    - Restricciones

La duración del servicio debe formar parte del cálculo.

Un horario de inicio solamente es válido si existe espacio suficiente para completar el servicio.

## 10. Reserva y concurrencia

La disponibilidad mostrada al usuario es informativa.

Al confirmar una reserva, el backend debe volver a comprobar las condiciones actuales.

Ejemplo:

    Usuario A consulta 10:00 → disponible
    Usuario B consulta 10:00 → disponible

    Usuario A confirma
    Usuario B confirma simultáneamente

El sistema debe garantizar que no se creen dos reservas incompatibles para el mismo recurso.

La protección debe depender de mecanismos de base de datos y transacciones apropiadas, no únicamente del frontend.

## 11. Estados de una reserva

La reserva tendrá un ciclo de vida definido mediante estados.

Como mínimo deberán contemplarse conceptualmente:

- Pendiente.
- Confirmada.
- Cancelada.
- Completada.

Los estados definitivos y las transiciones válidas deberán establecerse en las especificaciones funcionales correspondientes.

## 12. Página pública de reservas

La plataforma debe permitir generar una interfaz pública para negocios que no poseen sitio web.

Ejemplo conceptual:

    turnos.<plataforma>/<slug>

Esta interfaz debe permitir:

1. Identificar el negocio.
2. Seleccionar servicio.
3. Seleccionar profesional cuando corresponda.
4. Seleccionar fecha.
5. Consultar disponibilidad.
6. Seleccionar horario.
7. Introducir los datos requeridos.
8. Confirmar la reserva.

La interfaz no debe implementar reglas de disponibilidad por cuenta propia.

## 13. Widget

Los negocios que ya poseen un sitio web podrán integrar el sistema mediante un widget.

Objetivos:

- Integración sencilla.
- No modificar directamente el código fuente del sitio.
- Aislar la aplicación de reservas.
- Mantener responsive design.
- Permitir configuración visual.

La primera versión deberá priorizar una integración segura y desacoplada.

Las estrategias exactas de implementación del widget se definirán en una especificación propia.

## 14. Análisis de sitios existentes

Como parte de la experiencia de integración, la plataforma podrá analizar un sitio existente para proponer:

- Ubicación del widget.
- Estilo visual.
- Colores.
- Tipografías.
- Espaciado.
- Componentes visuales relevantes.

El análisis puede utilizar automatización, crawling, Playwright e IA cuando sea necesario.

### Restricción

El sistema no debe modificar automáticamente el sitio externo en esta etapa.

El resultado del análisis debe utilizarse para configurar o proponer la integración del widget.

## 15. Configuración del negocio

El sistema debe permitir que un profesional configure su sistema sin conocimientos técnicos.

La configuración podrá realizarse mediante:

- Formulario.
- Wizard.
- Interfaz conversacional.
- IA.

La interfaz utilizada para configurar el negocio no debe cambiar la naturaleza del motor de reservas.

## 16. IA como interfaz de configuración

La IA puede transformar lenguaje natural en una configuración estructurada.

Ejemplo conceptual:

    Usuario:
    "Trabajo de lunes a sábado de 9 a 17.
    Hago cortes de 30 minutos y coloración de 90 minutos."

    IA:
    {
      services: [...],
      schedule: [...]
    }

El backend debe:

1. Recibir la configuración.
2. Validarla.
3. Aplicar reglas de negocio.
4. Persistirla si es válida.

La IA no debe ser la fuente de verdad.

No debe decidir directamente si un horario puede reservarse.

## 17. Autenticación y autorización

El sistema distinguirá entre:

- Usuarios de la plataforma.
- Usuarios administrativos de un tenant.
- Profesionales.
- Clientes que realizan reservas.

El cliente no necesita una cuenta para realizar una reserva en el MVP.

La autenticación y autorización administrativa deberán diseñarse separadamente de la reserva pública.

## 18. Eventos y automatizaciones

El backend debe poder producir eventos de dominio relacionados con reservas.

Eventos iniciales:

    appointment.created
    appointment.updated
    appointment.cancelled

El sistema debe separar:

1. Persistencia de la operación.
2. Emisión/procesamiento del evento.
3. Automatización externa.

Esto permite que una falla de un servicio externo no invalide una reserva.

## 19. Outbox

Cuando la implementación requiera garantías mayores de entrega de eventos, se utilizará el patrón Outbox.

Conceptualmente:

    Transacción
        |
        +--> Appointment
        |
        +--> Outbox Event
                  |
                  v
             Procesamiento
                  |
                  v
                 n8n
                  |
             +----+----+
             |         |
           WhatsApp   Email

El uso concreto de Outbox debe justificarse según los requisitos de confiabilidad y no implementarse prematuramente si la especificación no lo necesita.

## 20. n8n

n8n será una capa de automatización externa.

Puede encargarse de:

- WhatsApp.
- Email.
- Recordatorios.
- Notificaciones.
- Flujos externos.

n8n no debe ser responsable de:

- Determinar disponibilidad.
- Garantizar ausencia de doble reserva.
- Ser la base de datos principal.
- Implementar las reglas centrales del Booking Engine.

## 21. Base de datos

La base de datos objetivo es PostgreSQL.

La infraestructura inicial prevista es Neon.

TypeORM será utilizado como ORM.

Las modificaciones estructurales de la base de datos deben realizarse mediante migraciones.

No se debe depender de modificaciones manuales de producción.

## 22. Zona horaria

La plataforma debe tratar la zona horaria como una propiedad explícita del tenant.

Para el contexto inicial del proyecto se utilizará:

    America/Argentina/Buenos_Aires

No debe asumirse que todos los tenants utilizan la misma zona horaria en la arquitectura definitiva.

Las reglas para almacenar y presentar fechas deberán definirse formalmente antes de implementar funcionalidades sensibles al tiempo.

## 23. API

El backend expondrá una API REST.

Los contratos se documentarán en:

    API-CONTRACT.md

Las rutas concretas no deben considerarse definitivas hasta que sean establecidas por las especificaciones correspondientes.

## 24. Seguridad

La arquitectura debe contemplar desde el inicio:

- Validación de entrada.
- Autenticación.
- Autorización.
- Aislamiento de tenants.
- Gestión de secretos.
- Rate limiting.
- CORS.
- Protección contra inyección.
- Logs seguros.
- Validación de recursos.
- Protección contra acceso horizontal indebido.

La seguridad debe aplicarse en backend aunque el frontend también realice validaciones.

## 25. Observabilidad

El sistema debe poder incorporar:

- Logs estructurados.
- Métricas.
- Trazabilidad de errores.
- Identificadores de correlación cuando corresponda.

La observabilidad debe permitir investigar problemas de reservas y disponibilidad sin registrar información sensible innecesaria.

## 26. Despliegue

La arquitectura inicial contempla:

    Frontend → Vercel
    Backend  → Render
    Database → Neon
    Automations → n8n

Estas tecnologías son decisiones de infraestructura iniciales y pueden cambiar mediante una decisión arquitectónica documentada.

## 27. Escalabilidad

La primera versión debe priorizar:

- Correctitud.
- Simplicidad.
- Mantenibilidad.
- Seguridad.

No se deben introducir microservicios solamente por anticipación de escala.

El backend debe mantener una estructura que permita evolucionar posteriormente si el crecimiento del sistema lo requiere.

## 28. Límites de responsabilidad

### Frontend

Responsable de presentación y experiencia de usuario.

### Backend

Responsable de reglas de negocio y consistencia.

### Base de datos

Responsable de persistencia e integridad de datos.

### IA

Responsable de asistencia y transformación de configuración cuando corresponda.

### n8n

Responsable de automatizaciones externas.

### Widget

Responsable de exponer la experiencia de reservas dentro de sitios externos.

Ningún componente debe asumir responsabilidades pertenecientes a otro sin una decisión explícita.

## 29. Flujo principal de una reserva

Conceptualmente:

    Cliente
      |
      v
    Frontend / Widget
      |
      v
    API
      |
      v
    Booking Engine
      |
      +--> Validar tenant
      +--> Validar servicio
      +--> Validar profesional
      +--> Calcular disponibilidad
      +--> Validar reglas
      +--> Validar concurrencia
      |
      v
    PostgreSQL
      |
      v
    Evento
      |
      v
    Automatización

La confirmación final siempre depende del backend.

## 30. Flujo de configuración

Conceptualmente:

    Profesional
        |
        v
    Wizard / IA
        |
        v
    Configuración estructurada
        |
        v
    Validación backend
        |
        v
    Persistencia
        |
        v
    Tenant configurado
        |
        +--> Página pública
        |
        +--> Widget

## 31. Evolución futura

La arquitectura debe permitir incorporar posteriormente:

- Múltiples sucursales.
- Múltiples calendarios.
- Recursos físicos.
- Pagos.
- Suscripciones.
- Recordatorios avanzados.
- Integraciones adicionales.
- Más canales de reserva.
- Aplicaciones móviles.
- Integraciones profundas con sitios externos.

Estas funcionalidades no forman parte del alcance inicial salvo que una especificación las incorpore explícitamente.

## 32. Regla arquitectónica principal

El sistema debe mantener una separación clara entre:

    Configuración
         ↓
    Booking Engine
         ↓
    Persistencia
         ↓
    Eventos
         ↓
    Integraciones

Las interfaces pueden cambiar.

Los canales de reserva pueden cambiar.

Las automatizaciones pueden cambiar.

Pero las reglas fundamentales de reserva deben permanecer centralizadas en el backend.
