# Especificación 015: Integración del widget mediante iframe

## 1. Objetivo

Definir el contrato mínimo para que un negocio con un sitio web existente pueda incorporar el sistema de turnos mediante un widget aislado.

La primera implementación utilizará un iframe servido por la plataforma.

El widget debe consumir la API pública existente y utilizar el mismo Booking Engine que la página pública.

## 2. Alcance

Esta especificación define:

- identificación pública de la integración;
- URL conceptual del widget;
- comunicación entre el widget y la API;
- origen autorizado;
- requisitos mínimos de seguridad;
- comportamiento cuando la integración no está disponible.

No define:

- diseño visual definitivo;
- personalización avanzada;
- modificación automática del sitio externo;
- plugins específicos de WordPress, Wix u otras plataformas;
- análisis mediante IA del sitio;
- pagos;
- cuentas de clientes.

## 3. Identificación

Cada integración de sitio tendrá un `publicKey`.

El `publicKey`:

- identifica públicamente una integración;
- permite resolver el tenant correspondiente;
- no es una credencial administrativa;
- no debe permitir acceso a recursos administrativos;
- no debe contener información sensible.

El cliente no enviará `tenantId`.

## 4. URL del widget

La plataforma expondrá una URL pública conceptual:

```text
https://<plataforma>/widget/<publicKey>
```

Ejemplo conceptual:

```text
https://turnos.example.com/widget/pk_live_xxx
```

El frontend del widget será responsable de renderizar la experiencia de reserva.

## 5. Integración mediante iframe

El administrador podrá incorporar el widget mediante un iframe:

```html
<iframe
  src="https://turnos.example.com/widget/pk_live_xxx"
  title="Reservar turno"
></iframe>
```

El sitio externo no necesita acceder al código interno de la aplicación de turnos.

El iframe mantiene aislada la interfaz del widget respecto del CSS y JavaScript del sitio anfitrión.

## 6. Flujo

```text
Sitio del negocio
      |
      v
    iframe
      |
      v
Widget de la plataforma
      |
      +--> GET /api/v1/public/sites/:publicKey
      |
      +--> GET /services
      |
      +--> GET /availability
      |
      +--> POST /appointments
      |
      v
Booking Engine
      |
      v
PostgreSQL
```

La interfaz pública y el widget utilizan el mismo flujo de reserva.

## 7. CORS

Debe distinguirse entre el origen del sitio anfitrión y el origen del widget.

En la implementación mediante iframe, el código JavaScript que realiza las llamadas a la API se ejecuta dentro del documento del widget. Por lo tanto, el `Origin` de esas llamadas corresponde al origen donde está alojado el widget, no al dominio que contiene el iframe.

El backend debe:

1. identificar la integración mediante `publicKey`;
2. comprobar que la integración está verificada;
3. comprobar que está conectada;
4. permitir el origen configurado para la aplicación del widget;
5. mantener, cuando corresponda, los orígenes del dominio del negocio para futuras integraciones JavaScript directas;
6. devolver `Access-Control-Allow-Origin` únicamente para el origen solicitado cuando esté autorizado.

La ausencia de un header `Origin` puede corresponder a clientes que no utilizan CORS de navegador y no constituye por sí misma autenticación.

CORS no debe considerarse un mecanismo de autenticación.

## 8. Orígenes autorizados

Al crear una integración de dominio, el backend registrará inicialmente los orígenes HTTPS correspondientes al dominio normalizado:

```text
https://dominio.com
https://www.dominio.com
```

Estos orígenes se asocian exclusivamente a la integración creada.

No deben aceptarse orígenes arbitrarios enviados por el cliente público.

## 9. Estado de la integración

El widget solamente podrá operar para una integración que cumpla:

```text
verificationStatus = VERIFIED
integrationStatus = CONNECTED
tenant.status = published
```

Si alguna condición deja de cumplirse, la API pública debe rechazar el acceso correspondiente.

