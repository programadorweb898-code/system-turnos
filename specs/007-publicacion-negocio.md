# Especificación 007: Publicación del negocio

## 1. Objetivo

Definir cuándo un tenant puede pasar de un estado de configuración interna a estar disponible públicamente para recibir clientes mediante una página pública de turnos o un widget integrado en un sitio existente.

La publicación debe garantizar que el negocio tenga la configuración mínima necesaria para ofrecer turnos válidos.

## 2. Contexto

Cada negocio se configura desde el panel administrativo.

Mientras la configuración está incompleta, el negocio no debe quedar disponible para reservas públicas.

Una vez cumplidos los requisitos mínimos, el administrador puede publicar el negocio.

La publicación no modifica la lógica del Booking Engine. El Booking Engine continúa siendo la fuente de verdad para disponibilidad y creación de turnos.

## 3. Actores

### Administrador

Puede consultar el estado de publicación, completar la configuración, publicar y despublicar el negocio. Todas estas operaciones requieren autenticación y autorización.

### Cliente

Accede únicamente a la representación pública de un negocio publicado. No puede publicar ni modificar el estado del negocio.

## 4. Estados del negocio

El tenant tendrá conceptualmente un estado de publicación:

- `draft`: configuración interna, no disponible públicamente.
- `published`: disponible públicamente.
- `unpublished`: anteriormente publicado, actualmente retirado de la disponibilidad pública.

El estado operativo del tenant debe mantenerse conceptualmente separado del estado de publicación.

## 5. Requisitos mínimos para publicar

Antes de permitir la publicación, el backend debe verificar como mínimo:

- nombre comercial
- slug público válido
- timezone válida
- al menos un servicio activo con duración positiva
- al menos un profesional activo
- al menos un profesional activo asignado a al menos un servicio activo
- configuración válida de horarios de atención
- consistencia general de la configuración necesaria para reservar

La validación debe realizarse en backend.

### 5.1 Asignación profesional-servicio como requisito de publicación

Publicar un negocio cuyos profesionales no pueden atender ningún servicio produce un negocio público sin disponibilidad posible.

Por lo tanto, la publicación exige que exista al menos una relación activa que permita reservar:

- un servicio con `status = active` y `duration > 0`;
- un profesional con `status = active`;
- una asignación vigente entre ese profesional y ese servicio.

Esta regla se deriva de la Especificación 013 (asignación de servicios a profesionales) y aplica el principio de que el negocio publicado debe poder recibir reservas válidas.

No es necesario que todos los profesionales estén asignados ni que todos los servicios tengan profesionales. Basta con que exista al menos una combinación elegible.

## 6. Publicación

La publicación debe ser una operación explícita del administrador.

Flujo:

~~~text
Administrador
     |
     v
Solicitar publicación
     |
     v
Backend
     |
     +--> validar configuración
     |
     +--> configuración incompleta --> rechazo
     |
     v
Publicar tenant
~~~

Completar un formulario no debe publicar automáticamente el negocio.

## 7. Configuración incompleta

Si no se cumplen los requisitos, el backend debe rechazar la publicación e indicar qué requisitos faltan.

Ejemplo conceptual:

~~~json
{
  "error": {
    "code": "TENANT_NOT_READY",
    "message": "El negocio todavía no está listo para publicarse.",
    "details": [
      "Debe existir al menos un servicio activo.",
      "Debe existir al menos un profesional activo.",
      "Debe configurarse el horario de atención."
    ]
  }
}
~~~

El formato definitivo debe respetar `API-CONTRACT.md`.

El envelope de error admite un campo `details` opcional de tipo arreglo de cadenas para enumerar los requisitos faltantes:

~~~json
{
  "error": {
    "code": "TENANT_NOT_READY",
    "message": "El negocio todavía no está listo para publicarse.",
    "details": [
      "Debe existir al menos un servicio activo.",
      "Debe existir al menos un profesional activo asignado a un servicio activo."
    ]
  }
}
~~~

`details` es opcional y solo se utiliza cuando el backend puede determinar qué requisitos concretos faltan.

## 8. Página pública

Cuando el tenant está publicado, la plataforma puede exponer una página pública asociada a su slug.

Conceptualmente:

~~~text
turnos.tuplataforma.com/{slug}
~~~

La URL exacta queda pendiente de infraestructura y dominio del producto.

La página pública debe:

- identificar el negocio
- mostrar servicios disponibles
- permitir seleccionar un servicio
- mostrar disponibilidad
- permitir solicitar un turno

No debe exponer información administrativa o privada.

## 9. Widget

Un tenant publicado también puede utilizarse mediante el widget para integrarlo en un sitio existente.

El widget y la página pública deben consumir los mismos recursos públicos y el mismo Booking Engine.

~~~text
Página pública ──┐
                 ├──> API ──> Booking Engine
Widget ──────────┘
~~~

La existencia de un sitio web propio no es requisito para publicar.

## 10. Despublicación

Un administrador autorizado puede retirar un negocio de la disponibilidad pública.

Al despublicar:

- la página pública deja de permitir nuevas reservas
- el widget deja de permitir nuevas reservas
- los datos históricos no se eliminan
- los turnos existentes no se eliminan automáticamente

