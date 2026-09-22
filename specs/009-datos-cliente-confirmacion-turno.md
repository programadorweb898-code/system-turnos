# Especificación 009: Datos del cliente y confirmación del turno

## 1. Objetivo

Definir los datos que debe proporcionar un cliente para crear un turno y cómo se confirma la reserva.

El cliente no requiere una cuenta autenticada en el MVP.

## 2. Contexto

El cliente accede mediante la página pública o el widget.

Después de seleccionar servicio, fecha, horario y profesional cuando corresponda, debe proporcionar sus datos y, opcionalmente, información adicional antes de confirmar la reserva.

El backend debe validar los datos y confirmar el turno únicamente después de verificar nuevamente la disponibilidad.

## 3. Datos del cliente

El formulario de reserva del MVP tendrá exactamente estos campos:

### 3.1 Nombre y apellido

Campo obligatorio.

Debe permitir identificar al cliente asociado al turno.

### 3.2 Teléfono

Campo obligatorio.

El número de teléfono será además el medio utilizado para enviar la confirmación del turno mediante WhatsApp.

El backend debe validar que el valor tenga un formato aceptable antes de crear el turno.

### 3.3 Información adicional

Campo opcional de tipo textarea.

El cliente puede utilizarlo para enviar:

- sugerencias
- pedidos
- información adicional
- aclaraciones relacionadas con el turno

Límite máximo:

**300 caracteres.**

El backend debe validar el límite de 300 caracteres independientemente de la validación realizada por el frontend.

No debe utilizarse este campo como mecanismo de autenticación.

## 4. Cliente sin cuenta

El cliente no se convierte en un usuario autenticado del sistema por realizar una reserva.

Por lo tanto:

- no necesita contraseña
- no necesita iniciar sesión
- no necesita crear una cuenta
- sus datos no deben utilizarse como mecanismo de autenticación

La identidad del cliente dentro del turno representa información de contacto, no una identidad autenticada.

## 5. Flujo de reserva

El flujo conceptual será:

~~~text
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
Ingresar:
  - Nombre y apellido
  - Teléfono
  - Información adicional (opcional, máximo 300)
       |
       v
Solicitar reserva
       |
       v
Backend
       |
       +--> validar tenant
       +--> validar servicio
       +--> validar profesional
       +--> validar datos
       +--> recalcular/revalidar disponibilidad
       +--> proteger contra concurrencia
       |
       v
Persistir turno
       |
       v
Confirmar reserva
       |
       v
Generar evento appointment.created
       |
       v
Enviar confirmación por WhatsApp
~~~

La creación del turno no debe depender de que WhatsApp haya enviado correctamente el mensaje.

## 6. Confirmación

Una reserva solo se considera confirmada cuando el backend la persiste correctamente.

La respuesta exitosa debe proporcionar al cliente información suficiente para reconocer su turno.

Como mínimo:

- identificador público o referencia del turno
- negocio
- servicio
- fecha
- hora de inicio
- hora de finalización
- profesional, si corresponde
- estado del turno

No debe exponerse un identificador interno si puede utilizarse un identificador público separado.

## 7. Confirmación mediante WhatsApp

La confirmación del turno será enviada al número de teléfono proporcionado por el cliente mediante WhatsApp.

El envío se realizará después de que el turno haya sido creado correctamente.

Conceptualmente:

~~~text
Turno creado
    |
    v
appointment.created
    |
    v
Automatización
    |
    v
WhatsApp
    |
    v
Cliente
~~~

El proveedor concreto de WhatsApp y su implementación quedan fuera de esta especificación.

Si el envío falla:

- el turno no debe cancelarse automáticamente
- el turno debe continuar confirmado
- el fallo debe poder registrarse para su posterior tratamiento

n8n puede encargarse de la automatización, pero no debe ser necesario para confirmar la reserva.

## 8. Estado de la reserva

El turno creado debe tener un estado que permita distinguir una reserva confirmada de otros estados futuros.

Para el MVP, la creación exitosa debe producir un estado equivalente a:

~~~text
confirmed
~~~

Los estados adicionales deberán definirse según las necesidades del dominio.

## 9. Respuesta ante conflicto

Si el horario deja de estar disponible antes de confirmar:

- no se crea el turno
- el backend responde con conflicto
- el cliente debe poder seleccionar otra disponibilidad

La respuesta debe utilizar el contrato de error establecido.

Conceptualmente:

~~~json
{
  "error": {
    "code": "APPOINTMENT_CONFLICT",
    "message": "El horario seleccionado ya no está disponible."
  }
}
~~~

HTTP:

~~~text
409 Conflict
~~~

## 10. Idempotencia

La creación de un turno debe contemplar solicitudes repetidas.

Esto es especialmente importante si:

