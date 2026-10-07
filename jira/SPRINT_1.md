# SPRINT_1.md — Desglose Operativo del Sprint 1: Historias de Usuario y Subtareas Técnicas

> **Proyecto:** Microservicio de Gestión de Entregas y Despachos (Grupo H / ERP Corporativo)  
> **Sprint:** 1  
> **Fuente de Trazabilidad:** Jira Cloud / GitHub  
> **Documento Padre:** [`PROJECT_BACKLOG.md`](./PROJECT_BACKLOG.md)

---

## 1. Alcance y Épicas Cubiertas en el Sprint 1

En este Sprint 1 se abordan los cimientos de infraestructura, seguridad, persistencia, los primeros catálogos maestros y la arquitectura inicial de las aplicaciones cliente (Web y Móvil):
* **Épica 1.0 (`ES-4`):** Configuración Base, Persistencia y Seguridad
* **Épica 2.0 (`ES-5`):** Gestión de Catálogos Operativos
* **Épica 3.0 (`ES-6`):** Aplicación Móvil y Persistencia Offline (Cimientos concluidos: `ES-27` [RF-U01], `ES-79` [RF-U13], `ES-77` [RF-U11], `ES-34` [RF-U08])

---

## 2. Historias de Usuario y Subtareas Técnicas

---

### ÉPICA 1.0 — Configuración Base, Persistencia y Seguridad (`ES-4`)

#### `ES-12` — Estructuración de la Base de Datos Relacional y Entorno Backend [RF-A20]
* **Épica:** 1.0 Configuración Base, Persistencia y Seguridad (`ES-4`)
* **Asignado (PE):** Jaime | **Story Points (Spe):** 5 | **Estado:** Finalizada (Done)
* **Fechas:** Inicio (FI): 14 de Sep | Vencimiento (FV): 16 de Sep
* **Dependencias:** Bloquea a (B): `ES-13`, `ES-17`, `ES-21`, `ES-22`, `ES-23`, `ES-24`, `ES-25`, `ES-26`, `ES-79` | Bloqueado por (BP): Ninguna (Cimiento raíz del sistema)

> **Como** Administrador del sistema / Arquitecto de Software  
> **Quiero** estructurar, modelar y versionar el esquema relacional en PostgreSQL mediante migraciones automáticas y TypeORM/Prisma  
> **Para** contar con persistencia estructurada, integridad referencial y desacople de esquemas frente al resto del ERP.

* **Criterios de Aceptación (GWT):**
  * **Escenario 1 (Migración exitosa):**  
    **Dado** el modelo de datos de usuarios, roles, catálogos y despachos en TypeORM/Prisma,  
    **cuando** se ejecuta el script de migración en NestJS,  
    **entonces** las tablas, claves foráneas e índices se crean en PostgreSQL sin errores.
  * **Escenario 2 (Carga de datos semilla):**  
    **Dado** el despliegue inicial de la base de datos,  
    **cuando** se corre el comando de seeders,  
    **entonces** se insertan automáticamente los roles del sistema (coordinador, supervisor, repartidor, cliente).

| ID Jira | Subtarea | Descripción Técnica y Tareas a Realizar | Asignado (PE) | Estimación | Estado |
| :--- | :--- | :--- | :---: | :---: | :---: |
| `ES-82` | `ST-12.1` | **Definir entidades de BD y relaciones relacionales:**<br>• Crear entidades TypeScript para usuarios, roles, catálogos operativos y despachos.<br>• Configurar claves primarias (UUID), claves foráneas y restricciones UNIQUE (correos, códigos de seguimiento).<br>• Definir índices de base de datos para optimizar consultas frecuentes por estado de despacho y fecha.<br>• Validar la generación y compilación correcta de los tipos de datos en NestJS. | Jaime | 6h | Finalizada |
| `ES-83` | `ST-12.2` | **Crear scripts de migración versionada y seeders:**<br>• Generar migración inicial de tablas con TypeORM / Prisma CLI.<br>• Implementar script de reversión (rollback) para deshacer cambios sin pérdida de integridad.<br>• Desarrollar seeders para la carga de roles del sistema (coordinador, supervisor, repartidor, cliente).<br>• Validar ejecución exitosa de comandos `migration:run` y `seed` en base de datos limpia. | Jaime | 4h | Finalizada |
| `ES-84` | `ST-12.3` | **Configurar pool de conexiones PostgreSQL y variables de entorno:**<br>• Configurar `@nestjs/config` con validación estricta de variables `.env` (host, puerto, credenciales, nombre de BD).<br>• Ajustar límites de tamaño del connection pool y tiempos de espera de consultas (timeout).<br>• Implementar reintentos automáticos de conexión ante caídas transitorias de red.<br>• Validar verificación de estado (health check) de la conexión a la base de datos. | Jaime | 2h | Finalizada |

---

#### `ES-13` — Autenticación Centralizada JWT y Bloqueo por Reintentos [RF-A21]
* **Épica:** 1.0 Configuración Base, Persistencia y Seguridad (`ES-4`)
* **Asignado (PE):** Jaime | **Story Points (Spe):** 5 | **Estado:** Finalizada (Done)
* **Fechas:** Inicio (FI): 17 de Sep | Vencimiento (FV): 20 de Sep
* **Dependencias:** Bloquea a (B): `ES-14`, `ES-15`, `ES-16`, `ES-18`, `ES-27` | Bloqueado por (BP): `ES-12`

> **Como** Usuario del Portal Administrativo  
> **Quiero** autenticarme mediante credenciales encriptadas con emisión de tokens JWT y protección ante reintentos fallidos  
> **Para** acceder a los módulos autorizados de mi perfil y evitar accesos no autorizados por fuerza bruta.

* **Criterios de Aceptación (GWT):**
  * **Escenario 1 (Autenticación exitosa):**  
    **Dado** un usuario registrado y activo,  
    **cuando** envía credenciales válidas a `POST /auth/login`,  
    **entonces** el servidor retorna código 200 con un Access Token JWT firmado y el objeto de perfil (id, rol).
  * **Escenario 2 (Bloqueo temporal por reintentos fallidos - RF-A21):**  
    **Dado** un usuario que falla su contraseña 5 veces consecutivas,  
    **cuando** intente un siguiente intento,  
    **entonces** el sistema responde con error 423 (Locked) y bloquea temporalmente la cuenta por 15 minutos en base de datos/Redis.

