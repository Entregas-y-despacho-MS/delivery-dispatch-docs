# Requerimientos No Funcionales (RNF)
**Microservicio de Gestión de Entregas y Despachos (Grupo H)**

Los requerimientos no funcionales definen los atributos de calidad, rendimiento, seguridad y restricciones arquitectónicas del microservicio:

---

## 📋 Matriz de Requerimientos No Funcionales

| Categoría | Descripción y Criterio de Aceptación | Alcance |
|---|---|---|
| **Disponibilidad** | Operar **24/7 con disponibilidad mínima del 99.5 %** para garantizar la continuidad operativa de la última milla. | General / Backend |
| **Rendimiento** | Las actualizaciones de estado y ubicación deben reflejarse en el tablero administrativo y portal de seguimiento en **menos de 3 segundos**. | General (WebSockets / REST) |
| **Seguridad** | Los datos sensibles del cliente (dirección, teléfono, correo) se cifran **en tránsito (HTTPS/TLS) y en reposo (AES-256)**. Los tokens de autenticación en la app móvil se almacenan exclusivamente en el contenedor seguro del sistema (**Keystore en Android / Keychain en iOS**). | Backend / Mobile / Web |
| **Trazabilidad** | Historial auditable completo e inmutable de todos los eventos de cada despacho (creación, asignación, ruta, entrega) y de cada acción administrativa (**quién, cuándo, qué estado, IP**). | General / Base de Datos |
| **Interoperabilidad** | Comunicación con los demás microservicios del ERP corporativo mediante **mensajería asíncrona con mensajes JSON** (Google Cloud Pub/Sub, simulada con su emulador durante el proyecto), con desacoplamiento total a nivel de esquemas de bases de datos. | Backend |
| **Escalabilidad** | Capacidad de soportar picos de operación (días de pago, campañas comerciales, fin de mes) sin degradación de tiempos de respuesta. | Backend / Base de Datos |
| **Compatibilidad Móvil** | La aplicación móvil para repartidores debe ser compatible con dispositivos **Android 10 (API level 29) o superior** e iOS contemporáneo. | App Móvil (Expo SDK) |
| **Eficiencia Energética** | El servicio de geolocalización en segundo plano en la app móvil debe utilizar un **muestreo adaptativo** (basado en distancia y movimiento) para minimizar el consumo de batería durante la jornada laboral. | App Móvil |
| **Optimización de Datos** | Las fotografías de evidencia digital (recibo, estado del producto, firma) tomadas por la app móvil deben **comprimirse en el dispositivo antes del envío HTTP** para reducir el consumo del plan de datos móviles. | App Móvil |

---

## 🔒 Consideraciones Específicas de Seguridad y Sesión

1. **Gestión de Tokens (JWT):**
   * Access tokens con tiempo de expiración corto (ej. 15–30 min).
   * Refresh token rotation para renovaciones automáticas seguras.
   * La app móvil en ruta mantiene persistencia de sesión segura vía SecureStore para evitar cierres de sesión intempestivos sin cobertura.
2. **Bloqueo por Fuerza Bruta:**
   * Bloqueo temporal de cuentas tras 5 intentos fallidos consecutivos de inicio de sesión (RF-A21).
3. **Control de Acceso Basado en Roles (RBAC):**
   * `Coordinador`: Gestión logística, despacho, catálogos, auditoría y administración de usuarios.
   * `Supervisor`: Monitoreo en vivo de flota, disponibilidad vehicular, mantenimientos y reasignaciones.
   * `Repartidor`: Acceso restringido únicamente a pedidos asignados en su ruta activa y reportes de entrega.
   * `Cliente Final`: Acceso temporal sin credenciales autenticado mediante token criptográfico único por pedido.