- el cliente pulsa varias veces el botón de confirmar
- existe una repetición de la solicitud HTTP
- el navegador reintenta una solicitud
- existe una interrupción durante la respuesta

La implementación deberá utilizar un mecanismo de idempotencia apropiado.

Una misma operación de reserva no debe crear múltiples turnos por una repetición accidental de la solicitud.

## 11. Privacidad

Los datos del cliente deben tratarse como información privada.

La API pública no debe permitir consultar libremente los datos de clientes de otros turnos.

Una respuesta pública de disponibilidad nunca debe revelar:

- nombres de clientes
- teléfonos
- información adicional
- información de otros turnos que permita identificar personas

## 12. Evento de creación y automatización

Una reserva confirmada debe generar el evento:

~~~text
appointment.created
~~~

El evento podrá ser procesado por el sistema de automatización para enviar la confirmación por WhatsApp.

El evento no debe utilizarse como condición para decidir si el turno fue creado.

Si n8n, WhatsApp o el proveedor de mensajería están temporalmente caídos, el turno debe seguir existiendo y permanecer confirmado.

## 13. Gestión posterior del turno

El cliente no tendrá cuenta en el MVP.

Por lo tanto, esta especificación no define todavía cómo podrá:

- cancelar
- reprogramar
- consultar nuevamente
- modificar sus datos

Antes de exponer esas operaciones públicamente deberá definirse un mecanismo seguro de acceso al turno, como un enlace privado o token.

## 14. Datos sensibles y notas

El campo de información adicional está limitado a 300 caracteres y debe utilizarse únicamente para información relacionada con el turno.

El backend debe validar longitud y formato.

No se deben agregar campos adicionales al formulario del cliente sin modificar esta especificación o crear una nueva decisión documentada.

## 15. Criterios de aceptación

### CA-01

Un cliente puede solicitar un turno sin crear una cuenta.

### CA-02

El formulario exige nombre y apellido.

### CA-03

El formulario exige teléfono.

### CA-04

El formulario permite información adicional opcional mediante un textarea.

### CA-05

La información adicional no puede superar los 300 caracteres.

### CA-06

El backend valida todos los campos obligatorios y el límite de 300 caracteres.

### CA-07

El backend vuelve a validar disponibilidad antes de confirmar.

### CA-08

Un conflicto de disponibilidad devuelve HTTP 409 y no crea un turno.

### CA-09

Una solicitud repetida no crea accidentalmente múltiples turnos cuando corresponde a la misma operación.

### CA-10

Una reserva solo se considera confirmada después de persistirse correctamente.

### CA-11

La confirmación del turno se envía al teléfono proporcionado mediante WhatsApp después de crear la reserva.

### CA-12

Un fallo de WhatsApp o del sistema de automatización no cancela ni impide confirmar el turno.

### CA-13

La API pública no expone datos privados de otros clientes.

### CA-14

Una reserva confirmada genera el evento `appointment.created`.

## 16. Casos límite

Deben contemplarse como mínimo:

- nombre y apellido vacío
- teléfono vacío
- teléfono inválido
- información adicional vacía
- información adicional de exactamente 300 caracteres
- información adicional de más de 300 caracteres
- datos excesivamente largos en campos obligatorios
- horario que deja de estar disponible durante la reserva
- doble envío del formulario
- reintento HTTP
- error después de persistir pero antes de responder
- tenant despublicado durante el proceso
- servicio desactivado durante el proceso
- profesional desactivado durante el proceso
- fallo de WhatsApp después de crear el turno

## 17. Pruebas

La implementación deberá incluir pruebas para:

- creación pública válida
- validación de nombre y apellido
- validación de teléfono
- validación del campo opcional
- límite de 300 caracteres
- cliente sin cuenta
- conflicto de disponibilidad
- concurrencia
- idempotencia
- respuesta de confirmación
- privacidad de datos
- generación de `appointment.created`
- envío de confirmación mediante el flujo de WhatsApp
- independencia respecto de WhatsApp/n8n para confirmar la reserva
- comportamiento ante errores de notificación

## 18. Dependencias

Esta especificación depende de:

- `ARCHITECTURE.md`
- `API-CONTRACT.md`
- `DECISIONS.md`
- `specs/002-disponibilidad.md`
- `specs/003-creacion-turno.md`
- `specs/007-publicacion-negocio.md`
- `specs/008-acceso-publico-widget.md`

## 19. Fuera de alcance

No se define aquí:

- cuentas de clientes
- autenticación de clientes
- cancelación pública
- reprogramación pública
- pagos
- promociones
- historial de clientes
- CRM
- campañas de marketing
- proveedor concreto de WhatsApp
- implementación concreta de n8n
- política legal de privacidad o retención de datos