Desconectar una integración no elimina automáticamente sus datos ni reservas.

## 10. API pública

El widget utilizará inicialmente:

```http
GET /api/v1/public/sites/:publicKey
GET /api/v1/public/sites/:publicKey/services
GET /api/v1/public/sites/:publicKey/availability
POST /api/v1/public/sites/:publicKey/appointments
```

El widget no debe llamar directamente a endpoints administrativos.

## 11. Datos públicos

El widget puede recibir únicamente la información necesaria para reservar.

No debe recibir:

- credenciales;
- JWT administrativos;
- verificationToken;
- configuración interna;
- datos privados de otros clientes;
- información administrativa innecesaria.

## 12. Creación de reservas

El widget enviará los datos públicos necesarios para crear el turno.

El backend determinará el tenant a partir de `publicKey`.

El backend volverá a validar:

- servicio;
- profesional;
- duración;
- horario;
- bloqueos;
- disponibilidad;
- reglas del negocio;
- concurrencia.

El resultado de una consulta previa de disponibilidad nunca garantiza que la reserva pueda crearse.

## 13. Errores

El widget debe interpretar los códigos de error de la API y no depender del texto de `message`.

Como mínimo:

- `400 INVALID_REQUEST`
- `403 CORS_ORIGIN_NOT_ALLOWED`
- `404 PUBLIC_SITE_NOT_FOUND`
- `409 APPOINTMENT_CONFLICT`
- `409 APPOINTMENT_UNAVAILABLE`

Ante un conflicto de reserva, el widget debe permitir volver a consultar disponibilidad.

## 14. Responsive

El iframe debe poder utilizarse dentro de diferentes tamaños de contenedor.

La primera versión no requiere un mecanismo complejo de auto-resize.

El diseño inicial debe funcionar correctamente dentro de un iframe con altura configurable.

Un mecanismo posterior mediante `postMessage` podrá evaluarse si se necesita adaptar automáticamente la altura.

## 15. Seguridad

El widget se considera completamente público.

No debe asumir que:

- el iframe oculta la API;
- el `publicKey` es secreto;
- CORS impide llamadas directas;
- el frontend puede aplicar las reglas de negocio.

Todas las restricciones críticas deben permanecer en el backend.

## 16. Despublicación

Si el tenant deja de estar publicado:

- el widget debe dejar de permitir nuevas reservas;
- la API pública debe rechazar nuevas reservas;
- no deben eliminarse automáticamente las reservas existentes.

## 17. Aislamiento multi-tenant

Modificar manualmente el `publicKey` no debe permitir:

- obtener información administrativa;
- acceder a datos privados;
- utilizar un tenant distinto mediante parámetros adicionales;
- crear una reserva para otro tenant.

El `publicKey` es la única referencia pública utilizada para resolver el contexto de la integración.

## 18. Criterios de aceptación

### CA-01

Una integración verificada y conectada puede cargar el widget.

### CA-02

El widget puede obtener la información pública del negocio.

### CA-03

El widget puede listar servicios activos.

### CA-04

El widget puede consultar disponibilidad.

### CA-05

El widget puede crear una reserva.

### CA-06

El widget utiliza el mismo Booking Engine que la API pública.

### CA-07

Un origen no autorizado no puede utilizar la API desde un navegador mediante CORS.

### CA-08

El widget no contiene credenciales administrativas.

### CA-09

Modificar el `publicKey` no permite acceder a recursos administrativos.

### CA-10

Un tenant despublicado no acepta nuevas reservas.

## 19. Evolución futura

Podrán agregarse posteriormente:

- Web Component;
- SDK JavaScript;
- personalización visual;
- auto-resize mediante `postMessage`;
- plugins para plataformas específicas;
- instalación asistida;
- integración profunda con sitios externos.

Estas funcionalidades requieren especificaciones adicionales y no deben introducirse en el MVP sin necesidad.
