# SDD.md — Specification-Driven Development

## 1. Objetivo

Este proyecto utiliza Specification-Driven Development (SDD) como metodología principal de desarrollo.

El objetivo es que las funcionalidades se definan mediante especificaciones claras y verificables antes de su implementación.

El flujo general es:

Idea → Especificación → Revisión → Plan → Implementación → Pruebas → Validación → Done

## 2. Principio fundamental

Ningún agente debe implementar una funcionalidad importante basándose únicamente en una descripción informal si existen requisitos ambiguos, incompletos o contradictorios.

La especificación correspondiente debe ser la fuente de verdad para la implementación.

## 3. Fuente de verdad

La documentación del proyecto se organiza de la siguiente manera:

- SDD.md: define el proceso de desarrollo.
- AGENTS.md: define las responsabilidades y reglas generales de los agentes.
- ARCHITECTURE.md: define la arquitectura del sistema.
- API-CONTRACT.md: define los contratos entre backend y frontend.
- DECISIONS.md: registra decisiones arquitectónicas relevantes.
- specs/: contiene las especificaciones funcionales concretas.

Cuando exista una contradicción, los agentes deben identificarla y detener la implementación afectada hasta resolverla.

## 4. Responsabilidades del Orchestrator Agent

El Orchestrator Agent coordina el proyecto completo.

Sus responsabilidades incluyen:

- Comprender los requisitos.
- Detectar ambigüedades.
- Convertir requisitos en especificaciones.
- Dividir funcionalidades en tareas.
- Determinar dependencias entre tareas.
- Coordinar Backend Agent y Frontend Agent.
- Mantener consistencia entre contratos y especificaciones.
- Revisar cambios importantes.
- Coordinar pruebas y validación.
- Mantener la documentación actualizada.
- Supervisar el flujo de Git.
- Evitar implementaciones duplicadas o incompatibles.

El Orchestrator Agent no debe implementar por defecto todo el sistema por sí mismo. Debe delegar la implementación especializada cuando corresponda.

## 5. Responsabilidades del Backend Agent

El Backend Agent es responsable de la implementación backend.

Su ámbito incluye:

- Node.js.
- TypeScript.
- Express.
- PostgreSQL.
- Neon.
- TypeORM.
- Migraciones.
- Entidades y relaciones.
- Motor de disponibilidad.
- Reservas.
- Prevención de doble reserva.
- Multi-tenancy.
- API REST.
- Validaciones.
- Seguridad backend.
- Pruebas backend.

Debe implementar exclusivamente lo definido por las especificaciones y contratos vigentes.

## 6. Responsabilidades del Frontend Agent

El Frontend Agent es responsable de la implementación frontend.

Su ámbito incluye:

- Next.js.
- TypeScript.
- UI/UX.
- Página pública de reservas.
- Panel de administración.
- Calendario.
- Selección de horarios.
- Widget de integración.
- Responsive design.
- Accesibilidad.
- Pruebas frontend.

Debe consumir los contratos definidos para el backend y no asumir endpoints o estructuras que no estén documentados.

## 7. Estructura de una especificación

Cada especificación funcional debe definir, cuando corresponda:

1. Objetivo.
2. Contexto.
3. Actores.
4. Requisitos funcionales.
5. Reglas de negocio.
6. Entradas.
7. Salidas.
8. Errores esperados.
9. Seguridad.
10. Dependencias.
11. Cambios de datos.
12. Contratos API.
13. Criterios de aceptación.
14. Casos límite.
15. Pruebas requeridas.

## 8. Criterios de aceptación

Una funcionalidad no se considera terminada únicamente porque el código compile o porque el flujo principal funcione.

Debe cumplir los criterios de aceptación definidos en su especificación.

Los criterios deben ser observables y verificables.

## 9. Pruebas

Cada especificación debe identificar las pruebas necesarias.

Dependiendo de la funcionalidad pueden incluir:

- Pruebas unitarias.
- Pruebas de integración.
- Pruebas de API.
- Pruebas de concurrencia.
- Pruebas de validación.
- Pruebas frontend.
- Pruebas end-to-end.

Las pruebas deben cubrir tanto el comportamiento esperado como los casos límite relevantes.

## 10. Coordinación entre backend y frontend

Backend y frontend deben compartir contratos explícitos.

Antes de implementar una integración entre ambos agentes deben estar definidos, cuando corresponda:

- Endpoint.
- Método HTTP.
- Parámetros.
- Request body.
- Response body.
- Códigos HTTP.
- Errores.
- Reglas de autenticación.
- Reglas de autorización.

El frontend no debe inventar endpoints.

El backend no debe modificar contratos consumidos por el frontend sin actualizar la documentación correspondiente y coordinar el cambio.

## 11. Cambios en una especificación

Si durante la implementación se descubre que una especificación es incorrecta o incompleta:

1. Detener la parte afectada de la implementación.
2. Identificar el problema.
3. Proponer el cambio.
4. Actualizar la especificación.
5. Actualizar contratos o arquitectura afectados.
6. Revisar dependencias.
7. Continuar la implementación con la nueva versión.

No se deben introducir cambios silenciosos de requisitos.

## 12. Ambigüedades

Cuando una decisión de negocio o técnica no pueda deducirse razonablemente de la documentación existente, el agente debe señalar la ambigüedad.

No debe inventar una regla que pueda modificar el comportamiento del producto.

## 13. Dependencias

Antes de implementar una funcionalidad, el Orchestrator Agent debe identificar sus dependencias.

Una tarea que dependa de otra funcionalidad incompleta no debe considerarse lista para validación final.

## 14. Validación final

Antes de marcar una especificación como terminada se debe comprobar:

- Implementación completa.
- Pruebas ejecutadas.
- Criterios de aceptación cumplidos.
- Contratos actualizados.
- Documentación actualizada.
- No existen errores conocidos que contradigan la especificación.
- Backend y frontend mantienen compatibilidad.

## 15. Cambios arquitectónicos

Los cambios que afecten significativamente la arquitectura deben registrarse en DECISIONS.md.

Ejemplos:

- Cambio de tecnología.
- Cambio de patrón arquitectónico.
- Cambio de estrategia de persistencia.
- Cambio del modelo multi-tenant.
- Cambio de estrategia de integración.
- Cambio de contrato importante entre servicios.

## 16. Definition of Done

Una especificación se considera Done cuando:

- Todos sus requisitos están implementados.
- Todos sus criterios de aceptación están satisfechos.
- Las pruebas relevantes pasan.
- Las integraciones afectadas fueron verificadas.
- La documentación está actualizada.
- No existen dependencias pendientes que impidan utilizar la funcionalidad.
- El Orchestrator Agent valida el resultado final.

## 17. Regla general

La velocidad de implementación no debe reemplazar la claridad de los requisitos.

Primero se define qué debe hacer el sistema.

Después se define cómo se implementará.

Finalmente se implementa, prueba y valida.

El objetivo de SDD es reducir ambigüedades, evitar trabajo duplicado y mantener coordinados a todos los agentes y componentes del sistema.
