# Especificación 009: Datos del cliente y confirmación del turno

## 1. Objetivo

Definir los datos mínimos que debe proporcionar un cliente para crear un turno y cómo se confirma la reserva.

El cliente no requiere una cuenta autenticada en el MVP.

## 2. Contexto

El cliente accede mediante la página pública o el widget.

Después de seleccionar servicio, fecha, horario y profesional cuando corresponda, debe proporcionar los datos necesarios para identificar el turno y permitir su gestión o comunicación.

El backend debe validar los datos y confirmar el turno únicamente después de verificar nuevamente la disponibilidad.

## 3. Datos del cliente

El MVP debe solicitar únicamente los datos necesarios para:

- identificar al cliente dentro del turno
- permitir la comunicación relacionada con el turno
- facilitar futuras operaciones sobre el turno si se define un mecanismo de gestión

Datos mínimos propuestos:

- nombre
- teléfono o medio de contacto principal
- correo electrónico, si el negocio lo requiere

La obligatoriedad exacta de teléfono y correo deberá definirse mediante configuración o decisión posterior. No se debe exigir información innecesaria.

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
Ingresar datos del cliente
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
~~~

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

## 7. Estado de la reserva

El turno creado debe tener un estado que permita distinguir una reserva confirmada de otros estados futuros.

Para el MVP, la creación exitosa debe producir un estado equivalente a:

~~~text
confirmed
~~~

Los estados adicionales deberán definirse según las necesidades del dominio.

## 8. Respuesta ante conflicto

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

## 9. Idempotencia

La creación de un turno debe contemplar solicitudes repetidas.

Esto es especialmente importante si:

- el cliente pulsa varias veces el botón de confirmar
- existe una repetición de la solicitud HTTP
- el navegador reintenta una solicitud
- existe una interrupción durante la respuesta

La implementación deberá utilizar un mecanismo de idempotencia apropiado.

Una misma operación de reserva no debe crear múltiples turnos por una repetición accidental de la solicitud.

## 10. Privacidad

Los datos del cliente deben tratarse como información privada.

La API pública no debe permitir consultar libremente los datos de clientes de otros turnos.

Una respuesta pública de disponibilidad nunca debe revelar:

- nombres de clientes
- teléfonos
- correos
- motivos o notas privadas
- información de otros turnos que permita identificar personas

## 11. Confirmación por canales externos

La creación del turno debe ser independiente de cualquier sistema externo de notificaciones.

Por lo tanto:

~~~text
Crear turno
    |
    +--> Persistir turno
    |
    +--> Evento appointment.created
               |
               v
          Automatización
               |
               +--> Email
               +--> WhatsApp
               +--> otros canales
~~~

Si el sistema de notificaciones está temporalmente caído, el turno debe seguir existiendo y permanecer confirmado.

n8n no debe ser necesario para confirmar la reserva.

## 12. Evento de creación

Una reserva confirmada debe poder generar el evento:

~~~text
appointment.created
~~~

El mecanismo concreto de publicación y procesamiento del evento se definirá durante la implementación de eventos/automatizaciones.

El evento no debe ser utilizado como condición para decidir si el turno fue creado.

## 13. Gestión posterior del turno

El cliente no tendrá cuenta en el MVP.

Por lo tanto, esta especificación no define todavía cómo podrá:

- cancelar
- reprogramar
- consultar nuevamente
- modificar sus datos

Antes de exponer esas operaciones públicamente deberá definirse un mecanismo seguro de acceso al turno, como un enlace privado o token.

## 14. Datos sensibles y notas

No se deben agregar campos libres o información sensible por defecto.

Si en el futuro un negocio necesita notas adicionales, deberán definirse:

- finalidad
- visibilidad
- almacenamiento
- acceso
- retención

mediante una especificación específica.

## 15. Criterios de aceptación

### CA-01

Un cliente puede solicitar un turno sin crear una cuenta.

### CA-02

El backend valida los datos requeridos antes de crear el turno.

### CA-03

El backend vuelve a validar disponibilidad antes de confirmar.

### CA-04

Un conflicto de disponibilidad devuelve HTTP 409 y no crea un turno.

### CA-05

Una solicitud repetida no crea accidentalmente múltiples turnos cuando corresponde a la misma operación.

### CA-06

Una reserva solo se considera confirmada después de persistirse correctamente.

### CA-07

La respuesta de confirmación contiene la información necesaria para reconocer el turno.

### CA-08

La API pública no expone datos privados de otros clientes.

### CA-09

La caída de n8n o de un proveedor de notificaciones no impide crear y confirmar el turno.

### CA-10

Una reserva confirmada puede generar el evento `appointment.created` para procesos posteriores.

## 16. Casos límite

Deben contemplarse como mínimo:

- nombre vacío o inválido
- teléfono inválido cuando sea obligatorio
- correo inválido cuando sea obligatorio
- datos excesivamente largos
- horario que deja de estar disponible durante la reserva
- doble envío del formulario
- reintento HTTP
- error después de persistir pero antes de responder
- tenant despublicado durante el proceso
- servicio desactivado durante el proceso
- profesional desactivado durante el proceso
- fallo del sistema de notificaciones después de crear el turno

## 17. Pruebas

La implementación deberá incluir pruebas para:

- creación pública válida
- validación de datos
- cliente sin cuenta
- conflicto de disponibilidad
- concurrencia
- idempotencia
- respuesta de confirmación
- privacidad de datos
- generación de `appointment.created`
- independencia respecto de n8n
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
- implementación concreta de WhatsApp o email
- implementación concreta de n8n
- política legal de privacidad o retención de datos
