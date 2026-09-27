# Especificación 011: Configuración de profesionales

## 1. Objetivo

Definir la configuración administrativa de los profesionales que atienden turnos dentro de un tenant.

Un profesional pertenece exclusivamente a un tenant y puede encontrarse activo o inactivo.

La configuración de profesionales es necesaria para que el negocio pueda cumplir el requisito de publicación definido en `specs/007-publicacion-negocio.md`.

## 2. Contexto

El sistema es multi-tenant.

Cada profesional pertenece a un único tenant. Las operaciones administrativas deben ejecutarse siempre dentro del tenant asociado al administrador autenticado.

El backend es la fuente de verdad y el cliente no puede seleccionar arbitrariamente el tenant mediante un `tenant_id` enviado en una solicitud.

## 3. Actor

### Administrador

Puede:

- consultar los profesionales de su tenant;
- crear un profesional;
- activar un profesional;
- desactivar un profesional.

Todas estas operaciones requieren autenticación y autorización administrativa.

No existe gestión de cuentas de acceso para profesionales en esta especificación.

## 4. Modelo de datos

El recurso profesional utiliza inicialmente la entidad `Employee`.

Campos:

- `id`: identificador UUID.
- `tenantId`: tenant propietario del profesional.
- `name`: nombre del profesional.
- `status`: `active` o `inactive`.
- `createdAt`: fecha de creación.
- `updatedAt`: fecha de última actualización.

El `tenantId` pertenece al contexto interno del backend y no debe ser utilizado desde el request para seleccionar el tenant autorizado.

## 5. Reglas de negocio

### RN-01 — Aislamiento por tenant

Un administrador solo puede consultar y modificar profesionales pertenecientes a su tenant.

### RN-02 — Nombre obligatorio

El nombre debe:

- ser un string;
- eliminar espacios externos;
- contener al menos un carácter después del trim;
- tener una longitud máxima de 120 caracteres.

### RN-03 — Estado

Un profesional solo puede tener uno de estos estados:

- `active`
- `inactive`

Un profesional nuevo se crea inicialmente como `active`.

### RN-04 — Profesional activo

Un profesional con estado `active` puede ser considerado por las funcionalidades que requieran profesionales activos.

En particular, la publicación del negocio debe considerar la existencia de al menos un profesional activo.

### RN-05 — Profesional inactivo

Un profesional `inactive` no debe considerarse como profesional disponible para nuevas reservas.

La lógica específica de disponibilidad y reservas se definirá en las especificaciones correspondientes.

### RN-06 — No eliminar como parte del MVP

Esta especificación no define eliminación física de profesionales.

Para conservar referencias históricas y evitar ambigüedades con turnos existentes, el ciclo de vida se gestiona mediante `active/inactive`.

## 6. API administrativa

Las rutas se encuentran bajo:

`/api/v1/admin/configuration`

### GET /professionals

Devuelve los profesionales del tenant autenticado.

Respuesta exitosa:

`200 OK`

Ejemplo:

```json
[
  {
    "id": "uuid",
    "name": "Juan",
    "status": "active"
  }
]
```

El resultado debe pertenecer exclusivamente al tenant del administrador autenticado.

### POST /professionals

Crea un profesional para el tenant autenticado.

Request:

```json
{
  "name": "Juan"
}
```

El cliente no debe utilizar `tenantId` para seleccionar el tenant.

Respuesta exitosa:

`201 Created`

Ejemplo:

```json
{
  "id": "uuid",
  "name": "Juan",
  "status": "active"
}
```

Datos inválidos:

`400 Bad Request`

### PATCH /professionals/:id/status

Modifica el estado del profesional.

Request:

```json
{
  "status": "inactive"
}
```

Valores permitidos:

- `active`
- `inactive`

Respuesta exitosa:

`200 OK`

El profesional debe pertenecer al tenant del administrador autenticado.

Si el profesional no existe dentro del tenant autenticado:

`404 Not Found`

Si el estado enviado no es válido:

`400 Bad Request`

## 7. Seguridad

Todas las rutas administrativas requieren autenticación.

El tenant se obtiene del contexto autenticado:

```text
JWT
 ↓
usuario autenticado
 ↓
tenantId
 ↓
profesionales del tenant
```

El backend no debe confiar en un `tenantId` enviado por el cliente.

Un administrador autenticado no debe poder leer ni modificar profesionales de otro tenant.

## 8. Errores

Los errores deben respetar la estructura definida en `API-CONTRACT.md`.

Casos mínimos:

- `401 Unauthorized`: no autenticado.
- `400 Bad Request`: datos inválidos.
- `404 Not Found`: profesional inexistente dentro del tenant.
- `500 Internal Server Error`: error inesperado.

## 9. Criterios de aceptación

### CA-01

Un administrador autenticado puede consultar los profesionales de su tenant.

### CA-02

Un administrador autenticado puede crear un profesional.

### CA-03

Un profesional nuevo queda inicialmente en estado `active`.

### CA-04

Un administrador puede activar o desactivar un profesional.

### CA-05

Un administrador no puede consultar profesionales de otro tenant.

### CA-06

Un administrador no puede modificar profesionales de otro tenant.

### CA-07

Enviar un `tenantId` arbitrario en una solicitud no permite cambiar el tenant utilizado por el backend.

### CA-08

No se puede crear un profesional con un nombre vacío.

### CA-09

No se aceptan estados distintos de `active` e `inactive`.

### CA-10

La existencia de al menos un profesional activo puede ser utilizada por la validación de publicación del tenant.

## 10. Casos límite

Deben contemplarse:

- creación sin nombre;
- nombre compuesto únicamente por espacios;
- nombre de más de 120 caracteres;
- estado inválido;
- profesional inexistente;
- profesional perteneciente a otro tenant;
- intento de acceder sin autenticación;
- tenant sin profesionales;
- tenant con profesionales pero todos inactivos;
- desactivar el único profesional activo.

Desactivar el último profesional activo no debe eliminarlo. El efecto sobre la publicación del negocio se determina por las reglas de publicación.

## 11. Pruebas requeridas

### Servicio

- creación de profesional;
- normalización del nombre;
- validación de nombre;
- listado por tenant;
- cambio de estado.

### Controller/API

- listado autenticado;
- creación autenticada;
- rechazo sin autenticación;
- aislamiento entre tenants;
- ignorar cualquier `tenantId` enviado por el cliente;
- activación;
- desactivación;
- rechazo de estado inválido;
- profesional inexistente.

## 12. Dependencias

Esta especificación depende de:

- `SDD.md`
- `AGENTS.md`
- `ARCHITECTURE.md`
- `API-CONTRACT.md`
- `DECISIONS.md`
- `specs/001-configuracion-tenant.md`
- `specs/006-autenticacion-autorizacion.md`
- `specs/007-publicacion-negocio.md`

## 13. Fuera de alcance

Esta especificación no define:

- cuentas de usuario para profesionales;
- permisos propios de cada profesional;
- horarios individuales por profesional;
- vacaciones individuales;
- límites diarios por profesional;
- asignación de servicios específicos a profesionales;
- eliminación física;
- notificaciones;
- disponibilidad pública;
- creación o modificación de turnos.

Estas reglas deberán definirse en las especificaciones correspondientes si pasan a formar parte del producto.
