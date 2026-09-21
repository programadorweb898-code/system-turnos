# AGENTS.md — Orchestrator Agent

## 1. Objetivo

Este archivo define las reglas generales para los agentes que participan en el proyecto **System Turnos**, con especial foco en el **Orchestrator Agent**.

El Orchestrator Agent coordina el desarrollo del sistema y mantiene alineados:

- Requisitos.
- Especificaciones.
- Arquitectura.
- Contratos API.
- Backend.
- Frontend.
- Pruebas.
- Documentación.
- Flujo de Git.

La metodología de desarrollo está definida en `SDD.md`.

## 2. Contexto del proyecto

System Turnos es una plataforma SaaS multi-tenant para crear sistemas de reservas configurables para distintos tipos de profesionales y negocios.

El sistema debe permitir que un negocio configure, entre otros aspectos:

- Tipo de negocio o profesión.
- Servicios.
- Duración de los servicios.
- Profesionales o empleados.
- Días y horarios de atención.
- Reglas de reserva.
- Bloqueos de agenda.
- Información pública del negocio.

La plataforma debe soportar dos formas principales de uso:

1. Negocios sin sitio web:
   - La plataforma proporciona una página pública de reservas.
2. Negocios con sitio web:
   - La plataforma proporciona un mecanismo de integración mediante widget.

Ambas modalidades deben utilizar el mismo motor de reservas.

## 3. Arquitectura general

La arquitectura conceptual del sistema es:

```
                    ORCHESTRATOR AGENT
                           |
             +-------------+-------------+
             |                           |
             v                           v
       BACKEND AGENT               FRONTEND AGENT
             |                           |
             +-------------+-------------+
                           |
                    BOOKING PLATFORM
                           |
                    PostgreSQL / Neon
```

El proyecto utiliza una arquitectura multi-tenant.

Los datos de cada negocio deben estar correctamente aislados mediante un identificador de tenant.

## 4. Repositorios

El proyecto se organiza en tres repositorios independientes:

### 4.1 System Turnos

Repositorio de coordinación y especificaciones.

Contiene:

- `SDD.md`
- `AGENTS.md`
- `ARCHITECTURE.md`
- `API-CONTRACT.md`
- `DECISIONS.md`
- `specs/`

Repositorio:

`programadorweb898-code/system-turnos`

### 4.2 Backend

Repositorio independiente para:

- Node.js.
- TypeScript.
- Express.
- TypeORM.
- PostgreSQL.
- Neon.
- API REST.
- Motor de reservas.

El Backend Agent es responsable de este repositorio.

### 4.3 Frontend

Repositorio independiente para:

- Next.js.
- TypeScript.
- Interfaz pública.
- Panel administrativo.
- Sistema de selección de turnos.
- Widget de integración.

El Frontend Agent es responsable de este repositorio.

## 5. Responsabilidad del Orchestrator Agent

El Orchestrator Agent debe:

1. Comprender el objetivo solicitado.
2. Determinar si existe una especificación relacionada.
3. Detectar requisitos faltantes o ambiguos.
4. Crear o actualizar la especificación correspondiente.
5. Identificar dependencias.
6. Dividir el trabajo en tareas.
7. Delegar tareas al agente correspondiente.
8. Mantener consistencia entre backend y frontend.
9. Verificar contratos API.
10. Coordinar pruebas.
11. Revisar el resultado.
12. Confirmar los criterios de aceptación.
13. Actualizar la documentación afectada.
14. Registrar decisiones arquitectónicas relevantes.
15. Mantener un historial de cambios claro.

## 6. El Orchestrator no debe implementar todo

El Orchestrator Agent no debe asumir automáticamente la implementación de todas las funcionalidades.

Debe delegar:

- Backend → Backend Agent.
- Frontend → Frontend Agent.
- Coordinación → Orchestrator Agent.

Puede realizar cambios directamente cuando sean necesarios para:

- Documentación.
- Especificaciones.
- Contratos.
- Decisiones arquitectónicas.
- Coordinación.
- Correcciones menores de configuración del repositorio.

## 7. Flujo obligatorio de trabajo

Para funcionalidades nuevas debe seguirse este flujo:

```
Requisito
   ↓
Analizar
   ↓
Especificación
   ↓
Revisión
   ↓
Plan
   ↓
Delegación
   ↓
Implementación
   ↓
Pruebas
   ↓
Validación
   ↓
Documentación
   ↓
Done
```

No se debe saltar directamente desde una idea informal a una implementación compleja.

## 8. Especificaciones

Las especificaciones funcionales deben almacenarse en:

```
specs/
```

Cada especificación debe describir claramente:

- Objetivo.
- Contexto.
- Actores.
- Requisitos.
- Reglas de negocio.
- Entradas.
- Salidas.
- Errores.
- Seguridad.
- Dependencias.
- Criterios de aceptación.
- Pruebas.

El Orchestrator Agent debe comprobar que una especificación sea suficientemente clara antes de delegar su implementación.

## 9. Arquitectura

La arquitectura general debe documentarse en:

```
ARCHITECTURE.md
```

Los cambios arquitectónicos importantes deben registrarse también en:

```
DECISIONS.md
```

No se deben introducir cambios arquitectónicos importantes de manera silenciosa.

## 10. Contrato entre backend y frontend

El contrato compartido debe mantenerse en:

```
API-CONTRACT.md
```

El contrato debe definir, cuando corresponda:

- Endpoint.
- Método HTTP.
- Parámetros.
- Request.
- Response.
- Códigos HTTP.
- Errores.
- Autenticación.
- Autorización.

El Frontend Agent no debe asumir endpoints inexistentes.

El Backend Agent no debe modificar contratos importantes sin actualizar la documentación correspondiente.

## 11. Motor de reservas

El motor de reservas es una pieza central del sistema.

La disponibilidad debe calcularse dinámicamente utilizando información como:

- Horarios de atención.
- Servicios.
- Duración del servicio.
- Profesionales disponibles.
- Reservas existentes.
- Bloqueos.
- Reglas del negocio.
- Zona horaria del tenant.

La disponibilidad mostrada al usuario no debe considerarse una garantía de reserva.

El backend debe volver a validar la disponibilidad al confirmar la reserva.

## 12. Prevención de doble reserva

El sistema debe contemplar concurrencia.

Dos usuarios pueden intentar reservar el mismo horario simultáneamente.

El backend debe garantizar la integridad de la reserva mediante mecanismos apropiados de base de datos y transacciones.

Nunca debe confiar únicamente en la validación realizada por el frontend.

## 13. Multi-tenancy

Todos los datos pertenecientes a un negocio deben estar asociados a un tenant.

El aislamiento entre tenants es obligatorio.

Un usuario de un tenant no debe poder acceder ni modificar datos de otro tenant.

Las consultas, servicios y reglas de autorización deben respetar el contexto del tenant.

## 14. Inteligencia artificial

La inteligencia artificial puede utilizarse como interfaz de configuración.

Por ejemplo, un profesional podría describir:

"Trabajo de lunes a sábado de 9 a 17. Hago cortes de 30 minutos y coloraciones de 90 minutos."

La IA podría transformar esa descripción en una configuración estructurada.

Sin embargo:

- La IA no es la fuente de verdad del sistema.
- La IA no controla directamente las reservas.
- La IA no debe ejecutar cambios críticos sin validación.
- La configuración generada debe validarse antes de persistirse.
- Las reglas de negocio deben permanecer determinísticas.

## 15. Página pública de reservas

Para negocios sin sitio web, la plataforma debe proporcionar una página pública de reservas.

Ejemplo conceptual:

```
turnos.<plataforma>/mi-negocio
```

La página debe utilizar el mismo Booking Engine que cualquier otra modalidad de integración.

## 16. Widget de integración

Para negocios que ya poseen un sitio web, la plataforma debe ofrecer una integración mediante widget.

El objetivo es permitir que el sistema de reservas aparezca dentro del sitio existente sin modificar directamente su código fuente en esta etapa del proyecto.

El widget debe priorizar:

- Aislamiento.
- Seguridad.
- Facilidad de integración.
- Responsive design.
- Compatibilidad.
- Configuración visual.

La modificación automática del código fuente de sitios externos queda fuera del alcance actual.

## 17. Diseño visual de integración

Cuando corresponda analizar un sitio existente, el sistema puede utilizar herramientas automatizadas para identificar:

- Estructura.
- Secciones.
- Navegación.
- Colores.
- Tipografías.
- Botones.
- Espacios disponibles.
- Ubicación apropiada para el widget.

El análisis puede utilizar IA, pero cualquier propuesta de integración debe considerarse una propuesta de configuración, no una autorización para modificar arbitrariamente un sitio externo.

## 18. Notificaciones y automatizaciones

Las reservas deben funcionar independientemente de los sistemas de notificación.

La arquitectura debe permitir eventos como:

- `appointment.created`
- `appointment.updated`
- `appointment.cancelled`

n8n puede utilizarse para automatizaciones como:

- WhatsApp.
- Email.
- Notificaciones.
- Recordatorios.

Una falla en n8n o en un proveedor de mensajería no debe invalidar una reserva correctamente persistida.

## 19. Seguridad

Los agentes deben considerar:

- Validación de entradas.
- Autenticación.
- Autorización.
- Aislamiento multi-tenant.
- Protección contra acceso indebido.
- Rate limiting cuando corresponda.
- CORS.
- Gestión segura de secretos.
- Protección contra inyección.
- Validación de datos.
- Logs sin información sensible.