| ID Jira | Subtarea | Descripción Técnica y Tareas a Realizar | Asignado (PE) | Estimación | Estado |
| :--- | :--- | :--- | :---: | :---: | :---: |
| `ES-85` | `ST-13.1` | **Servicio de hashing bcrypt y emisión de JWT en NestJS:**<br>• Configurar `@nestjs/jwt` y `@nestjs/passport` con claves secretas inyectadas por variables de entorno.<br>• Implementar servicio de hashing de contraseñas con factor de coste seguro (salt rounds = 10/12).<br>• Desarrollar endpoint `/auth/login` retornando Access Token y payload de usuario (id, rol).<br>• Validar la correcta deserialización del token mediante la estrategia `JwtStrategy`. | Jaime | 8h | Finalizada |
| `ES-86` | `ST-13.2` | **Middleware de control de intentos fallidos y bloqueo temporal:**<br>• Añadir campos de control de intentos fallidos (`failed_attempts`, `locked_until`) en la entidad de usuarios.<br>• Implementar interceptor o servicio que incremente el contador ante credenciales inválidas.<br>• Configurar la regla de negocio de bloqueo por 15 minutos al alcanzar el 5to intento.<br>• Retornar respuesta HTTP 423 Locked y limpiar contador tras expirar el bloqueo o login exitoso. | Jaime | 6h | Finalizada |
| `ES-87` | `ST-13.3` | **Pruebas unitarias de autenticación y bloqueo temporal:**<br>• Escribir tests unitarios para validación de hashes de contraseña correctos e incorrectos.<br>• Simular escenarios de 5 intentos fallidos consecutivos validando la activación del bloqueo temporal.<br>• Verificar la correcta emisión y estructura del token JWT en escenarios de éxito.<br>• Validar cobertura de código mínima requerida para el servicio de autenticación. | Jaime | 4h | Finalizada |
| `ES-146` | `ST-13.4` | **Frontend Web: Pantalla de Login, formulario con React Hook Form y AuthContext:**<br>• Crear la página `/login` con diseño corporativo y campos de correo y contraseña.<br>• Validar inputs con React Hook Form y Zod/Yup mostrando mensajes de error.<br>• Conectar con el endpoint `POST /auth/login` y almacenar el Access Token en el cliente.<br>• Crear el contexto global de autenticación (`AuthContext`) para proteger rutas privadas del dashboard.<br>• Manejar errores de credenciales inválidas (401) y advertencia visual de cuenta bloqueada (423). | Pardo | 5h | Finalizada |

---

#### `ES-14` — Control de Acceso Basado en Roles (RBAC) y Guards [RF-A22]
* **Épica:** 1.0 Configuración Base, Persistencia y Seguridad (`ES-4`)
* **Asignado (PE):** Jaime | **Story Points (Spe):** 5 | **Estado:** Finalizada (Done)
* **Fechas:** Inicio (FI): 22 de Sep | Vencimiento (FV): 24 de Sep
* **Dependencias:** Bloquea a (B): `ES-19`, `ES-72`, `ES-74`, `ES-48`, `ES-57`, `ES-37`, `ES-59`, `ES-68` | Bloqueado por (BP): `ES-13`

> **Como** Administrador de seguridad / Desarrollador backend  
> **Quiero** proteger las rutas y endpoints del microservicio mediante Guards de autorización basados en roles (RBAC)  
> **Para** garantizar que cada perfil (Coordinador, Supervisor, Repartidor y Cliente) acceda exclusivamente a las operaciones autorizadas.

* **Criterios de Aceptación (GWT):**
  * **Escenario 1 (Acceso autorizado según rol):**  
    **Dado** un usuario autenticado con token JWT válido y rol asignado (coordinador o supervisor),  
    **cuando** realiza una petición a un endpoint protegido con `@Roles(...)` correspondiente a su rol,  
    **entonces** el Guard permite la ejecución de la petición retornando HTTP 200/201.
  * **Escenario 2 (Denegación por privilegios insuficientes - RF-A22):**  
    **Dado** un token válido perteneciente a un usuario con rol repartidor,  
    **cuando** intenta consumir un endpoint administrativo restringido (ej. `/api/v1/zonas` o `/api/v1/usuarios`),  
    **entonces** el sistema intercepta la petición y responde inmediatamente con código HTTP 403 Forbidden.
  * **Escenario 3 (Petición sin credenciales de sesión):**  
    **Dado** un cliente sin token o con un token manipulado/expirado,  
    **cuando** intenta consultar una ruta que no esté decorada como `@Public()`,  
    **entonces** el Guard global rechaza la solicitud retornando HTTP 401 Unauthorized.

| ID Jira | Subtarea | Descripción Técnica y Tareas a Realizar | Asignado (PE) | Estimación | Estado |
| :--- | :--- | :--- | :---: | :---: | :---: |
| `ES-88` | `ST-14.1` | **Decorador `@Roles()` y guard de autorización `RolesGuard` en NestJS:**<br>• Crear enumeración estricta de roles del sistema (`COORDINADOR`, `SUPERVISOR`, `REPARTIDOR`, `CLIENTE`).<br>• Implementar el decorador `@Roles(...roles: Role[])` utilizando `SetMetadata` de NestJS.<br>• Desarrollar `RolesGuard` implementando `CanActivate` y extrayendo los roles vía `Reflector`.<br>• Validar compatibilidad y lectura del rol inyectado en el payload del objeto `req.user`. | Jaime | 6h | Finalizada |
| `ES-89` | `ST-14.2` | **Configuración de `JwtAuthGuard` global y decorador `@Public()`:**<br>• Registrar `JwtAuthGuard` como guard global en el módulo principal (`APP_GUARD`).<br>• Crear decorador `@Public()` para omitir validación de token en endpoints de login, health check y tracking.<br>• Configurar el orden de ejecución de guards (`JwtAuthGuard` antes de `RolesGuard`).<br>• Implementar respuestas homogéneas y normalizadas ante fallos 401. | Jaime | 4h | Finalizada |
| `ES-90` | `ST-14.3` | **Pruebas de integración E2E sobre endpoints protegidos (401 y 403):**<br>• Crear suite de tests simulando llamadas con tokens válidos de cada rol.<br>• Validar que peticiones sin header `Authorization` reciban HTTP 401.<br>• Verificar rechazo con HTTP 403 al intentar consumir endpoints administrativos con rol repartidor.<br>• Confirmar que los endpoints públicos respondan sin requerir Bearer token. | Jaime | 4h | Finalizada |

---

