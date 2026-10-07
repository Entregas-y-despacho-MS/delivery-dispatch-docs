# PROJECT_BACKLOG.md — Backlog y Plan de Avance del Proyecto

> **Proyecto:** Microservicio de Gestión de Entregas y Despachos (Grupo H / ERP Corporativo)  
> **Fuente de Trazabilidad:** Jira Cloud / GitHub  
> **Objetivo:** Documentar la estructura técnica, catálogo de épicas, historias de usuario (HU), políticas de ingeniería y hoja de ruta por sprints.

---

## 1. Estructura y Navegación de la Documentación en Jira

Para mantener la información ordenada y escalable, el desglose detallado de historias de usuario, tareas, criterios GWT y asignaciones horarias se divide por Sprints operativos:

| Documento | Alcance y Contenido Principal | Estado |
| :--- | :--- | :---: |
| 📘 [**`PROJECT_BACKLOG.md`**](./PROJECT_BACKLOG.md) *(este archivo)* | Políticas de Git/Jira, catálogo maestro de las 8 Épicas, fichas de DoD y roadmap global. | **Activo** |
| 🏃‍♂️ [**`SPRINT_1.md`**](./SPRINT_1.md) | **Sprint 1:** Épica 1.0 (Setup, Auth, RBAC, Usuarios), Épica 2.0 (Catálogos) y Épica 3.0 (Cimientos Mobile SQLite). | **En Curso** |
| 🏃‍♂️ [**`SPRINT_2.md`**](./SPRINT_2.md) | **Sprint 2:** Épica 3.0 (App Móvil Repartidor), Épica 5.0 (Despachos y Coordinación Web), Épica 8.0 (Fase 1 ERP). | **Estructurado** |
| 🏃‍♂️ [**`SPRINT_3.md`**](./SPRINT_3.md) | **Sprint 3:** Épica 6.0 (Torre de Control en Vivo / WebSockets) y Épica 4.0 (Portal Tracking Cliente). | **Planificado** |
| 🏃‍♂️ [**`SPRINT_4.md`**](./SPRINT_4.md) | **Sprint 4:** Épica 8.0 (Conciliación ERP / Batch) y Épica 7.0 (Auditoría Forense inmutable y Cierre). | **Planificado** |

---

## 2. Políticas de Ingeniería, Git y Trazabilidad con Jira

### Flujo de Ramas (Feature Branch Workflow)
Queda prohibido el uso de ramas genéricas por desarrollador. Cada incremento técnico debe aislarse en una rama propia originada desde la rama base (`main` o `develop`):

* **Funcionalidad nueva:** `feature/ST-XX-descripcion-corta` (o `feature/ES-XX-...` si la Historia de Usuario es unitaria y ejecutada por un solo dev).
* **Corrección de errores:** `fix/ST-XX-descripcion-corta`.

### Formato Estándar de Commits
Cada confirmación debe prefijarse obligatoriamente con el código de la subtarea o historia para garantizar la vinculación en Jira:

* **Formato:** `ST-XX.X: Verbo en infinitivo y alcance del cambio.`
* **Ejemplo:** `git commit -m "ST-18.2: estructurar router protegido y sidebar administrativo"`

### Ciclo de Vida y Limpieza
1. La subtarea pasa a **In Progress** únicamente al comenzar el desarrollo local (límite WIP: máximo 2 tareas simultáneas por dev).
2. Al finalizar, se abre un **Pull Request (PR)** hacia la rama principal y la tarea pasa a **Code Review**.
### Revisiones de código y Gobernanza de Equipo
* **Gobernanza Base (Estructura aplicada en Sprint 1):**
  * **Pardo:** Product Owner (PO).
  * **Joan:** Scrum Master (SM).
  * **Sergio:** Frontend Lead (Web & Mobile).
  * **Jaime:** Backend Lead (DBA & NestJS).
