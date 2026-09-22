# Especificación 006: Autenticación y autorización del administrador

## 1. Objetivo

Definir el mecanismo conceptual de autenticación y autorización para el administrador o dueño de un negocio dentro de la plataforma de turnos.

En el MVP, únicamente el administrador requiere autenticación y autorización.

Los clientes podrán utilizar el sistema público de turnos sin crear una cuenta.

---

## 2. Contexto

La plataforma es multi-tenant. Cada negocio constituye un tenant independiente.

El administrador es el propietario o responsable del negocio y debe poder acceder a las funciones privadas de administración de su tenant.

El cliente utiliza la página pública de turnos o el widget integrado en un sitio existente para consultar disponibilidad y solicitar turnos.

La identidad y los permisos del administrador no deben depender de datos enviados arbitrariamente por el frontend.

---

## 3. Actores

### 3.1 Administrador

Usuario autenticado asociado a un tenant.

Puede gestionar los recursos de su negocio según los permisos definidos por el sistema.

### 3.2 Cliente

Usuario público.

No requiere cuenta ni autenticación para el flujo público de reserva del MVP.

Las operaciones posteriores sobre un turno del cliente, como cancelación o reprogramación, requieren definir un mecanismo seguro de identificación del turno. Este mecanismo queda fuera de esta especificación y no implica crear una cuenta de cliente.

---

## 4. Alcance

Esta especificación cubre:

- identidad del administrador
- autenticación del administrador
- autorización del administrador
- asociación entre administrador y tenant
- protección de recursos administrativos
- aislamiento entre tenants
- respuestas de autenticación y autorización
- requisitos básicos de seguridad
- pruebas

Esta especificación no define todavía:

- proveedor concreto de autenticación
- JWT frente a sesiones
- recuperación de contraseña
- autenticación social
- MFA
- cuentas de clientes
- mecanismo de acceso del cliente a sus turnos
- facturación o roles avanzados de SaaS

Las decisiones concretas de implementación deberán registrarse en `DECISIONS.md` cuando corresponda.

---

## 5. Requisitos funcionales

### RF-01: Identidad del administrador

Cada administrador debe poseer una identidad única dentro del sistema.

La identidad debe permitir determinar de forma confiable:

- quién realiza la operación
- a qué tenant pertenece
- qué permisos posee

### RF-02: Autenticación

Los endpoints administrativos deben requerir que el administrador esté autenticado.

Una solicitud sin una identidad autenticada válida debe ser rechazada.

### RF-03: Autorización

La autenticación no implica autorización automática para cualquier operación.

El backend debe comprobar que el administrador posee permisos suficientes para realizar la operación solicitada.

### RF-04: Asociación con tenant

La relación entre administrador y tenant debe mantenerse en el backend.

El cliente no debe poder cambiar su tenant simplemente modificando un `tenant_id` enviado en una solicitud.

### RF-05: Aislamiento entre tenants

Un administrador únicamente puede acceder y modificar recursos pertenecientes a los tenants que tiene autorizados.

Un identificador de recurso perteneciente a otro tenant no debe permitir acceso, modificación, cancelación ni eliminación.

### RF-06: Protección de endpoints

Los recursos administrativos deberán diferenciarse de los recursos públicos.

Ejemplos de operaciones administrativas:

- configurar el negocio
- crear o modificar servicios
- gestionar profesionales
- configurar horarios
- gestionar bloqueos
- consultar y gestionar turnos

### RF-07: Cliente público

Las operaciones públicas necesarias para consultar disponibilidad y solicitar un turno no deben exigir una cuenta de cliente.

El backend continuará validando todas las reglas de negocio aunque la solicitud sea pública.

---

## 6. Autenticación vs autorización

El sistema debe mantener separadas ambas responsabilidades.

### Autenticación

Responde:

> ¿Quién es este usuario?

Resultado esperado:

- identidad del administrador
- tenant asociado
- información necesaria para evaluar permisos

### Autorización

Responde:

> ¿Este administrador puede realizar esta operación sobre este recurso?

Debe considerar como mínimo:

- identidad
- permisos
- tenant
- recurso solicitado
- operación solicitada

---

## 7. Flujo administrativo

Conceptualmente:

```text
Solicitud HTTP
      |
      v
Autenticación
      |
      +-- no válida --> 401
      |
      v
Identidad del administrador
      |
      v
Contexto del tenant
      |
      v
Autorización
      |
      +-- no permitida --> 403
      |
      v
Reglas de negocio
      |
      v
Persistencia
```

La implementación concreta del middleware o mecanismo de autenticación queda pendiente.

---

## 8. Respuestas HTTP

### 8.1 No autenticado

Cuando un endpoint protegido recibe una solicitud sin credenciales válidas:

**HTTP 401 Unauthorized**

Ejemplo:

```json
{
  "error": {
    "code": "AUTHENTICATION_REQUIRED",
    "message": "Se requiere autenticación."
  }
}
```

### 8.2 Autenticado pero sin permisos

Cuando el administrador está autenticado pero no tiene autorización para realizar la operación:

**HTTP 403 Forbidden**

Ejemplo:

```json
{
  "error": {
    "code": "FORBIDDEN",
    "message": "No tiene permisos para realizar esta operación."
  }
}
```

El sistema no debe utilizar 403 para una solicitud que simplemente carece de autenticación.

---

## 9. Autorización y acceso a recursos