#### `ES-15` — Recuperación Segura de Contraseña mediante Tokens Temporales [RF-A23]
* **Épica:** 1.0 Configuración Base, Persistencia y Seguridad (`ES-4`)
* **Asignado (PE):** Joan | **Story Points (Spe):** 3 | **Estado:** Finalizada (Done)
* **Fechas:** Inicio (FI): 23 de Sep | Vencimiento (FV): 26 de Sep
* **Dependencias:** Bloquea a (B): Ninguna | Bloqueado por (BP): `ES-13`

> **Como** Usuario del sistema  
> **Quiero** solicitar el restablecimiento de mi contraseña olvidada mediante un enlace seguro con token temporal  
> **Para** recuperar el acceso a mis herramientas de trabajo de forma autónoma sin comprometer la integridad de mi cuenta.

* **Criterios de Aceptación (GWT):**
  * **Escenario 1 (Generación y vigencia de token seguro):**  
    **Dado** un correo electrónico registrado y activo en el sistema,  
    **cuando** el usuario solicita restablecimiento de contraseña desde la interfaz web,  
    **entonces** el backend genera un token criptográfico aleatorio, almacena su hash con vigencia máxima de 15 minutos y responde confirmando la solicitud (sin revelar existencia de cuenta).
  * **Escenario 2 (Cambio de contraseña exitoso - RF-A23):**  
    **Dado** un token de restablecimiento vigente y no utilizado previamente,  
    **cuando** el usuario envía la nueva contraseña cumpliendo las políticas de seguridad,  
    **entonces** la clave se actualiza con hash bcrypt en PostgreSQL, el token queda invalidado de forma permanente y se responde con HTTP 200.
  * **Escenario 3 (Manejo de token inválido o expirado):**  
    **Dado** un token con más de 15 minutos de antigüedad o ya consumido,  
    **cuando** se intenta cambiar la contraseña,  
    **entonces** la solicitud se rechaza con HTTP 400 Bad Request indicando que el enlace ha caducado.

| ID Jira | Subtarea | Descripción Técnica y Tareas a Realizar | Asignado (PE) | Estimación | Estado |
| :--- | :--- | :--- | :---: | :---: | :---: |
| `ES-91` | `ST-15.1` | **Backend: Endpoints de solicitud y restablecimiento de contraseña con token temporal:**<br>• Crear migración/tabla para persistir tokens de reseteo (`token_hash`, `user_id`, `expires_at`, `used`).<br>• Implementar endpoint `POST /auth/forgot-password` generando tokens criptográficos (`crypto.randomBytes`).<br>• Implementar endpoint `POST /auth/reset-password` validando caducidad (15 min) y actualizando hash de clave.<br>• Invalidar el token inmediatamente tras su primer uso exitoso. | Jaime | 6h | Finalizada |
| `ES-92` | `ST-15.2` | **Frontend: Pantallas y formularios de recuperación y cambio de clave en React:**<br>• Diseñar y programar la vista `/forgot-password` para ingreso del correo electrónico.<br>• Crear la vista `/reset-password?token=...` capturando el token por query param.<br>• Implementar formulario con campos "Nueva contraseña" y "Confirmar contraseña" con validación visual.<br>• Integrar feedback al usuario mediante alertas/toasts de éxito, error o token vencido. | Joan | 6h | Finalizada |
| `ES-93` | `ST-15.3` | **Pruebas de integración del ciclo de expiración y un solo uso del token:**<br>• Probar el flujo completo: solicitud $\rightarrow$ recepción $\rightarrow$ actualización $\rightarrow$ login con nueva clave.<br>• Validar que un mismo token no pueda reutilizarse una segunda vez.<br>• Comprobar que transcurridos 15 minutos el enlace sea rechazado con mensaje de caducidad.<br>• Verificar que la sesión anterior se invalide tras el cambio de contraseña. | Joan | 3h | Finalizada |

---

#### `ES-18` — Gestión de Usuarios Internos y Arquitectura Portal Web [RF-A01]
* **Épica:** 1.0 Configuración Base, Persistencia y Seguridad (`ES-4`)
* **Asignado (PE):** Pardo | **Story Points (Spe):** 5 | **Estado:** Finalizada (Done)
* **Fechas:** Inicio (FI): 21 de Sep | Vencimiento (FV): 24 de Sep
* **Dependencias:** Bloquea a (B): `ES-19`, `ES-20` | Bloqueado por (BP): `ES-13`

> **Como** Coordinador de logística / Administrador del sistema  
> **Quiero** registrar, editar y desactivar usuarios internos mediante un formulario web conectado a la base de datos  
> **Para** mantener actualizado el personal operativo y asegurar que solo colaboradores autorizados accedan a la plataforma.

* **Criterios de Aceptación (GWT):**
  * **Escenario 1 (Alta exitosa de usuario):**  
    **Dado** que el coordinador completa los campos obligatorios (nombre, correo corporativo, rol inicial, teléfono) con un correo no registrado,  
    **cuando** confirma el envío del formulario,  
    **entonces** el backend almacena el usuario en PostgreSQL con estado activo (`status: ACTIVE`), genera un registro de auditoría y retorna HTTP 201 Created.
  * **Escenario 2 (Validación de correo duplicado):**  
    **Dado** que se intenta crear un usuario con un email existente,  
    **cuando** se envía la petición,  
    **entonces** el backend rechaza la operación con HTTP 409 Conflict y el frontend despliega una advertencia en el campo de correo.
  * **Escenario 3 (Desactivación lógica con bloqueo de seguridad - RF-A01 / RF-A26):**  
    **Dado** un operador que tiene despachos o rutas asignadas en curso,  
    **cuando** el administrador solicita desactivar la cuenta,  
    **entonces** el sistema impide la baja lógica, emite una alerta indicando reasignar órdenes primero, y solo procede a marcarlo como `INACTIVE` cuando no posea despachos pendientes.

