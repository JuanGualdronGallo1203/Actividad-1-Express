# DriveSwift API - Diseño Reflexivo de Endpoints

**Nombre de la aplicación:** DriveSwift

**Autor:** Gemini, Asistente de IA

---

## Preguntas Guía

A continuación, se responden brevemente las preguntas guía para una reflexión previa al diseño:

| Pregunta | Respuesta Breve |
| :--- | :--- |
| **¿Qué entiendes por “endpoint” en el contexto de una API?** | Es una **URL** específica (una ruta) a la que las aplicaciones cliente pueden enviar solicitudes para acceder a o modificar recursos en el servidor, utilizando un método HTTP. |
| **¿Cuál es la diferencia entre un endpoint público y uno privado?** | Un **público** no requiere autenticación (ej: registro de usuario). Uno **privado** sí requiere autenticación y/o autorización (ej: solicitar un viaje, ver el historial de pagos). |
| **¿Qué información de un usuario consideras confidencial y no debería exponerse?** | **Contraseñas** (hasheadas), **tokens**, **dirección de correo electrónico completa** (a otros usuarios), **información de pago sensible**, **ubicación en tiempo real** (a menos que sea para el viaje activo). |
| **¿Por qué es importante definir bien los métodos HTTP (GET, POST, PUT/PATCH, DELETE) en cada endpoint?** | Para garantizar la **semántica** y la **idempotencia** (donde aplica) de la API, haciendo que su uso sea predecible y que el cliente sepa qué esperar de la operación (ej: `POST` para crear, `GET` para leer). |
| **¿Qué tipo de información requiere autenticación en este sistema?** | Toda aquella información que sea **específica de un usuario** o que implique una **modificación de recursos** (ej: solicitar un viaje, actualizar perfil, ver pagos, aceptar un viaje). |
| **¿Cómo manejarías la seguridad de la ubicación de conductores y pasajeros?** | La ubicación se transmitiría mediante **conexiones seguras (HTTPS/WSS)**, solo se compartiría entre los **participantes directos del viaje** y solo durante la **duración activa del viaje**. |
| **¿Qué pasaría si un viaje es solicitado y no hay conductores disponibles? ¿Cómo debería responder la API?** | La API debería responder con un **código de error 404/408/409 específico** si se determina inmediatamente que no hay disponibilidad, junto con un mensaje claro (ej: "No hay conductores disponibles cerca"). |
| **¿Cómo identificarías los recursos principales de esta aplicación?** | Mediante los **sustantivos clave** del dominio: `users`, `rides`, `payments`, `ratings`, `vehicles`. |
| **¿Qué ventajas tendría versionar la API (por ejemplo, `/v1/...) desde el inicio?** | Permite realizar **cambios no retrocompatibles** en el futuro sin romper las aplicaciones cliente existentes, facilitando una **transición controlada**. |
| **¿Por qué es importante documentar las respuestas de error y no solo las exitosas?** | Permite a los desarrolladores clientes **manejar fallos de manera predecible y robusta**, mejorando la **experiencia de usuario** al poder mostrar mensajes de error útiles. |

---

## Roles y Permisos

| Rol | Acciones Principales |
| :--- | :--- |
| **Pasajero** | Registrarse, iniciar sesión, actualizar perfil, **solicitar un viaje**, ver estado del viaje, cancelar viaje, ver historial de viajes, ver recibos de pago, **calificar conductor**. |
| **Conductor** | Registrarse, iniciar sesión, actualizar perfil/vehículo, **activar/desactivar disponibilidad**, **aceptar/rechazar una solicitud**, actualizar estado del viaje (iniciar, finalizar), ver ganancias, **calificar pasajero**. |
| **Administrador** | Iniciar sesión, **gestionar usuarios** (bloquear/desbloquear), **gestionar vehículos**, **ver métricas y reportes**, administrar tarifas, **revisar y moderar calificaciones/disputas**. |

---

## Recursos Principales

Los recursos principales que manejará la API, siguiendo las convenciones RESTful, son: `users`, `rides`, `payments`, `ratings`, `vehicles`, y `admin`.

---

## Tabla de Endpoints (Mínimo 20)

| Método HTTP | Ruta del Endpoint | Descripción Breve | Parámetros Esperados | Autenticación Requerida |
| :--- | :--- | :--- | :--- | :--- |
| **Público** | | | | |
| `POST` | `/v1/users/register` | Crea un nuevo usuario (pasajero o conductor). | **Body:** `name`, `email`, `password`, `role` | Público |
| `POST` | `/v1/users/login` | Inicia sesión y devuelve un token de autenticación. | **Body:** `email`, `password` | Público |
| **Usuarios (Auth: Pasajero/Conductor)** | | | | |
| `GET` | `/v1/users/me` | Obtiene la información del perfil del usuario autenticado. | - | Requiere Token |
| `PUT` | `/v1/users/me` | Actualiza la información del perfil del usuario. | **Body:** `name`, `phone`, `avatar_url`, etc. | Requiere Token |
| `POST` | `/v1/users/me/vehicles` | Registra un nuevo vehículo para un conductor. | **Body:** `model`, `plate`, `color`, `year` | Requiere Token (Conductor) |
| **Viajes (Auth: Pasajero)** | | | | |
| `POST` | `/v1/rides` | Solicita un nuevo viaje. | **Body:** `origin`, `destination`, `payment_method`, `requested_time` | Requiere Token (Pasajero) |
| `GET` | `/v1/rides/{rideId}` | Obtiene el estado y detalles de un viaje específico. | **Path:** `rideId` | Requiere Token |
| `PATCH` | `/v1/rides/{rideId}/cancel` | Cancela un viaje pendiente o en curso (con lógica de penalización). | **Path:** `rideId` | Requiere Token (Pasajero/Conductor) |
| `GET` | `/v1/rides/history` | Obtiene la lista de viajes completados y cancelados del usuario. | **Query:** `limit`, `offset`, `status` | Requiere Token |
| **Viajes (Auth: Conductor)** | | | | |
| `GET` | `/v1/rides/available` | Obtiene una lista de solicitudes de viaje disponibles cercanas. | **Query:** `latitude`, `longitude`, `radius` | Requiere Token (Conductor) |
| `POST` | `/v1/rides/{rideId}/accept` | El conductor acepta una solicitud de viaje. | **Path:** `rideId` | Requiere Token (Conductor) |
| `POST` | `/v1/rides/{rideId}/start` | El conductor marca el viaje como iniciado. | **Path:** `rideId` | Requiere Token (Conductor) |
| `POST` | `/v1/rides/{rideId}/complete`| El conductor marca el viaje como finalizado y dispara el cobro. | **Path:** `rideId` | Requiere Token (Conductor) |
| `PUT` | `/v1/drivers/status` | Actualiza el estado de disponibilidad del conductor y su ubicación. | **Body:** `is_available`, `latitude`, `longitude` | Requiere Token (Conductor) |
| **Pagos y Calificaciones** | | | | |
| `GET` | `/v1/payments/{paymentId}/receipt` | Obtiene el recibo detallado de un pago. | **Path:** `paymentId` | Requiere Token |
| `POST` | `/v1/ratings` | Envía una calificación (de pasajero a conductor o viceversa). | **Body:** `ride_id`, `target_user_id`, `score`, `comment` | Requiere Token |
| `GET` | `/v1/ratings/me` | Obtiene el promedio de calificación y el historial de reseñas recibidas. | - | Requiere Token |
| **Administración (Auth: Administrador)** | | | | |
| `GET` | `/v1/admin/users` | Obtiene una lista paginada de todos los usuarios. | **Query:** `role`, `status`, `page` | Requiere Token (Admin) |
| `PATCH` | `/v1/admin/users/{userId}/status` | Bloquea o desbloquea un usuario. | **Path:** `userId` | Requiere Token (Admin) |
| `GET` | `/v1/admin/reports/earnings` | Genera un reporte de ganancias por periodo. | **Query:** `start_date`, `end_date`, `role` | Requiere Token (Admin) |
| `DELETE` | `/v1/admin/ratings/{ratingId}` | Elimina una calificación por motivos de moderación. | **Path:** `ratingId` | Requiere Token (Admin) |

---

## Flujos de Uso

### 1. Solicitud, Aceptación y Finalización de un Viaje

1.  **Pasajero solicita viaje:** Llama a `POST /v1/rides`. El servidor responde con el viaje en estado **"Buscando Conductor"** (`201 Created`).
2.  **Conductor acepta:** El conductor obtiene solicitudes cercanas con `GET /v1/rides/available`. Luego llama a `POST /v1/rides/{rideId}/accept`. El estado cambia a **"Aceptado"**.
3.  **Conductor inicia y finaliza:**
    * El conductor marca el inicio: `POST /v1/rides/{rideId}/start`. Estado: **"En Curso"**.
    * El conductor finaliza: `POST /v1/rides/{rideId}/complete`. Estado: **"Finalizado"**. Esto dispara el procesamiento del pago.

---

### 2. Cancelación de un Viaje y Devolución Parcial

1.  **Cancelación:** El Pasajero o Conductor llama a `PATCH /v1/rides/{rideId}/cancel`.
2.  **Lógica del servidor:** El servidor verifica las políticas de cancelación.
3.  **Respuesta:** El servidor responde `200 OK`, el estado del viaje cambia a **"Cancelado"**. Si hubo penalización/devolución, se incluye un resumen de la transacción en la respuesta.
4.  **Verificación:** El Pasajero puede verificar el detalle con `GET /v1/rides/{rideId}`.

---

### 3. Calificación de Conductor y Pasajero

1.  **Calificación del Conductor:** El Pasajero (después de la finalización) llama a `POST /v1/ratings` con el `ride_id` y el `target_user_id` del conductor. (`201 Created`).
2.  **Calificación del Pasajero:** El Conductor llama a `POST /v1/ratings` con el mismo `ride_id` y el `target_user_id` del pasajero. (`201 Created`).
3.  **Consulta de Calificación:** Cualquier usuario puede obtener su promedio y reseñas con `GET /v1/ratings/me`.

---

## Decisiones de Diseño y Justificación

La API se diseñó siguiendo los principios **RESTful** (uso de sustantivos plurales y métodos HTTP semánticos). Se implementó **versionado** (`/v1/`) para garantizar la retrocompatibilidad futura. La **seguridad** se maneja mediante tokens **JWT** y autorización basada en el **rol** del usuario. Las acciones no-CRUD se representan con **sub-recursos** (ej: `/rides/{rideId}/accept`) para mantener la claridad.

---

## Manejo de Errores

Se utiliza el esquema de códigos de estado HTTP estándar con un cuerpo JSON detallado.

```json
{
  "status": 404,
  "code": "RESOURCE_NOT_FOUND",
  "message": "El viaje con ID '123' no fue encontrado.",
  "details": "Verifique el identificador proporcionado."
}
```

| Código HTTP | Código de Error Interno | Situación Correspondiente |
| :--- | :--- | :--- |
| **400 Bad Request** | `INVALID_INPUT` | El cuerpo de la solicitud tiene campos inválidos o faltantes (ej: coordenadas mal formadas). |
| **401 Unauthorized** | `INVALID_TOKEN` | Se intenta acceder a un recurso privado sin un token válido o con uno expirado. |
| **403 Forbidden** | `FORBIDDEN_ACTION` | Un usuario Pasajero intenta usar un endpoint exclusivo de Conductor o Administrador. |
| **409 Conflict** | `RIDE_STATE_CONFLICT` | Se intenta realizar una acción en un estado de recurso inválido (ej: intentar cancelar un viaje ya finalizado). |
| **503 Service Unavailable** | `NO_DRIVERS_AVAILABLE` | Error al solicitar un viaje debido a la ausencia de conductores cercanos disponibles. |


## Propuestas de Mejora
1. **Integración de WebSockets/SSE**: Para la comunicación de la ubicación en tiempo real y notificaciones, mejorando la experiencia de usuario y reduciendo la carga de `polling HTTP`.

2. **Sistema de Tarifas Dinámicas**: Añadir un endpoint de estimación `(GET /v1/fares/estimate)` que considere la demanda y la oferta para calcular tarifas flexibles.

3. **Gestión de Métodos de Pago**: Crear un recurso `/v1/users/me/payment_methods` para la gestión de múltiples métodos de pago por parte del usuario.

4. **Sistema de Disputas**: Implementar un recurso `/v1/disputes` para que los usuarios puedan registrar y dar seguimiento a quejas o problemas de cobro.