La autorización debe aplicarse más allá de las rutas HTTP.

El acceso a datos debe respetar el tenant y los permisos también en las capas internas correspondientes.

No debe considerarse suficiente una comprobación únicamente en el frontend.

Ejemplo incorrecto:

```text
Frontend envía tenant_id
       |
Backend confía en tenant_id
       |
Consulta datos
```

Ejemplo esperado:

```text
Credencial autenticada
       |
Identidad del administrador
       |
Tenant autorizado obtenido del backend
       |
Consulta filtrada por tenant
```

---

## 10. Seguridad básica

La implementación deberá contemplar como mínimo:

- almacenamiento seguro de credenciales si se utiliza contraseña
- nunca almacenar contraseñas en texto plano
- protección de credenciales y secretos
- expiración o invalidación adecuada de sesiones/tokens según el mecanismo elegido
- protección contra intentos abusivos de autenticación
- rate limiting en endpoints sensibles cuando corresponda
- validación de entradas
- HTTPS en entornos donde se transmitan credenciales
- no exponer información sensible en mensajes de error
- registro de eventos de seguridad relevantes

Los valores concretos de configuración se definirán durante la implementación.

---

## 11. Cliente y turnos

El cliente no tendrá una cuenta en el MVP.

Por lo tanto, las operaciones futuras sobre un turno que actualmente requieren identificar al cliente deberán utilizar otro mecanismo de seguridad.

Ejemplos posibles, pendientes de decisión:

- enlace privado con token
- token de gestión del turno
- código de verificación
- verificación mediante un dato de contacto

No se debe implementar ninguno de estos mecanismos como decisión implícita dentro de esta especificación.

Antes de implementar cancelación o reprogramación pública de turnos deberá definirse y documentarse el mecanismo elegido.

---

## 12. Integración con el resto del sistema

La autenticación y autorización deberán integrarse con:

- configuración del tenant
- servicios
- profesionales
- horarios
- bloqueos
- gestión administrativa de turnos

El Booking Engine debe seguir siendo responsable de las reglas de disponibilidad y reservas.

La autenticación no reemplaza las validaciones de negocio.

---

## 13. API conceptual

Los endpoints concretos de autenticación quedan pendientes de la decisión de implementación.

Conceptualmente existirán operaciones equivalentes a:

```text
POST /api/v1/auth/...
```

y endpoints administrativos protegidos.

Los contratos definitivos deberán agregarse a `API-CONTRACT.md` o a la especificación correspondiente cuando se defina el mecanismo concreto.

---

## 14. Criterios de aceptación

### CA-01
Un administrador no autenticado no puede acceder a recursos administrativos.

### CA-02
Un administrador autenticado puede acceder únicamente a los recursos para los que está autorizado.

### CA-03
Un administrador no puede acceder a recursos pertenecientes a otro tenant.

### CA-04
Modificar un `tenant_id` enviado por el cliente no permite cambiar el contexto autorizado.

### CA-05
Una solicitud sin autenticación recibe HTTP 401 cuando intenta acceder a un recurso protegido.

### CA-06
Una solicitud autenticada sin permisos suficientes recibe HTTP 403.

### CA-07
Los endpoints públicos necesarios para reservar un turno no requieren una cuenta de cliente.

### CA-08
Las operaciones administrativas continúan validando las reglas de negocio correspondientes.

### CA-09
No se almacenan contraseñas en texto plano si el sistema utiliza autenticación mediante contraseña.

### CA-10
La solución puede evolucionar posteriormente para incorporar otros roles o mecanismos de autenticación sin rediseñar el Booking Engine.

---

## 15. Casos límite

Deben contemplarse como mínimo:

- credenciales inválidas
- credenciales expiradas o sesión/token inválido
- administrador deshabilitado
- tenant deshabilitado
- administrador intentando acceder a otro tenant
- recurso inexistente
- recurso existente pero perteneciente a otro tenant
- permisos insuficientes
- múltiples solicitudes simultáneas con la misma identidad
- intento de manipular el contexto del tenant
- abuso de endpoints de autenticación

---

## 16. Pruebas

La implementación deberá incluir pruebas para:

- autenticación válida
- autenticación inválida
- acceso sin autenticación
- autorización válida
- autorización insuficiente
- aislamiento entre tenants
- manipulación de `tenant_id`
- administrador deshabilitado
- tenant deshabilitado
- acceso a recursos inexistentes
- acceso a recursos de otro tenant
- endpoints públicos sin autenticación
- protección de credenciales

Las pruebas de seguridad deben ejecutarse también sobre las capas internas que realizan acceso a recursos, no únicamente sobre las rutas HTTP.

---

## 17. Dependencias

Esta especificación depende conceptualmente de:

- `ARCHITECTURE.md`
- `API-CONTRACT.md`
- `DECISIONS.md`
- `specs/001-configuracion-tenant.md`

Las especificaciones de disponibilidad y gestión de turnos deberán reutilizar el contexto de seguridad definido aquí cuando incorporen operaciones administrativas.

---

## 18. Fuera de alcance del MVP

Quedan fuera de esta especificación:

- cuentas de clientes
- login de clientes
- roles complejos
- equipos con múltiples administradores
- permisos configurables por usuario
- autenticación social
- MFA
- SSO
- facturación
- administración de suscripciones

Estas funcionalidades pueden incorporarse posteriormente mediante nuevas especificaciones y decisiones arquitectónicas.