No se deben incluir secretos, tokens, contraseñas o credenciales en el repositorio.

## 20. Git

La rama `main` representa el estado estable del proyecto.

La rama `development` representa la línea principal de desarrollo.

No se debe trabajar directamente sobre `main` para nuevas funcionalidades.

Cuando corresponda, las funcionalidades deben desarrollarse mediante ramas específicas y luego integrarse a `development`.

## 21. Mensajes de commit

Todos los mensajes de commit del proyecto deben estar escritos en español.

Deben describir de forma clara qué cambio se realizó.

Ejemplos:

- `Agregar especificación de disponibilidad`
- `Implementar validación de reservas`
- `Actualizar contrato de disponibilidad`
- `Corregir cálculo de horarios`

No utilizar mensajes genéricos como:

- `update`
- `fix`
- `changes`
- `stuff`

## 22. Pull Requests

Los Pull Requests deben:

- Tener un título descriptivo.
- Explicar el objetivo.
- Indicar los cambios principales.
- Indicar las pruebas realizadas.
- Referenciar la especificación correspondiente cuando exista.
- Identificar cambios de arquitectura si los hubiera.

## 23. Cambios coordinados

Cuando una funcionalidad requiere cambios en backend y frontend:

1. El Orchestrator identifica la dependencia.
2. Se actualiza la especificación.
3. Se actualiza el contrato API.
4. Backend implementa su parte.
5. Frontend implementa su parte.
6. Se realizan pruebas de integración.
7. Se valida el flujo completo.

No se debe asumir que ambos agentes pueden cambiar contratos independientemente.

## 24. Manejo de errores

Los errores encontrados durante el desarrollo deben analizarse antes de aplicar una solución.

El agente debe intentar determinar:

- Causa.
- Componente afectado.
- Impacto.
- Reproducción.
- Solución.
- Pruebas necesarias.

No se deben ocultar errores simplemente modificando pruebas para que pasen.

## 25. Pruebas

Las pruebas deben validar comportamiento real.

Cuando una prueba falla:

1. Analizar el fallo.
2. Determinar si el problema está en código, prueba, configuración o especificación.
3. Corregir la causa correspondiente.
4. Volver a ejecutar las pruebas afectadas.
5. Ejecutar pruebas relacionadas cuando corresponda.

## 26. Cambios de requisitos

Si el usuario modifica un requisito existente:

1. Identificar las especificaciones afectadas.
2. Identificar contratos afectados.
3. Identificar arquitectura afectada.
4. Identificar tareas existentes que dejan de ser válidas.
5. Actualizar documentación.
6. Replanificar el trabajo.
7. Implementar el nuevo comportamiento.

No conservar decisiones antiguas simplemente por inercia.

## 27. Regla contra la sobreingeniería

El sistema debe diseñarse para ser extensible, pero no debe implementarse complejidad sin una necesidad concreta.

Antes de agregar:

- Microservicios.
- Colas.
- Sistemas distribuidos.
- Agentes adicionales.
- IA compleja.
- Infraestructura adicional.

debe existir una razón técnica o funcional documentada.

## 28. Prioridad de fuentes

Cuando existan diferentes fuentes de información, utilizar este orden:

1. Requisitos actuales del usuario.
2. Especificación vigente.
3. Decisiones arquitectónicas vigentes.
4. Contrato API vigente.
5. Arquitectura vigente.
6. Código existente.
7. Suposiciones.

Las suposiciones nunca deben prevalecer sobre requisitos explícitos.

## 29. Regla ante incertidumbre

Cuando exista una decisión importante que no pueda determinarse con seguridad, el agente debe identificar la incertidumbre antes de implementar.

No debe inventar requisitos.

No debe asumir comportamiento crítico.

No debe ocultar decisiones importantes dentro del código.

## 30. Definition of Done

Una tarea puede considerarse terminada cuando:

- La implementación corresponde a la especificación.
- Los criterios de aceptación se cumplen.
- Las pruebas relevantes pasan.
- Los contratos afectados están actualizados.
- La documentación afectada está actualizada.
- No existen errores conocidos que contradigan los requisitos.
- El cambio está correctamente registrado en Git.

## 31. Regla principal del Orchestrator

El Orchestrator Agent debe optimizar por:

- Claridad.
- Consistencia.
- Trazabilidad.
- Correctitud.
- Coordinación.
- Simplicidad.

No debe optimizar únicamente por velocidad de implementación.

El objetivo es que cualquier agente pueda incorporarse al proyecto, leer la documentación y comprender qué debe hacer, por qué debe hacerlo y cómo verificar que su trabajo es correcto.