| ID Jira | Subtarea | Descripción Técnica y Tareas a Realizar | Asignado (PE) | Estimación | Estado |
| :--- | :--- | :--- | :---: | :---: | :---: |
| `ES-145` | `ST-18.0` | **Inicialización del proyecto React (Vite, TailwindCSS, Axios) y repositorio GitHub:**<br>• Inicializar proyecto React con Vite y soporte para TypeScript.<br>• Instalar y configurar Tailwind CSS, PostCSS y `tailwind.config.js` con paleta corporativa.<br>• Configurar cliente base de Axios (`apiClient.ts`) parametrizado por variables de entorno.<br>• Configurar linter y formateo de código (ESLint, Prettier).<br>• Crear repositorio GitHub, `.gitignore` y commit inicial a la rama `main`. | Sergio | 2h | Finalizada |
| `ES-100` | `ST-18.1` | **Backend: Endpoints CRUD de usuarios (`/users`) y validación de unicidad en NestJS:**<br>• Crear controlador y servicio en NestJS para métodos `POST`, `GET`, `PUT /users/:id` y `PATCH /users/:id/status`.<br>• Implementar validación en DTO para correo electrónico corporativo único y formato de datos.<br>• Integrar validación relacional: comprobar que el usuario no tenga despachos asignados/en ruta antes de desactivar.<br>• Persistir la fecha de desactivación (`deleted_at`) para garantizar trazabilidad. | Jaime | 6h | Finalizada |
| `ES-101` | `ST-18.2` | **Frontend: Layout administrativo (Router/Sidebar) y formulario modal de usuarios:**<br>• Configurar `react-router-dom` con la estructura base de rutas del portal administrativo.<br>• Desarrollar el componente de Layout administrativo con barra superior y Sidebar lateral.<br>• Diseñar modal accesible con campos: Nombre, Apellidos, Correo, Teléfono y selector de Rol.<br>• Integrar validación con React Hook Form y Yup/Zod para retroalimentación en tiempo real.<br>• Conectar mutaciones con el backend gestionando estados de carga y toasts.<br>• Precargar los datos existentes del colaborador al abrir el formulario en modo edición. | Pardo | 6h | Finalizada |
| `ES-102` | `ST-18.3` | **Frontend: Flujo de desactivación (soft delete) con modal de confirmación y advertencias:**<br>• Construir modal de confirmación destructiva con advertencia de pérdida de acceso.<br>• Capturar y renderizar mensajes de bloqueo en caso de que el backend reporte órdenes activas asignadas.<br>• Actualizar la caché del cliente (React Query / Zustand) para reflejar el nuevo estado sin recargar. | Pardo | 3h | Finalizada |

---

#### `ES-16` — Gestión de Sesión Activa, Refresh Tokens y Timeouts [RF-A24]
* **Épica:** 1.0 Configuración Base, Persistencia y Seguridad (`ES-4`)
* **Asignado (PE):** Jaime | **Story Points (Spe):** 3 | **Estado:** Finalizada (Done)
* **Fechas:** Inicio (FI): 24 de Sep | Vencimiento (FV): 26 de Sep
* **Dependencias:** Bloquea a (B): Ninguna | Bloqueado por (BP): `ES-13`

> **Como** Usuario y Administrador de seguridad del sistema  
> **Quiero** que la sesión del portal web expire de forma automática tras 30 minutos continuos de inactividad y que soporte renovación transparente mediante Refresh Tokens  
> **Para** evitar el secuestro de sesiones en terminales desatendidas protegiendo los datos corporativos de despacho.

* **Criterios de Aceptación (GWT):**
  * **Escenario 1 (Expiración en portal web por inactividad - RF-A24):**  
    **Dado** un usuario autenticado en el portal web administrativo,  
    **cuando** transcurren 30 minutos consecutivos sin registrar actividad física (clics, teclas o scroll),  
    **entonces** el sistema destruye el token en memoria, cierra la sesión automáticamente, redirige al `/login` y presenta una notificación de expiración.
  * **Escenario 2 (Renovación transparente mediante Refresh Token):**  
    **Dado** un usuario con sesión abierta interactuando cuyo Access Token expira (15 min),  
    **cuando** emite una petición HTTP a cualquier recurso protegido,  
    **entonces** el interceptor envía silenciosamente el Refresh Token a `/auth/refresh`, actualiza el token en segundo plano y completa la petición sin interrupciones.
  * **Escenario 3 (Rechazo ante Refresh Token revocado o expirado):**  
    **Dado** un Refresh Token vencido o invalidado administrativamente en base de datos,  
    **cuando** el cliente intenta renovar credenciales,  
    **entonces** el backend responde HTTP 401 Unauthorized y la aplicación fuerza el retorno al login.

| ID Jira | Subtarea | Descripción Técnica y Tareas a Realizar | Asignado (PE) | Estimación | Estado |
| :--- | :--- | :--- | :---: | :---: | :---: |
| `ES-94` | `ST-16.1` | **Backend: Servicio de rotación y revocación de Refresh Tokens en NestJS:**<br>• Crear migración para persistir tokens de refresco hasheados con fecha de caducidad en PostgreSQL.<br>• Implementar endpoint `POST /auth/refresh` con validación criptográfica y rotación obligatoria de token.<br>• Crear endpoint `POST /auth/logout` para revocar el Refresh Token activo en base de datos.<br>• Configurar revocación en cascada de tokens ante detección de intentos de reutilización previa. | Jaime | 5h | Finalizada |
| `ES-95` | `ST-16.2` | **Frontend Web: Hook de detección de inactividad y modal de advertencia en React:**<br>• Crear hook `useIdleTimer` escuchando eventos globales (`mousemove`, `keydown`, `scroll`, `click`).<br>• Configurar umbral de inactividad a 30 minutos continuos.<br>• Diseñar modal de advertencia a los 28 minutos con cuenta regresiva de 2 minutos para extender sesión.<br>• Ejecutar purga de sesión (`localStorage`), limpiar estado global y redirigir a `/login`. | Pardo | 4h | Finalizada |
| `ES-96` | `ST-16.3` | **Frontend Web: Interceptor Axios para renovación silenciosa de tokens (Refresh Flow):**<br>• Configurar Axios Response Interceptor para capturar respuestas con estado HTTP 401.<br>• Implementar cola de peticiones concurrentes pausadas mientras se procesa la renovación del token.<br>• Reintentar automáticamente las peticiones inyectando el nuevo Bearer token devuelto por `/auth/refresh`.<br>• Limpiar tokens y redirigir al login si el endpoint de refresco falla o retorna 401/403. | Pardo | 3h | Finalizada |

---

#### `ES-17` — Políticas de Robustez y Caducidad de Contraseñas [RF-A25]
* **Épica:** 1.0 Configuración Base, Persistencia y Seguridad (`ES-4`)
* **Asignado (PE):** Jaime | **Story Points (Spe):** 2 | **Estado:** Finalizada (Done)
* **Fechas:** Inicio (FI): 22 de Sep | Vencimiento (FV): 23 de Sep
* **Dependencias:** Bloquea a (B): Ninguna | Bloqueado por (BP): `ES-12`

> **Como** Administrador de seguridad / Coordinador del sistema  
> **Quiero** definir y validar de forma obligatoria las políticas de complejidad y el período de caducidad de contraseñas  
> **Para** garantizar contraseñas robustas y mitigar vulnerabilidades derivadas del uso prolongado de credenciales.