* **Revisiones de código (Code Review):**
  * **Sergio:** revisa y aprueba el código de Frontend Web y Móvil.
  * **Jaime:** revisa y aprueba el código de Backend.
* **Rotación Técnica de Roles:** Los roles técnicos y de liderazgo por capa rotan a partir del Sprint 2 para equilibrar el dominio integral del stack (consultar la rotación específica en el documento operativo de cada sprint).
* Una vez fusionado el PR, la rama remota debe eliminarse inmediatamente de GitHub y el ticket pasa a **Done**.

---

## 3. Catálogo Maestro de Épicas del Sistema

| ID Jira | Código | Nombre de la Épica | Alcance Funcional | Sprint | Rol Líder |
| :--- | :---: | :--- | :--- | :---: | :--- |
| **ES-4** | `1.0` | **Configuración Base, Persistencia y Seguridad** | Arquitectura inicial, bases de datos (PostgreSQL/Redis), autenticación JWT, RBAC, gestión de sesiones y usuarios. | 1 | Backend / Web Front |
| **ES-5** | `2.0` | **Gestión de Catálogos Operativos** | Catálogos maestros: zonas logísticas, flota vehicular, SLAs de entrega, estados y motivos de rechazo. | 1 | Backend / Web Front |
| **ES-6** | `3.0` | **Aplicación Móvil de Repartidor y Telemetría** | App móvil nativa (Expo/React Native), login, gestión de rutas, lectura QR/firma digital, persistencia SQLite y GPS. | 1 y 2 | Mobile Lead / Architecture |
| **ES-7** | `4.0` | **Módulo de Seguimiento y Portal de Cliente** | Portal público web/móvil para rastreo de guías en tiempo real, consulta de estado y notificaciones a destinatarios. | 3 | Frontend Web / Backend |
| **ES-8** | `5.0` | **Panel Administrativo de Coordinación y Despacho** | Módulo web del Coordinador: asignación de órdenes a repartidores, balanceo de rutas y reprogramación operativa. | 2 | Frontend Web / Backend |
| **ES-9** | `6.0` | **Panel de Supervisión y Monitoreo en Tiempo Real** | Dashboard de torre de control: mapa interactivo con websockets, monitoreo de flotas, alertas de desvío y métricas. | 3 | Fullstack / Realtime |
| **ES-11** | `8.0` | **Interoperabilidad con el ERP y Sistemas Externos** | Ingesta de pedidos por REST, confirmación síncrona/asíncrona con Almacén, gestión con Compras y contratos OpenAPI. | 2 | Backend Lead |
| **ES-10** | `7.0` | **Auditoría, Trazabilidad y Logs** | Pista de auditoría inmutable, logs centralizados de seguridad, métricas de cumplimiento y reportes forenses. | 4 | Architecture / Backend |

---

## 4. Fichas Descriptivas y Dependencias por Épica

### ES-4 — Épica 1.0: Configuración Base, Persistencia y Seguridad
* **Objetivo:** Establecer la infraestructura base de contenedores, diseño de base de datos relacional y servicios de autenticación robustos para todo el ecosistema.
* **Historias Contenidas:**
  * `ES-12`: Setup BD/Docker
  * `ES-13`: JWT/Login
  * `ES-14`: Roles RBAC
  * `ES-15`: Recuperación de clave
  * `ES-16`: Sesión y timeouts
  * `ES-17`: Políticas password
  * `ES-18`: Gestión usuarios internos
  * `ES-19`: Asignación/modificación de roles y permisos
  * `ES-20`: Listado de usuarios con filtros y estados
* **Dependencias:**
  * **Predecesoras:** Ninguna (Cimiento técnico del proyecto).
  * **Sucesoras directas:** Habilita el inicio de las Épicas 2.0, 3.0 y 5.0.
