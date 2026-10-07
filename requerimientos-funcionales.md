# Catálogo de Requerimientos Funcionales (63 RF)
**Microservicio de Gestión de Entregas y Despachos (Grupo H)**

El sistema cuenta con un total de **63 requerimientos funcionales**, organizados en dos grandes módulos:
* **Módulo Usuario (23 RF):** Subdividido en *Repartidor / Transportista* (App Móvil) y *Cliente Final* (Portal Web).
* **Panel Administrativo (40 RF):** Subdividido en *Coordinador*, *Supervisor*, *Autenticación y Seguridad*, *Gestión de Usuarios*, *Catálogos* y *Auditoría*.

---

## 1. Módulo Usuario (23 RF)

### 1.1 Repartidor / Transportista (App Móvil — 14 RF)

| Código | Requerimiento Funcional | Plataforma | Descripción detallada |
|---|---|---|---|
| **RF-U01** | Iniciar sesión | App móvil | Iniciar sesión en el sistema con sus credenciales antes de acceder a sus pedidos del día. |
| **RF-U02** | Consultar pedidos del día | App móvil | Consultar la lista de pedidos asignados para la jornada actual. |
| **RF-U03** | Ver detalle de pedido | App móvil | Ver el detalle informativo de cada pedido (dirección, productos, etiqueta informativa de estado de pago —ej. Prepagado—). |
| **RF-U04** | Actualizar estado de entrega | App móvil | Actualizar el estado de cada entrega (`en camino`, `entregado`, `no entregado`, `devuelto`). |
| **RF-U05** | Registrar evidencia digital | App móvil | Registrar evidencia digital de la entrega (fotografía de respaldo, firma electrónica o código OTP). |
| **RF-U06** | Registrar incidencias | App móvil | Registrar incidencias durante la entrega (dirección incorrecta, cliente ausente, producto dañado). |
| **RF-U07** | Historial de entregas | App móvil | Consultar el historial de entregas realizadas en días anteriores. |
| **RF-U08** | Cerrar sesión | App móvil | Cerrar sesión al finalizar su turno. |
| **RF-U09** | Resumen de desempeño | App móvil | Consultar su propio resumen de desempeño (entregas completadas, calificación promedio). |
| **RF-U10** | Notificaciones push de asignación | App móvil | Recibir notificaciones push nativas al asignársele un nuevo pedido durante su turno. |
| **RF-U11** | Transmisión GPS en segundo plano | App móvil | Transmitir la ubicación GPS periódicamente y en segundo plano mientras exista un despacho activo en curso. |
| **RF-U12** | Navegación externa | App móvil | Iniciar la navegación hacia la dirección de entrega abriendo directamente aplicaciones externas de mapas (Google Maps o Waze). |
| **RF-U13** | Modo offline y sincronización | App móvil | Almacenar localmente los cambios de estado e incidencias ante pérdida de conectividad, sincronizándolos automáticamente al recuperar conexión. |
| **RF-U23** | Recojo de devoluciones en ruta | App móvil | Visualizar órdenes de recojo de devoluciones asignadas en su ruta, verificar el estado físico del producto y capturar fotografía/firma como evidencia del retiro. |

### 1.2 Cliente Final (Portal Web Público — 9 RF)

| Código | Requerimiento Funcional | Plataforma | Descripción detallada |
|---|---|---|---|
| **RF-U14** | Acceso con token de seguimiento | Portal web | Acceder a la vista de seguimiento del pedido mediante un enlace seguro único (con token) enviado por correo electrónico o ingresando el código de seguimiento. |
| **RF-U15** | Consultar estado y ubicación | Portal web | Consultar el estado y la última ubicación registrada de su pedido en tiempo real. |
| **RF-U16** | Notificaciones por correo | Portal web | Recibir notificaciones automáticas por correo electrónico en cada cambio de estado del despacho. |
| **RF-U17** | Tiempo estimado de llegada (ETA) | Portal web | Consultar el tiempo estimado de llegada (ETA) calculado para su pedido. |
| **RF-U18** | Datos del repartidor | Portal web | Consultar los datos básicos del repartidor asignado (nombre, vehículo). |
| **RF-U19** | Calificar servicio | Portal web | Calificar el servicio de entrega recibido mediante puntuación. |
| **RF-U20** | Comentarios de entrega | Portal web | Dejar comentarios u observaciones sobre la experiencia de entrega. |
| **RF-U21** | Modificar entrega en ventana | Portal web | Solicitar, dentro de la ventana permitida, un cambio de dirección u horario de entrega. |
| **RF-U22** | Reportar reclamos | Portal web | Reportar un reclamo formal sobre una entrega ya finalizada. |