* **Criterios de Aceptación (GWT):**
  * **Escenario 1 (Validación estricta de complejidad en alta/cambio - RF-A25):**  
    **Dado** un formulario de registro o cambio de clave en el portal,  
    **cuando** el usuario introduce una contraseña que no cumple con el estándar (mínimo 8 caracteres, 1 mayúscula, 1 minúscula, 1 número y 1 carácter especial),  
    **entonces** el DTO del backend rechaza la petición con HTTP 400 Bad Request y el frontend resalta en rojo las reglas no superadas.
  * **Escenario 2 (Control de caducidad periódica de contraseñas):**  
    **Dado** un usuario cuya contraseña ha superado el período máximo de vigencia configurado (90 días),  
    **cuando** inicia sesión con credenciales correctas,  
    **entonces** el backend marca `mustChangePassword: true`, forzando la redirección inmediata a la vista `/force-change-password`.
  * **Escenario 3 (Prevención de reutilización de contraseñas recientes):**  
    **Dado** un usuario cambiando su contraseña caducada,  
    **cuando** intenta registrar la misma contraseña actual o una de sus últimas 3 contraseñas utilizadas,  
    **entonces** el sistema deniega el cambio emitiendo un mensaje de error con la restricción de historial.

| ID Jira | Subtarea | Descripción Técnica y Tareas a Realizar | Asignado (PE) | Estimación | Estado |
| :--- | :--- | :--- | :---: | :---: | :---: |
| `ES-97` | `ST-17.1` | **Backend: Validadores DTO con class-validator y control de caducidad en NestJS:**<br>• Crear validación en DTO para: $\ge$ 8 caracteres, mayúscula, minúscula, número y símbolo.<br>• Añadir columna `password_changed_at` en tabla usuarios para auditar la fecha de último cambio.<br>• Modificar `/auth/login` para calcular si transcurrieron > 90 días y emitir cambio obligatorio.<br>• Crear tabla `password_history` para almacenar hashes históricos y validar no repetir últimas 3. | Jaime | 5h | Finalizada |
| `ES-98` | `ST-17.2` | **Frontend Web: Checklist interactivo de complejidad y vista de cambio obligatorio en React:**<br>• Construir componente reusable con checklist visual que marque en verde los requisitos en tiempo real.<br>• Bloquear el botón de envío del formulario si no se cumplen todos los requisitos de complejidad.<br>• Crear pantalla protegida `/force-change-password` accesible únicamente ante cambio obligatorio.<br>• Implementar alertas contextuales de error ante respuestas HTTP 400 del servidor. | Pardo | 4h | Finalizada |
| `ES-99` | `ST-17.3` | **Pruebas unitarias de validación de contraseñas y caducidad:**<br>• Escribir tests unitarios para el validador DTO cubriendo contraseñas débiles, cortas y correctas.<br>• Simular inicios de sesión con usuarios de más de 90 días de antigüedad validando el flag de caducidad.<br>• Validar que el endpoint rechace hashes equivalentes almacenados en `password_history`. | Jaime | 3h | Finalizada |

---

#### `ES-19` — Asignación y Modificación de Roles y Permisos [RF-A27]
* **Épica:** 1.0 Configuración Base, Persistencia y Seguridad (`ES-4`)
* **Asignado (PE):** Jaime | **Story Points (Spe):** 2 | **Estado:** Finalizada (Done)
* **Fechas:** Inicio (FI): 27 de Sep | Vencimiento (FV): 29 de Sep
* **Dependencias:** Bloquea a (B): Ninguna | Bloqueado por (BP): `ES-14`, `ES-16`

> **Como** Administrador del sistema / Oficial de accesos y seguridad  
> **Quiero** asignar y actualizar el rol y los permisos operativos de cualquier cuenta interna desde el panel administrativo  
> **Para** adaptar los privilegios del colaborador ante cambios de puesto o transferencias operativas.

* **Criterios de Aceptación (GWT):**
  * **Escenario 1 (Modificación exitosa de rol - RF-A27):**  
    **Dado** un usuario existente seleccionado en el panel,  
    **cuando** el administrador cambia su rol (ej. de repartidor a supervisor) y confirma los cambios,  
    **entonces** el registro en base de datos se actualiza de inmediato, se invalidan los tokens de sesión previos para forzar el nuevo rol en el próximo login y se responde HTTP 200 OK.
  * **Escenario 2 (Protección contra auto-revocación del rol Administrador):**  
    **Dado** un usuario autenticado con rol administrador,  
    **cuando** intenta revocar o degradar su propio rol en el panel,  
    **entonces** el sistema bloquea la acción notificando que no es posible remover los permisos de la propia cuenta en sesión.
  * **Escenario 3 (Visualización dinámica de permisos asignados):**  
    **Dado** que el operador selecciona un rol en el menú desplegable del formulario,  
    **cuando** cambia de opción,  
    **entonces** el frontend despliega dinámicamente el listado de módulos y permisos autorizados para dicho perfil.

| ID Jira | Subtarea | Descripción Técnica y Tareas a Realizar | Asignado (PE) | Estimación | Estado |
| :--- | :--- | :--- | :---: | :---: | :---: |
| `ES-103` | `ST-19.1` | **Backend: Endpoint `PATCH /users/:id/role` con validación de roles y revocación de sesión:**<br>• Crear endpoint `PATCH /users/:id/role` protegido con Guard exclusivo de Administrador.<br>• Validar que el id a modificar no coincida con el usuario autenticado (`req.user.id`) si implica pérdida de privilegios administrativos.<br>• Forzar la invalidación de los refresh tokens activos del usuario modificado en base de datos.<br>• Emitir evento o log de auditoría del cambio de privilegios. | Jaime | 4h | Finalizada |
| `ES-104` | `ST-19.2` | **Frontend: Selector de roles con panel interactivo de matriz de permisos en React:**<br>• Crear componente selector desplegable con roles disponibles (Coordinador, Supervisor, Repartidor).<br>• Diseñar panel colapsable que liste los módulos accesibles según el rol seleccionado.<br>• Deshabilitar la opción de auto-modificación si el usuario visualiza su propia cuenta.<br>• Conectar petición al backend con notificación de confirmación de cambio exitoso. | Pardo | 4h | Finalizada |
| `ES-105` | `ST-19.3` | **Backend/Frontend: Pruebas de integración para validación de roles y reglas de auto-degradación:**<br>• Escribir prueba unitaria comprobando que un usuario no pueda degradar su propio rol.<br>• Validar que tras el cambio de rol, el usuario modificado reciba un token con los nuevos claims al volver a loguearse.<br>• Comprobar rechazo 403 ante peticiones sin credenciales de administrador. | Jaime | 2h | Finalizada |