* **Criterio de Entrega (DoD):** Contenedores Docker levantados, esquema TypeORM migrado, autenticación validada con Access/Refresh Tokens y portal web con layout base protegido.
* **Desglose Operativo:** Ver detalle en [`SPRINT_1.md`](./SPRINT_1.md).

---

### ES-5 — Épica 2.0: Gestión de Catálogos Operativos
* **Objetivo:** Centralizar y parametrizar las variables de negocio necesarias para el cálculo de tiempos, distribución geográfica y gestión vehicular.
* **Historias Contenidas:**
  * `ES-21`: Zonas logísticas y tiempos base
  * `ES-22`: Flota y tipos de vehículos
  * `ES-23`: Políticas de SLA
  * `ES-24`: Tipificación de incidencias
  * `ES-25`: Motivos de cancelación
  * `ES-26`: Configuración de sucursales
* **Dependencias:**
  * **Predecesoras:** `ES-12` y `ES-14` (Persistencia relacional y control de roles).
  * **Sucesoras directas:** Bloquea la asignación en la Épica 5.0 y la validación en la Épica 3.0.
* **Criterio de Entrega (DoD):** CRUDs administrativos completos en web (React Hook Form + Tailwind) con endpoints validados en NestJS mediante Swagger.
* **Desglose Operativo:** Ver detalle en [`SPRINT_1.md`](./SPRINT_1.md).

---

### ES-6 — Épica 3.0: Aplicación Móvil de Repartidor y Telemetría
* **Objetivo:** Proveer al personal en campo una herramienta móvil con capacidades offline para gestionar entregas, capturar evidencias y emitir telemetría GPS.
* **Historias Contenidas:**
  * `ES-27`: 3.1 Inicio de sesión móvil para repartidor [RF-U01] (✅ Finalizada en Sprint 1)
  * `ES-28`: 3.2 Consulta de listado consolidado de pedidos asignados de la jornada [RF-U02]
  * `ES-29`: 3.3 Visualización exhaustiva del pedido con navegación y teléfono [RF-U03]
  * `ES-30`: 3.4 Actualización de estado operativo de cada despacho con soporte offline [RF-U04]
  * `ES-31`: 3.5 Registro de evidencia digital (firma en pantalla, foto o código OTP) [RF-U05]
  * `ES-32`: 3.6 Reporte de incidencias operativas tipificadas con evidencia [RF-U06]
  * `ES-33`: 3.7 Consulta de historial de entregas concluidas [RF-U07]
  * `ES-34`: 3.8 Cierre formal de jornada y reconciliación en app móvil [RF-U08] (✅ Finalizada en Sprint 1)
  * `ES-35`: 3.9 Panel de métricas de desempeño y calificación de servicio [RF-U09]
  * `ES-36`: 3.10 Notificaciones push nativas ante asignación de pedidos [RF-U10]
  * `ES-77`: 3.11 Servicio en segundo plano de telemetría GPS [RF-U11] (✅ Finalizada en Sprint 1)
  * `ES-78`: 3.12 Navegación guiada externa en Google Maps o Waze [RF-U12]
  * `ES-79`: 3.13 Motor y persistencia local SQLite para eventos offline [RF-U13] (✅ Finalizada en Sprint 1)
  * `ES-80`: 3.14 Gestión de órdenes de recojo para logística inversa [RF-U23]
* **Dependencias:**
  * **Predecesoras:** `ES-13` (JWT para sesión móvil) y `ES-79` (Esquema local SQLite embebido).
  * **Sucesoras directas:** Alimenta los eventos de la Épica 4.0 y la telemetría en vivo de la Épica 6.0.
* **Criterio de Entrega (DoD):** App Expo compilando en Android/iOS, sincronización bidireccional automática mediante cola FIFO y captura de firma/GPS funcional.
* **Desglose Operativo:** Cimientos en [`SPRINT_1.md`](./SPRINT_1.md), continuación en [`SPRINT_2.md`](./SPRINT_2.md).

---