---

## 2. Panel Administrativo (40 RF)

### 2.1 Coordinador de Despachos / Logística (Portal Web — 13 RF)

| Código | Requerimiento Funcional | Plataforma | Descripción detallada |
|---|---|---|---|
| **RF-A01** | Iniciar sesión como Coordinador | Portal web | Iniciar sesión en el sistema con su rol de coordinador. |
| **RF-A02** | Registro automático de despacho | Portal web | Registrar automáticamente una orden de despacho a partir de un pedido confirmado y listo para despacho, recibido desde *Gestión de Inventarios y Almacén*. |
| **RF-A03** | Agrupación por zona y capacidad | Portal web | Agrupar los pedidos por zona de reparto y validar la capacidad disponible del vehículo antes de asignarlos. |
| **RF-A04** | Asignación de repartidor y vehículo | Portal web | Asignar repartidor y vehículo a cada pedido o lote de pedidos. |
| **RF-A05** | Reprogramación de entregas | Portal web | Reprogramar entregas fallidas o rechazadas con nueva fecha y motivo. |
| **RF-A06** | Tablero en tiempo real | Portal web | Visualizar en un tablero interactivo el estado en tiempo real de todos los despachos activos. |
| **RF-A07** | Reportes de cumplimiento y SLA | Portal web | Generar reportes de cumplimiento de entrega (tiempos, SLA, % de entregas a tiempo). |
| **RF-A08** | Historial de despachos | Portal web | Consultar el historial de despachos por repartidor, vehículo o periodo de tiempo. |
| **RF-A09** | Vista unificada de despacho | Portal web | Consultar el detalle completo de un despacho específico en una vista unificada (trazabilidad, eventos, evidencias, incidencias). |
| **RF-A10** | Configuración de parámetros de servicio | Portal web | Configurar parámetros generales del servicio (ventana horaria de entregas, tiempo máximo de espera). |
| **RF-A11** | Prioridad de despachos | Portal web | Marcar y filtrar despachos por nivel de prioridad (`urgente` / `normal`). |
| **RF-A39** | Órdenes de recojo en domicilio | Portal web | Registrar y programar órdenes de recojo en domicilio para devoluciones o garantías aprobadas (logística inversa desde el cliente hacia el almacén). |
| **RF-A40** | Bandeja central de pedidos pendientes | Portal web | Visualizar en una bandeja central estructurada los pedidos consolidados pendientes de asignación y despacho, con filtros rápidos por zona, turno y búsqueda reactiva. |

### 2.2 Supervisor de Flota (Portal Web — 9 RF)

| Código | Requerimiento Funcional | Plataforma | Descripción detallada |
|---|---|---|---|
| **RF-A12** | Iniciar sesión como Supervisor | Portal web | Iniciar sesión en el sistema con su rol de supervisor. |
| **RF-A13** | Monitoreo en mapa de flota | Portal web | Monitorear en un mapa en tiempo real la última ubicación registrada de los vehículos activos. |
| **RF-A14** | Indicadores de desempeño de flota | Portal web | Consultar indicadores de desempeño de la flota (entregas a tiempo, incidencias por repartidor). |
| **RF-A15** | Disponibilidad de vehículos | Portal web | Gestionar la disponibilidad y estado de los vehículos (`activo`, `en mantenimiento`, `fuera de servicio`). |
| **RF-A16** | Reasignación ante contingencias | Portal web | Reasignar despachos entre repartidores ante contingencias (averías, emergencias). |
| **RF-A17** | Registro de mantenimientos | Portal web | Registrar y dar seguimiento al historial de mantenimiento preventivo y correctivo de cada vehículo. |
| **RF-A18** | Reportes periódicos de flota | Portal web | Generar reportes de desempeño e incidentes de la flota por periodo. |
| **RF-A19** | Alerta de incidencias críticas | Portal web | Recibir una alerta automática ante el registro de una incidencia crítica en ruta. |
| **RF-A20** | Configuración de umbrales SLA | Portal web | Configurar umbrales de alerta de desempeño (ej. SLA mínimo aceptable). |