---

#### `ES-20` — Listado de Usuarios con Filtros por Rol y Estado [RF-A28]
* **Épica:** 1.0 Configuración Base, Persistencia y Seguridad (`ES-4`)
* **Asignado (PE):** Pardo | **Story Points (Spe):** 3 | **Estado:** Finalizada (Done)
* **Fechas:** Inicio (FI): 01 de Oct | Vencimiento (FV): 02 de Oct
* **Dependencias:** Bloquea a (B): Ninguna | Bloqueado por (BP): `ES-18`

> **Como** Coordinador de logística / Auditor del sistema  
> **Quiero** consultar la tabla completa de usuarios con filtros avanzados por rol, estado y buscador por nombre/correo  
> **Para** auditar rápidamente la nómina y localizar operadores disponibles para la asignación de despachos.

* **Criterios de Aceptación (GWT):**
  * **Escenario 1 (Filtrado combinado y búsqueda en tiempo real - RF-A28):**  
    **Dado** la vista de listado de usuarios en el portal web,  
    **cuando** el usuario introduce un término de búsqueda (ej. "Carlos") y selecciona los filtros de estado "Inactivo" y rol "Repartidor",  
    **entonces** la tabla actualiza los resultados mostrando exclusivamente los registros que cumplan todos los filtros.
  * **Escenario 2 (Paginación eficiente y ordenamiento):**  
    **Dado** un volumen representativo de usuarios registrados,  
    **cuando** se carga la pantalla o se navega entre páginas (10 registros por página),  
    **entonces** el backend responde con paginación server-side retornando únicamente el segmento solicitado, el total de registros y la cantidad total de páginas.
  * **Escenario 3 (Indicadores visuales de estado operativo):**  
    **Dado** el renderizado de la tabla en pantalla,  
    **cuando** un usuario se encuentra activo, inactivo o con bloqueo temporal,  
    **entonces** el registro exhibe un badge de color identificatorio (verde para activo, gris para inactivo, rojo para bloqueado) junto a la fecha de su último acceso.

| ID Jira | Subtarea | Descripción Técnica y Tareas a Realizar | Asignado (PE) | Estimación | Estado |
| :--- | :--- | :--- | :---: | :---: | :---: |
| `ES-106` | `ST-20.1` | **Backend: Paginación server-side y filtros dinámicos en endpoint `GET /users`:**<br>• Configurar parámetros query: `page`, `limit`, `search`, `role`, `status`, `sortBy`, `order`.<br>• Implementar QueryBuilder para aplicar filtros condicionales sin inyección SQL.<br>• Optimizar índices en columnas de búsqueda frecuente (`email`, `status`, `role`).<br>• Estructurar respuesta estándar: `{ data: User[], meta: { total, page, lastPage } }`. | Jaime | 4h | Finalizada |
| `ES-107` | `ST-20.2` | **Frontend: Tabla responsiva con paginación, debounce de búsqueda y badges de estado:**<br>• Implementar componente de tabla con cabeceras ordenables (Nombre, Correo, Rol, Estado, Último Acceso).<br>• Integrar barra de búsqueda con debounce (300ms) para evitar sobrecarga del servidor.<br>• Añadir selectores desplegables para filtrar por Rol y Estado.<br>• Diseñar badges cromáticos para representar los estados operativos. | Pardo | 5h | Finalizada |
| `ES-108` | `ST-20.3` | **Frontend: Gestión de estados vacíos (empty states), skeleton loading y errores:**<br>• Crear componente de carga tipo Skeleton reflejando la estructura de las filas de la tabla.<br>• Diseñar vista para "Sin resultados encontrados" con botón de limpieza rápida de filtros.<br>• Configurar alertas y botón de reintento automático ante fallas de red o respuestas HTTP 500. | Pardo | 3h | Finalizada |

---

### ÉPICA 2.0 — Gestión de Catálogos Operativos (`ES-5`)

#### `ES-21` — Catálogo de Zonas Logísticas y Tiempos Estimados [RF-A02]
* **Épica:** 2.0 Gestión de Catálogos Operativos (`ES-5`)
* **Prioridad:** Crítica | **Story Points:** 5 | **Estado:** Finalizada (Done)
* **Dependencias Previas:** `ES-12` (BD Relacional)

> **Como** Coordinador Logístico  
> **Quiero** registrar y parametrizar las zonas de reparto con sus tiempos base de entrega  
> **Para** permitir la asignación correcta de rutas y la estimación de tiempos hacia los clientes.

* **Criterios de Aceptación (GWT):**
  * **Dado** el panel de zonas, **cuando** se envía el nombre, código de zona y tiempo base en minutos a `POST /zones`, **entonces** se almacena en PostgreSQL y se retorna código 201.
  * **Dado** que se consulta el listado de zonas activas, **cuando** el frontend realiza la petición, **entonces** la API entrega la lista paginada y filtrada por estado.

| Subtarea | Descripción Técnica | Asignado | Horas | Estado |
| :--- | :--- | :---: | :---: | :---: |
| `ST-21.1` | Entidad `Zone`, migración TypeORM y endpoints CRUD completos en NestJS | Jaime | 5h | Finalizada |
| `ST-21.2` | Interfaz React: tabla de zonas, badges de estado y modal de alta/edición | Sergio | 5h | Finalizada |

---

#### `ES-22` — Catálogo de Flota Vehicular y Tipos de Unidad [RF-A03]
* **Épica:** 2.0 Gestión de Catálogos Operativos (`ES-5`)
* **Prioridad:** Alta | **Story Points:** 3 | **Estado:** Finalizada (Done)
* **Dependencias Previas:** `ES-21`

> **Como** Encargado de Flota  
> **Quiero** catalogar los vehículos de la empresa especificando placa, volumen máximo y peso límite  
> **Para** asegurar que no se sobrecarguen las unidades durante los despachos.

* **Criterios de Aceptación (GWT):**
  * **Dado** un vehículo nuevo, **cuando** se registra con una placa duplicada, **entonces** el servidor rechaza la transacción con código 409 (Conflict).
  * **Dado** el catálogo vehicular, **cuando** se consulta desde el portal, **entonces** se despliegan las capacidades volumétricas en metros cúbicos y kilogramos.