La política para turnos futuros existentes deberá definirse en una especificación específica si resulta necesaria.

## 11. Cambios después de publicar

Un negocio publicado puede continuar modificando su configuración.

El backend debe impedir que una modificación produzca una configuración inválida para nuevas reservas.

Los cambios que afecten disponibilidad deberán ser validados por las reglas correspondientes del Booking Engine.

Ejemplos:

- modificar horarios
- desactivar un servicio
- desactivar un profesional
- crear un bloqueo

No se deben cancelar ni modificar automáticamente turnos existentes sin una regla de negocio explícita.

## 12. Seguridad

Publicar y despublicar son operaciones administrativas.

Deben requerir:

- administrador autenticado
- autorización válida
- tenant correctamente identificado

El cliente público nunca puede cambiar el estado de publicación.

El backend no debe confiar en un `tenant_id` enviado arbitrariamente por el cliente.

## 13. API

Las rutas definitivas son:

~~~text
POST /api/v1/admin/tenant/publication
POST /api/v1/admin/tenant/unpublication
~~~

Ambas requieren autenticación y autorización administrativa.

El backend obtiene el tenant desde el contexto autenticado. Un `tenantId` enviado por el cliente nunca puede utilizarse para seleccionar el tenant de la operación.

Se adopta el namespace `/api/v1/admin/**` vigente en `API-CONTRACT.md` y en el resto del backend.

### 13.1 Publicación

`POST /api/v1/admin/tenant/publication`

El backend valida los requisitos mínimos y, si se cumplen, cambia el estado del tenant a `published`.

Respuesta `200`:

~~~json
{
  "status": "published"
}
~~~

### 13.2 Publicación repetida

Publicar un tenant que ya se encuentra en `published` es idempotente.

El backend vuelve a validar los requisitos mínimos y responde `200` con el estado actual. No devuelve error.

La revalidación permite detectar una configuración que se volvió inválida después de la publicación.

### 13.3 Despublicación

`POST /api/v1/admin/tenant/unpublication`

Cambia el estado del tenant a `unpublished` y responde `200` con el estado resultante.

Despublicar un tenant que ya se encuentra en `unpublished` es idempotente y responde `200`.

Despublicar no elimina turnos existentes ni historial.

### 13.4 Tenant no listo

Cuando la configuración no cumple los requisitos mínimos, el backend responde `409 Conflict` con código `TENANT_NOT_READY` y el campo `details` opcional.

## 14. Criterios de aceptación

### CA-01

Un tenant nuevo no está disponible públicamente por defecto.

### CA-02

Un tenant no puede publicarse si no cumple los requisitos mínimos.

### CA-03

Solo un administrador autorizado puede publicar un tenant.

### CA-04

Una vez publicado, la página pública puede utilizarse para iniciar el flujo de reserva.

### CA-05

Un tenant publicado utiliza el Booking Engine para determinar disponibilidad.

### CA-06

El widget utiliza la misma lógica de disponibilidad que la página pública.

### CA-07

Un administrador puede despublicar el negocio.

### CA-08

Despublicar impide nuevas reservas públicas sin eliminar automáticamente el historial de turnos.

### CA-09

La publicación o despublicación no permite acceder ni modificar información de otro tenant.

### CA-10

El frontend no puede publicar un tenant sin pasar por la validación del backend.

### CA-11

Un tenant sin al menos un profesional activo asignado a un servicio activo no puede publicarse.

### CA-12

Publicar o despublicar un tenant que ya se encuentra en el estado solicitado responde `200` con el estado actual y vuelve a validar la configuración en el caso de la publicación.

## 15. Casos límite

Deben contemplarse como mínimo:

- intento de publicar sin servicios
- intento de publicar sin profesionales
- intento de publicar sin asignación profesional-servicio
- intento de publicar con asignación de profesional inactivo
- intento de publicar con asignación de servicio inactivo
- intento de publicar sin horarios
- servicio inactivo
- profesional inactivo
- tenant deshabilitado
- slug inválido o no disponible
- publicación repetida
- despublicación repetida
- acceso público a tenant no publicado
- acceso público a tenant inexistente
- intento de publicar otro tenant
- cambios de configuración mientras existen turnos futuros

## 16. Pruebas

La implementación deberá incluir pruebas para:

- publicación con configuración válida
- publicación con configuración incompleta
- publicación sin autorización
- aislamiento entre tenants
- acceso público a tenant publicado
- rechazo de acceso público a tenant no publicado
- despublicación
- despublicación sin autorización
- publicación repetida
- despublicación repetida
- utilización del Booking Engine desde página pública
- utilización del Booking Engine desde widget

## 17. Dependencias

Esta especificación depende de:

- `ARCHITECTURE.md`
- `API-CONTRACT.md`
- `DECISIONS.md`
- `specs/001-configuracion-tenant.md`
- `specs/002-disponibilidad.md`
- `specs/003-creacion-turno.md`
- `specs/006-autenticacion-autorizacion.md`

## 18. Fuera de alcance

Esta especificación no define:

- dominios personalizados
- certificados SSL
- despliegue de infraestructura
- personalización visual avanzada
- análisis automático de sitios existentes
- modificación automática del código de sitios existentes
- pagos
- suscripciones SaaS
- notificaciones
- SEO avanzado
- analítica