### 2.3 Administración — Autenticación y Seguridad (5 RF)

| Código | Requerimiento Funcional | Plataforma | Descripción detallada |
|---|---|---|---|
| **RF-A21** | Autenticación y bloqueo por reintentos | Ambos | Autenticar usuarios con usuario/contraseña, con bloqueo temporal tras varios intentos fallidos. |
| **RF-A22** | Control de acceso por roles | Ambos | Restringir el acceso a funcionalidades según el rol (`Coordinador`, `Supervisor`, `Repartidor`). |
| **RF-A23** | Recuperación y cambio de contraseña | Ambos | Permitir el cambio y la recuperación de contraseña de forma segura. |
| **RF-A24** | Cierre de sesión por inactividad | Ambos | Cerrar sesión tras un periodo de inactividad (excluyendo la app móvil en ruta, con refresh tokens seguros). |
| **RF-A25** | Políticas de contraseñas | Portal web | Configurar la política de contraseñas (longitud mínima, complejidad, vencimiento periódico). |

### 2.4 Administración — Gestión de Usuarios (3 RF)

| Código | Requerimiento Funcional | Plataforma | Descripción detallada |
|---|---|---|---|
| **RF-A26** | ABM de usuarios internos | Portal web | Registrar, editar y desactivar usuarios internos (coordinadores, supervisores, repartidores). |
| **RF-A27** | Asignación de roles y permisos | Portal web | Asignar y modificar el rol y los permisos de cada usuario. |
| **RF-A28** | Listado de usuarios | Portal web | Consultar el listado de usuarios activos e inactivos del microservicio. |

### 2.5 Administración — Gestión de Catálogos (6 RF)

| Código | Requerimiento Funcional | Plataforma | Descripción detallada |
|---|---|---|---|
| **RF-A29** | Catálogo de zonas de reparto | Portal web | Administrar el catálogo de zonas de reparto y sus tiempos estimados de entrega. |
| **RF-A30** | Catálogo de vehículos | Portal web | Administrar el catálogo de vehículos de la flota (tipo, capacidad en kg, placa). |
| **RF-A31** | Catálogo de niveles de servicio | Portal web | Administrar el catálogo de niveles de servicio (`estándar`, `express`) y tiempos objetivo. |
| **RF-A32** | Catálogo de motivos de incidencia | Portal web | Administrar el catálogo de motivos de incidencia (`cliente ausente`, `dirección incorrecta`, `producto dañado`, etc.). |
| **RF-A33** | Catálogo de motivos de reprogramación | Portal web | Administrar el catálogo de motivos de reprogramación o reasignación de despachos. |
| **RF-A34** | Catálogo de tipos de incidente de vehículo | Portal web | Administrar el catálogo de tipos de incidente vehicular para control de mantenimiento. |

### 2.6 Administración — Auditoría y Logs (4 RF)

| Código | Requerimiento Funcional | Plataforma | Descripción detallada |
|---|---|---|---|
| **RF-A35** | Logs de auditoría administrativa | Ambos | Registrar log de auditoría de cada inicio de sesión y acción relevante realizada en el sistema. |
| **RF-A36** | Logs de eventos de despacho | Ambos | Registrar log de eventos por cada pedido y despacho (creación, asignación, cambios de estado). |
| **RF-A37** | Filtrado de auditoría | Portal web | Consultar y filtrar los logs de auditoría por usuario, rango de fechas o tipo de evento. |
| **RF-A38** | Exportación de auditoría | Portal web | Exportar los logs de auditoría a archivos descargables (CSV / PDF). |