| Subtarea | Descripción Técnica | Asignado | Horas | Estado |
| :--- | :--- | :---: | :---: | :---: |
| `ST-22.1` | Modelo de datos `Vehicle`, relaciones con transportistas y validación de placa | Jaime | 4h | Finalizada |
| `ST-22.2` | Vistas en React para control de vehículos, estados operativos y asignación | Pardo | 4h | Finalizada |

---

#### `ES-23` — Políticas y Tiempos Límite de Entrega (SLAs) [RF-A04]
* **Épica:** 2.0 Gestión de Catálogos Operativos (`ES-5`)
* **Prioridad:** Media | **Story Points:** 3 | **Estado:** Finalizada (Done)
* **Dependencias Previas:** `ES-21`

> **Como** Gerente de Operaciones  
> **Quiero** configurar las ventanas horarias de entrega según la tipología de servicio (Express, Estándar)  
> **Para** parametrizar los compromisos de entrega y monitorear desvíos.

* **Criterios de Aceptación (GWT):**
  * **Dado** un tipo de servicio logístico, **cuando** se define su SLA en horas, **entonces** el sistema calcula automáticamente la fecha/hora máxima de entrega al generar un pedido.

| Subtarea | Descripción Técnica | Asignado | Horas | Estado |
| :--- | :--- | :---: | :---: | :---: |
| `ST-23.1` | Entidad `SlaPolicy`, cálculo de ventanas hábiles de entrega y endpoints REST | Jaime | 4h | Finalizada |
| `ST-23.2` | Formulario en portal web para parametrización de horas máximas por zona/servicio | Pardo | 3h | Finalizada |

---

#### `ES-24` — Catálogo de Tipificación de Incidencias en Ruta [RF-A05]
* **Épica:** 2.0 Gestión de Catálogos Operativos (`ES-5`)
* **Prioridad:** Media | **Story Points:** 2 | **Estado:** Finalizada (Done)
* **Dependencias Previas:** `ES-21`

> **Como** Coordinador Logístico  
> **Quiero** tipificar los motivos de no entrega (cliente ausente, dirección incorrecta, siniestro)  
> **Para** que los repartidores en calle seleccionen opciones normalizadas desde la app móvil.

* **Criterios de Aceptación (GWT):**
  * **Dado** el catálogo de incidencias, **cuando** se envía una solicitud `GET /catalogs/incidents`, **entonces** retorna la lista activa ordenada por frecuencia de uso.

| Subtarea | Descripción Técnica | Asignado | Horas | Estado |
| :--- | :--- | :---: | :---: | :---: |
| `ST-24.1` | Entidad `IncidentType` y endpoints de sincronización para clientes web y móvil | Jaime | 3h | Finalizada |

---

#### `ES-25` — Catálogo de Motivos de Cancelación y Devolución [RF-A06]
* **Épica:** 2.0 Gestión de Catálogos Operativos (`ES-5`)
* **Prioridad:** Baja | **Story Points:** 2 | **Estado:** Finalizada (Done)
* **Dependencias Previas:** `ES-24`

> **Como** Administrador de Despacho  
> **Quiero** contar con un listado cerrado de causales de devolución  
> **Para** consolidar las estadísticas de devoluciones hacia el almacén central.

* **Criterios de Aceptación (GWT):**
  * **Dado** un paquete no entregado, **cuando** se marca como devuelto, **entonces** se valida que el código de causal pertenezca al catálogo activo en base de datos.

| Subtarea | Descripción Técnica | Asignado | Horas | Estado |
| :--- | :--- | :---: | :---: | :---: |
| `ST-25.1` | Entidad `CancellationReason`, seeds operativos y endpoints de consulta | Jaime | 3h | Finalizada |

---

#### `ES-26` — Catálogo de Sucursales y Centros de Distribución [RF-A07]
* **Épica:** 2.0 Gestión de Catálogos Operativos (`ES-5`)
* **Prioridad:** Media | **Story Points:** 3 | **Estado:** Finalizada (Done)
* **Dependencias Previas:** `ES-21`

> **Como** Jefe de Almacén  
> **Quiero** registrar los centros de distribución (CEDIS) y sucursales con su geolocalización  
> **Para** definir los puntos de partida de las rutas de despacho.

* **Criterios de Aceptación (GWT):**
  * **Dado** el registro de una sucursal, **cuando** se suministra latitud y longitud válidas, **entonces** se guarda el punto geográfico y su radio de cobertura.

| Subtarea | Descripción Técnica | Asignado | Horas | Estado |
| :--- | :--- | :---: | :---: | :---: |
| `ST-26.1` | Estructura de tabla `Warehouse`, geocodificación de puntos y endpoints REST | Jaime | 4h | Finalizada |
| `ST-26.2` | Componente de mapa en React para visualización y alta de almacenes centrales | Pardo | 4h | Finalizada |

---

### ÉPICA 3.0 — Aplicación Móvil y Persistencia Offline (`ES-6`) [Cimientos Sprint 1]

#### `ES-79` — Almacenamiento Local Offline y Motor Embebido SQLite [RF-U13]
* **Épica:** 3.0 Aplicación Móvil de Repartidor y Telemetría (`ES-6`)
* **Prioridad:** Crítica / Arquitectura | **Story Points:** 5 | **Estado:** Finalizada (Done)
* **Dependencias Previas:** Inicialización de proyecto móvil por Sergio (`ST-27.1`)

> **Como** Repartidor en zonas con mala conectividad  
> **Quiero** que la aplicación guarde todas las transacciones localmente en una base de datos embebida  
> **Para** operar con fluidez sin interrupciones por pérdida de señal móvil.

* **Criterios de Aceptación (GWT):**
  * **Dado** el dispositivo móvil sin conexión a internet (modo avión), **cuando** se registra un evento de ruta, **entonces** el registro se inserta exitosamente en la tabla `local_events` con `synced = 0`.
  * **Dado** que se restablece la conexión a internet (`NetInfo` retorna `isConnected = true`), **cuando** el despachador de la cola se activa, **entonces** procesa los eventos en orden FIFO y los marca con `synced = 1`.

| Subtarea | Descripción Técnica | Asignado | Horas | Estado |
| :--- | :--- | :---: | :---: | :---: |
| `ST-79.1` | Configuración de motor SQLite/WatermelonDB, esquemas de tablas y migraciones locales | Joan | 6h | Finalizada |
| `ST-79.2` | Servicio de detección de conectividad (`NetInfo`) y despachador de cola FIFO | Joan | 5h | Finalizada |

