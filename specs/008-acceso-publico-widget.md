# Especificación 008: Acceso público y widget de turnos

## 1. Objetivo

Definir cómo un negocio publicado ofrece su sistema de turnos a clientes mediante:

- una página pública generada por la plataforma; o
- un widget integrado en un sitio web existente.

Ambas modalidades deben utilizar el mismo backend, contrato público y Booking Engine.

## 2. Contexto

La plataforma debe funcionar sin exigir al negocio que tenga un sitio web propio.

Cuando el negocio no tiene sitio, la plataforma proporciona una página pública.

Cuando el negocio ya tiene un sitio, puede integrar el sistema mediante un widget.

La integración no debe generar un backend independiente por negocio.

## 3. Modalidades

### 3.1 Página pública

Cada tenant publicado dispone de una URL pública basada en su slug.

Conceptualmente:

~~~text
turnos.tuplataforma.com/{slug}
~~~

La página obtiene la configuración pública del negocio desde la API.

### 3.2 Widget

El administrador puede utilizar un widget para incorporar el flujo de turnos en un sitio existente.

El widget consume los mismos endpoints públicos que la página pública.

No debe existir una segunda implementación del Booking Engine para el widget.

## 4. Información pública

La API pública puede exponer únicamente la información necesaria para el flujo de reserva, por ejemplo:

- nombre comercial
- información pública del negocio
- servicios activos
- duración de los servicios
- profesionales disponibles cuando corresponda
- disponibilidad
- información necesaria para completar una reserva

No debe exponer:

- credenciales
- información administrativa
- datos privados de otros clientes
- configuración interna innecesaria
- información sensible de empleados

## 5. Flujo público

El flujo mínimo será:

~~~text
Cliente
  |
  v
Negocio publicado
  |
  v
Seleccionar servicio
  |
  v
Seleccionar fecha
  |
  v
Consultar disponibilidad
  |
  v
Seleccionar horario
  |
  v
Ingresar datos necesarios para reservar
  |
  v
Crear turno
  |
  v
Confirmación
~~~

La disponibilidad mostrada al cliente es informativa.

La creación del turno debe volver a validar la disponibilidad en el backend.

## 6. Tenant público

El tenant debe determinarse a partir del contexto público correspondiente, como el slug o la configuración del widget.

El cliente no debe poder utilizar este mecanismo para acceder a información privada de otro tenant.

La API debe validar que el tenant:

- existe
- está publicado
- está habilitado para recibir reservas

## 7. Widget

El widget debe poder integrarse sin requerir acceso al código interno del backend del negocio.

La integración debe limitarse al mecanismo público definido por la plataforma.

La implementación inicial puede utilizar aislamiento mediante iframe.

Una implementación posterior puede utilizar un componente JavaScript/Web Component si se necesita una integración visual más profunda.

La elección definitiva de tecnología del widget deberá registrarse como decisión cuando se implemente.

## 8. Seguridad del widget

El widget se considera una interfaz pública.

No debe contener:

- credenciales administrativas
- secretos del backend
- claves privadas
- tokens administrativos

Las operaciones administrativas no deben estar disponibles desde el widget público.

## 9. Reserva desde página pública o widget

La solicitud de creación debe utilizar el mismo endpoint de creación de turnos.

Conceptualmente:

~~~text
Página pública ──┐
                 ├──> API pública ──> Booking Engine ──> PostgreSQL
Widget ──────────┘
~~~

Esto garantiza que ambas modalidades respeten las mismas reglas de:

- duración
- horarios
- bloqueos
- profesionales
- conflictos
- concurrencia
- estado del tenant

## 10. Datos del cliente

El cliente no requiere una cuenta para realizar una reserva en el MVP.

El sistema solo debe solicitar los datos necesarios para gestionar el turno y las comunicaciones previstas.

La definición exacta de los datos obligatorios debe establecerse en el contrato de creación de turnos antes de la implementación.

No se debe crear una entidad de usuario autenticado para cada cliente como requisito del MVP.

## 11. Operaciones posteriores al turno

El acceso del cliente a operaciones posteriores, como cancelación o reprogramación, no se resuelve mediante una cuenta de cliente en esta especificación.

Antes de habilitar esas operaciones públicamente deberá definirse un mecanismo seguro de identificación del turno.

Posibles alternativas a evaluar:

- enlace privado
- token de gestión
- código de verificación

La elección queda pendiente y no debe asumirse automáticamente.

## 12. Negocio no publicado

Cuando un tenant no está publicado:

- la página pública no debe permitir nuevas reservas
- el widget no debe permitir nuevas reservas
- no debe exponerse configuración administrativa

El backend debe aplicar esta restricción aunque alguien intente llamar directamente a la API pública.

## 13. Negocio despublicado

Si un negocio publicado es despublicado:

- dejan de aceptarse nuevas reservas públicas
- el widget deja de aceptar nuevas reservas
- los turnos existentes no se eliminan automáticamente

El tratamiento de turnos futuros existentes se definirá mediante reglas específicas si resulta necesario.

## 14. Criterios de aceptación

### CA-01

Un tenant publicado puede ofrecer turnos mediante una página pública.

### CA-02

Un tenant publicado puede utilizar un widget para ofrecer turnos.

### CA-03

La página pública y el widget utilizan el mismo Booking Engine.

### CA-04

La API pública no expone información administrativa o sensible.

### CA-05

Un tenant no publicado no acepta reservas públicas.

### CA-06

Un tenant despublicado deja de aceptar nuevas reservas públicas.

### CA-07

El cliente puede iniciar una reserva sin crear una cuenta.

### CA-08

La creación del turno se revalida en backend aunque el cliente haya consultado disponibilidad previamente.

### CA-09

El widget no contiene credenciales administrativas ni secretos del backend.

### CA-10

Un cliente no puede utilizar parámetros públicos para acceder a información privada de otro tenant.

## 15. Casos límite

Deben contemplarse como mínimo:

- slug inexistente
- tenant no publicado
- tenant deshabilitado
- servicio inexistente
- servicio inactivo
- profesional inactivo
- disponibilidad vacía
- horario que deja de estar disponible entre consulta y confirmación
- negocio despublicado mientras un cliente está realizando una reserva
- widget utilizado con configuración inválida
- manipulación del identificador del tenant
- acceso directo a endpoints públicos no destinados a clientes

## 16. Pruebas

La implementación deberá incluir pruebas para:

- acceso a página pública publicada
- rechazo de página pública no publicada
- consulta pública de servicios
- consulta pública de disponibilidad
- creación pública de turno
- revalidación de disponibilidad durante creación
- acceso mediante widget
- aislamiento entre tenants
- protección de información administrativa
- comportamiento después de despublicar
- comportamiento con tenant inexistente
- comportamiento con tenant deshabilitado

## 17. Dependencias

Esta especificación depende de:

- `ARCHITECTURE.md`
- `API-CONTRACT.md`
- `DECISIONS.md`
- `specs/001-configuracion-tenant.md`
- `specs/002-disponibilidad.md`
- `specs/003-creacion-turno.md`
- `specs/007-publicacion-negocio.md`

## 18. Fuera de alcance

No se define aquí:

- diseño visual definitivo
- personalización avanzada del widget
- modificación automática del código de sitios existentes
- análisis automático mediante IA de sitios existentes
- cuentas de clientes
- pagos
- notificaciones
- dominios personalizados
- analítica avanzada