### ES-7 — Épica 4.0: Módulo de Seguimiento y Portal de Cliente
* **Objetivo:** Brindar visibilidad externa a los remitentes y destinatarios finales sobre el estado de sus envíos de forma pública y segura.
* **Historias Contenidas:**
  * `ES-37`: Consulta pública por tracking ID
  * `ES-38`: Línea de tiempo de estados
  * `ES-39`: Estimación de entrega ETA
  * `ES-40`: Notificaciones por correo/SMS
  * `ES-41`: Descarga de comprobante de recepción
  * `ES-42`: Calificación del servicio
  * `ES-43`: Redirección o cambio de fecha
  * `ES-44`: Portal autoservicio B2B
  * `ES-45`: Reporte de reclamos
* **Dependencias:**
  * **Predecesoras:** Épica 2.0 (Catálogos y SLAs) y eventos generados por la Épica 3.0.
  * **Sucesoras directas:** Retroalimenta las métricas de servicio de la Épica 7.0.
* **Criterio de Entrega (DoD):** Interfaz pública adaptativa (Mobile-first) con verificación de captcha, consulta indexada de alta velocidad y visor de comprobante digital.
* **Desglose Operativo:** Ver detalle en [`SPRINT_3.md`](./SPRINT_3.md).

---

### ES-8 — Épica 5.0: Panel Administrativo de Coordinación y Despacho
* **Objetivo:** Dotar a los coordinadores logísticos de herramientas web para consolidar pedidos, armar paquetes de despacho y balancear la carga de trabajo.
* **Historias Contenidas:**
  * `ES-48`: 5.1 Bandeja central de pedidos consolidados pendientes de despacho [RF-A40]
  * `ES-49`: 5.2 Creación automática de despachos desde Almacén [RF-A02]
  * `ES-50`: 5.3 Agrupación por zonas y capacidad vehicular [RF-A03]
  * `ES-51`: 5.4 Asignación operativa de repartidor y vehículo en lote [RF-A04]
  * `ES-52`: 5.5 Reprogramación de entregas fallidas o rechazadas [RF-A05]
  * `ES-53`: 5.6 Tablero Kanban de despachos en tiempo real [RF-A06]
  * `ES-54`: 5.7 Reportes de cumplimiento de entrega y SLA [RF-A07]
  * `ES-55`: 5.8 Consulta avanzada de historial de despachos [RF-A08]
  * `ES-56`: 5.9 Ficha unificada 360° del despacho [RF-A09]
  * `ES-57`: 5.10 Configuración de parámetros operativos [RF-A10]
  * `ES-58`: 5.11 Prioridad y filtros en el panel [RF-A11]
  * `ES-81`: 5.12 Órdenes de recojo domiciliario (logística inversa) [RF-A39]
* **Dependencias:**
  * **Predecesoras:** Épica 1.0 (Usuarios/Roles) y Épica 2.0 (Zonas, Flotas, SLAs).
  * **Sucesoras directas:** Despacha las rutas hacia la Épica 3.0 (App Móvil).
* **Criterio de Entrega (DoD):** Tablero kanban/tabla interactiva de despachos con drag-and-drop o selector rápido y generación de manifiestos en PDF.
* **Desglose Operativo:** Ver detalle en [`SPRINT_2.md`](./SPRINT_2.md).

---