---

#### `ES-27` — Inicio de Sesión Móvil para Repartidor / Transportista [RF-U01]
* **Épica:** 3.0 Aplicación Móvil de Repartidor y Telemetría (`ES-6`)
* **Prioridad:** Alta | **Story Points:** 5 | **Estado:** Finalizada (Done)
* **Dependencias Previas:** `ES-13` (Endpoints de login terminados)

> **Como** Repartidor / Transportista  
> **Quiero** ingresar a la app móvil utilizando mis credenciales asignadas  
> **Para** visualizar mi hoja de ruta y despachos asignados para el turno de trabajo.

* **Criterios de Aceptación (GWT):**
  * **Dado** que el repartidor introduce su usuario y clave en la app, **cuando** presiona "Iniciar Sesión", **entonces** se consume `POST /auth/login` y el JWT se persiste de forma segura en hardware.
  * **Dado** un cierre forzado de la aplicación, **cuando** el usuario vuelve a abrirla, **entonces** la app lee el token desde `expo-secure-store` y lo redirige a la pantalla de ruta sin pedir login de nuevo.

| Subtarea | Descripción Técnica | Asignado | Horas | Estado |
| :--- | :--- | :---: | :---: | :---: |
| `ST-27.1` | Setup Expo React Native, estructura de carpetas, temas con NativeWind y UI Login | Sergio | 6h | Finalizada |
| `ST-27.2` | Conexión con Axios, almacenamiento en `expo-secure-store` y manejo de sesión | Sergio | 4h | Finalizada |

---

#### `ES-77` — Servicio en Segundo Plano de Telemetría GPS [RF-U11]
* **Épica:** 3.0 Aplicación Móvil de Repartidor y Telemetría (`ES-6`)
* **Prioridad:** Alta | **Story Points:** 5 | **Estado:** Finalizada (Done)
* **Dependencias Previas:** `ES-27` (Inicio de sesión completado)

> **Como** Supervisor de Operaciones  
> **Quiero** que la aplicación móvil reporte periódicamente las coordenadas del repartidor  
> **Para** calcular estimaciones de llegada precisas y monitorear la seguridad de la carga.

* **Criterios de Aceptación (GWT):**
  * **Dado** el turno iniciado, **cuando** la aplicación pasa a segundo plano o se apaga la pantalla, **entonces** el Foreground Service de Android permanece activo mostrando la notificación requerida por el sistema operativo.
  * **Dado** que el repartidor está en movimiento, **cuando** transcurren 30 segundos o avanza 50 metros, **entonces** se capturan las coordenadas (`lat`, `lng`, `speed`, `timestamp`) para su posterior sincronización.

| Subtarea | Descripción Técnica | Asignado | Horas | Estado |
| :--- | :--- | :---: | :---: | :---: |
| `ST-77.1` | Configuración de Foreground Service, permisos nativos de ubicación y notificación persistente | Sergio | 6h | Finalizada |
| `ST-77.2` | Algoritmo de muestreo adaptativo de coordenadas y guardado en cola local SQLite | Sergio | 4h | Finalizada |

---

#### `ES-34` — Cierre Formal de Jornada y Reconciliación en App Móvil [RF-U08]
* **Épica:** 3.0 Aplicación Móvil de Repartidor y Telemetría (`ES-6`)
* **Prioridad:** Media | **Story Points:** 3 | **Estado:** Finalizada (Done)
* **Dependencias Previas:** `ES-27` y `ES-77`

> **Como** Repartidor al finalizar mi ruta de trabajo  
> **Quiero** cerrar mi turno formalmente en la aplicación móvil  
> **Para** sincronizar los paquetes restantes, detener el rastreo satelital y cerrar la sesión.

* **Criterios de Aceptación (GWT):**
  * **Dado** que el repartidor presiona "Cerrar Jornada", **cuando** existan paquetes sin entregar ni reportar, **entonces** la app muestra una alerta impidiendo el cierre hasta justificar cada guía.
  * **Dado** que todos los pedidos están cerrados y la cola local está vacía (`synced = 1`), **cuando** se confirma el cierre, **entonces** se apaga el servicio de GPS y se limpia el token de sesión.

| Subtarea | Descripción Técnica | Asignado | Horas | Estado |
| :--- | :--- | :---: | :---: | :---: |
| `ST-34.1` | Pantalla de resumen de jornada (entregados, fallidos, pendientes) y validación de cierre | Sergio | 4h | Finalizada |
| `ST-34.2` | Endpoint `POST /shift/close`, reconciliación en servidor y apagado de Foreground Service | Jaime | 3h | Finalizada |

---

## 3. Matriz Resumen de Carga Horaria y Capacidad — Sprint 1

| Desarrollador | Rol Principal | Subtareas Asignadas (Sprint 1) | Horas Totales Estimadas |
| :--- | :--- | :--- | :---: |
| **Jaime** | Backend Lead | `ST-12.1` (6h), `ST-12.2` (4h), `ST-12.3` (2h), `ST-13.1` (8h), `ST-13.2` (6h), `ST-13.3` (4h), `ST-14.1` (6h), `ST-14.2` (4h), `ST-14.3` (4h), `ST-15.1` (6h), `ST-16.1` (5h), `ST-17.1` (5h), `ST-17.3` (3h), `ST-18.1` (6h), `ST-19.1` (4h), `ST-19.3` (2h), `ST-20.1` (4h), `ST-21.1` (5h), `ST-22.1` (4h), `ST-23.1` (4h), `ST-24.1` (3h), `ST-25.1` (3h), `ST-26.1` (4h), `ST-34.2` (3h) | **95 hrs** |
| **Sergio** | Frontend Lead & Mobile | `ST-18.0` (2h), `ST-21.2` (5h), `ST-27.1` (6h), `ST-27.2` (4h), `ST-77.1` (6h), `ST-77.2` (4h), `ST-34.1` (4h) | **31 hrs** |
| **Pardo** | Frontend Web Developer | `ST-13.4` (5h), `ST-16.2` (4h), `ST-16.3` (3h), `ST-17.2` (4h), `ST-18.2` (6h), `ST-18.3` (3h), `ST-19.2` (4h), `ST-20.2` (5h), `ST-20.3` (3h), `ST-22.2` (4h), `ST-23.2` (3h), `ST-26.2` (4h) | **48 hrs** |
| **Joan** | Scrum Master / Mobile Architecture | `ST-15.2` (6h), `ST-15.3` (3h), `ST-79.1` (6h), `ST-79.2` (5h) | **20 hrs** |