### ES-9 — Épica 6.0: Panel de Supervisión y Monitoreo en Tiempo Real
* **Objetivo:** Monitorear en una torre de control centralizada el progreso de las rutas, el tráfico y las desviaciones operativas en vivo.
* **Historias Contenidas:**
  * `ES-59`: 6.1 Acceso e inicio de sesión para el Supervisor de Flota [RF-A12]
  * `ES-60`: 6.2 Integración de mapas web para ubicación de vehículos activos [RF-A13]
  * `ES-61`: 6.3 Panel de indicadores (KPIs) de desempeño de flota [RF-A14]
  * `ES-62`: 6.4 Módulo de gestión de estados de vehículos [RF-A15]
  * `ES-63`: 6.5 Reasignación rápida de despachos ante contingencias [RF-A16]
  * `ES-64`: 6.6 Bitácora y seguimiento a mantenimientos de flota [RF-A17]
  * `ES-65`: 6.7 Generador de reportes de rendimiento de flota por periodo [RF-A18]
  * `ES-66`: 6.8 Generación de alertas automáticas ante incidencias críticas [RF-A19]
  * `ES-67`: 6.9 Configuración de umbrales para disparo de alertas de rendimiento [RF-A20]
* **Dependencias:**
  * **Predecesoras:** `ES-77` (Telemetría móvil) y Épica 5.0 (Despachos activos).
  * **Sucesoras directas:** Almacenamiento histórico hacia la Épica 7.0.
* **Criterio de Entrega (DoD):** Servidor Socket.io / Redis Pub-Sub con refresco menor a 2 segundos en mapa interactivo (Leaflet/Mapbox).
* **Desglose Operativo:** Ver detalle en [`SPRINT_3.md`](./SPRINT_3.md).

---

### ES-11 — Épica 8.0: Interoperabilidad con el ERP y Sistemas Externos
* **Objetivo:** Automatizar el intercambio de información con sistemas legados (Almacén e Inventarios, Compras y Proveedores) mediante interfaces REST y contratos OpenAPI.
* **Historias Contenidas:**
  * `ES-72`: 8.1 Endpoint REST seguro para recibir pedidos preparados desde Almacén [ERP-01]
  * `ES-73`: 8.2 Notificación de salida física hacia Almacén e Inventarios [ERP-02]
  * `ES-74`: 8.3 Interfaces REST para recibir solicitudes de transporte y logística inversa desde Compras [ERP-03]
  * `ES-75`: 8.4 Programación y gestión de recojo en proveedores hacia almacenes centrales [ERP-04]
  * `ES-76`: 8.5 Configuración y publicación de documentación OpenAPI/Swagger para API Gateway [ERP-05]
* **Dependencias:**
  * **Predecesoras:** Épica 1.0 (Infraestructura y seguridad base).
  * **Sucesoras directas:** Alimenta automáticamente las órdenes de la Épica 5.0 y confirma custodia física.
* **Criterio de Entrega (DoD):** Endpoints REST M2M documentados con OpenAPI, validados mediante pruebas de integración automáticas y tolerancia a fallos por cola de mensajes.
* **Desglose Operativo:** Ver detalle en [`SPRINT_2.md`](./SPRINT_2.md).

---

### ES-10 — Épica 7.0: Auditoría, Trazabilidad y Logs
* **Objetivo:** Garantizar la seguridad forense, el no repudio de las transacciones y la generación de métricas históricas de rendimiento.
* **Historias Contenidas:**
  * `ES-68`: 7.1 Interceptor en NestJS para registro automático de auditoría [RF-A35]
  * `ES-69`: 7.2 Almacenamiento inmutable de eventos de negocio por pedido y despacho [RF-A36]
  * `ES-70`: 7.3 Interfaz administrativa para consulta y filtrado granular de logs [RF-A37]
  * `ES-71`: 7.4 Exportación de registros de auditoría a formatos CSV y PDF [RF-A38]
* **Dependencias:**
  * **Predecesoras:** Concurrencia de datos de todas las épicas transaccionales (1.0 a 6.0 y 8.0).
  * **Sucesoras directas:** Entrega final y cierre de proyecto para defensa.
* **Criterio de Entrega (DoD):** Tablas de auditoría particionadas o centralizadas, sanitización de datos sensibles en logs y exportación a formatos estándar (Excel/PDF).
* **Desglose Operativo:** Ver detalle en [`SPRINT_4.md`](./SPRINT_4.md).
