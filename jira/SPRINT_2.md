# SPRINT_2.md — Desglose Operativo del Sprint 2: Historias de Usuario y Subtareas Técnicas

> **Proyecto:** Microservicio de Gestión de Entregas y Despachos (Grupo H / ERP Corporativo)  
> **Sprint:** 2  
> **Fuente de Trazabilidad:** Jira Cloud / GitHub  
> **Documento Padre:** [`PROJECT_BACKLOG.md`](./PROJECT_BACKLOG.md)  
> **Rotación de Roles en Sprint 2:**
> * **Backend (NestJS / REST / Integraciones ERP):** Sergio & Joan
> * **App Mobile (Expo / React Native / SQLite):** Jaime & Joan
> * **Frontend Web (React 19 / Vite / Tailwind):** Pardo & Sergio
> * **Base de Datos (PostgreSQL / Migraciones / Modelado):** Sergio & Joan

---

## 1. Alcance y Épicas Cubiertas en el Sprint 2

En este Sprint 2 se implementan las operaciones nucleares de despacho, la interacción móvil en ruta y la integración de pedidos con el ERP corporativo:
* **Épica 3.0 (`ES-6`):** Aplicación Móvil de Repartidor y Telemetría
* **Épica 5.0 (`ES-8`):** Panel Administrativo de Coordinación y Despacho
* **Épica 8.0 (`ES-11`):** Interoperabilidad con el ERP y Sistemas Externos (Fase 1: Ingesta y Contratos)

---

## 2. Historias de Usuario y Subtareas Técnicas

---

### ÉPICA 3.0 — Aplicación Móvil de Repartidor y Telemetría (`ES-6`)

#### `ES-28`: 3.2 Como Repartidor en ruta, requiero consultar el listado consolidado de pedidos asignados para mi jornada actual [RF-U02]
* **Épica:** 3.0 Aplicación Móvil de Repartidor y Telemetría (`ES-6`)
* **Asignado (PE):** Jaime | **Story Points (Spe):** 5 | **Estado:** Por hacer (To Do)
* **Fechas:** Inicio (FI): 05 de Oct | Vencimiento (FV): 07 de Oct
* **Dependencias:** Bloquea a (B): `ES-29`, `ES-30`, `ES-36` | Bloqueado por (BP): `ES-27 (✅ Finalizada en Sprint 1)`

> **Como** Repartidor en ruta  
> **Quiero** consultar en la aplicación móvil el listado consolidado de pedidos asignados para mi jornada actual en orden estricto de paradas optimizadas y con soporte offline  
> **Para** tener visibilidad clara de mi carga de trabajo, respetar las franjas horarias comprometidas con los clientes y ejecutar el recorrido sin desvíos no autorizados.

* **Criterios de Aceptación (GWT):**
  * **Escenario 1 (Visualización de la hoja de ruta con orden estricto y franjas comprometidas - RF-U02):**  
    **Dado** que el repartidor se encuentra autenticado al iniciar su turno,  
    **cuando** accede a la pestaña "Mi Jornada",  
    **entonces** la aplicación consulta al backend las asignaciones de la jornada del repartidor y despliega la lista de paradas ordenadas cronológica y geográficamente (`sequence_order`: 1, 2, 3…) con: número de parada, código de guía, cliente, dirección resumida, franja horaria comprometida (`14:00 - 16:00`), estado operativo (`assigned`, `out_for_delivery`, `in_transit`) y política de evidencia requerida.
  * **Escenario 2 (Inmutabilidad del orden de paradas por el conductor):**  
    **Dado** un lote de pedidos en estado `out_for_delivery`,  
    **cuando** el repartidor intenta iniciar el traslado hacia la parada #4 saltándose la parada #1 pendiente,  
    **entonces** la aplicación bloquea la acción indicando que debe atender las paradas en la secuencia optimizada calculada por Despacho para garantizar los SLAs comprometidos, permitiendo únicamente reportar una incidencia formal (`ES-32`) si la parada actual está bloqueada.
  * **Escenario 3 (Disponibilidad sin conexión celular mediante caché SQLite):**  
    **Dado** que el repartidor ya sincronizó su jornada y transita por una zona sin cobertura celular,  
    **cuando** navega por la lista de despachos o abre el detalle de una parada,  
    **entonces** la app lee los datos íntegros desde la tabla local `local_dispatches` en SQLite sin desplegar pantallas de error ni bloquear la interfaz.
  * **Escenario 4 (Actualización manual pull-to-refresh y detección de cambios de ruta):**  
    **Dado** que el coordinador web realizó una modificación justificada de la ruta o canceló una orden,  
    **cuando** el transportista ejecuta el gesto de pull-to-refresh,  
    **entonces** la app sincroniza con el backend, actualiza la base SQLite local, renumera las paradas y muestra una alerta visual informando la actualización de la hoja de ruta.
  * **Escenario 5 (Descarga masiva inicial al inicio de turno):**  
    **Dado** que el repartidor inicia su turno con conectividad activa,  
    **cuando** se autentica por primera vez en el día,  
    **entonces** el sistema descarga en segundo plano el paquete completo de órdenes (ítems, coordenadas GPS, contactos de clientes, ventanas pactadas y políticas de evidencia), persistiendo todo en SQLite local para garantizar operatividad 100% offline ante caídas de red durante el trayecto.

| ID Jira | Subtarea | Descripción Técnica y Tareas a Realizar | Asignado (PE) | Estimación | Estado |
| :--- | :--- | :--- | :---: | :---: | :---: |
| `ES-130` | `ST-28.1` | **Mobile UI: Pantalla «Mi Jornada» con lista virtualizada (FlashList), orden de paradas y pull-to-refresh:**<br>• Construir la pantalla con `@shopify/flash-list`, alimentada por el repositorio de `ST-28.2` (SQLite primero) y ordenada siempre por `sequence_order` ascendente; el repartidor no puede reordenar.<br>• Tarjeta de parada: badge numérico (`Parada #1`, `#2`…), código de guía, nombre del cliente, dirección resumida, franja comprometida (`hh:mm - hh:mm`) y badge de estado (`assigned`, `out_for_delivery`, `in_transit`) con un color distinto por estado.<br>• Parada activa: destacar visualmente la primera parada no cerrada y atenuar las siguientes; si el repartidor intenta iniciar una parada posterior, mostrar un mensaje que explique que debe atender las paradas en el orden calculado por Despacho y ofrecer el acceso a «Reportar incidencia» (`ES-32`).<br>• `RefreshControl` para pull-to-refresh que dispare la sincronización de `ST-28.2`; si la ruta cambió, mostrar la alerta visual «Tu hoja de ruta fue actualizada».<br>• Estados de pantalla: skeleton con la misma estructura de la tarjeta mientras carga, estado vacío ilustrado cuando no hay asignaciones y banner discreto «Sin conexión, mostrando datos guardados» cuando solo se lee de SQLite.<br>• Criterio de terminado: pruebas de componente del orden de paradas, de la parada activa, de la lista vacía y del modo sin conexión; lista de 50 paradas sin caídas de fluidez perceptibles en un dispositivo de gama media. | Jaime | 6h | Por hacer |
| `ES-131` | `ST-28.2` | **Mobile Data: Servicio de asignaciones, repositorio SQLite (`local_dispatches`) y sincronización:**<br>• Crear el servicio que consulta al backend las asignaciones de la jornada del repartidor (GET) inyectando el Bearer token en Axios; si responde 401, delegar en el refresco de sesión existente y reintentar una sola vez.<br>• Definir en SQLite la tabla `local_dispatches` con: `dispatch_id` (único), `sequence_order`, `status`, código de guía, datos del cliente y del destino (nombre, teléfono, dirección, referencias, coordenadas), franja comprometida, política de evidencia, etiqueta de pago, bultos (JSON), `is_synced` y `updated_at`. Índices en `sequence_order`, `status` y `dispatch_id`.<br>• Repositorio híbrido: leer siempre de SQLite y refrescar desde el servidor al abrir la pantalla, al hacer pull-to-refresh y al volver la conectividad. Si la red falla o tarda más de 10 s (valor propuesto), devolver los datos locales sin mostrar un error bloqueante.<br>• Reconciliación: reemplazar los despachos por ID, renumerar `sequence_order` según el servidor, conservar los cambios locales con `is_synced = 0` y marcar como removidos los despachos que el servidor ya no envía (por ejemplo, cancelados).<br>• Descarga masiva inicial: en el primer inicio de sesión del día, descargar en segundo plano el paquete completo de la jornada (ítems, coordenadas, contactos, ventanas y políticas de evidencia).<br>• Purga: eliminar los despachos de jornadas anteriores ya concluidos y sincronizados, conservando las últimas 48 horas de despachos cerrados para el historial de `ST-33.1`.<br>• Criterio de terminado: pruebas del repositorio (lectura sin red, reconciliación con cambios del servidor, conservación de cambios locales y purga). | Joan | 6h | Por hacer |
| `ES-132` | `ST-28.3` | **Mobile UI: Componentes de tarjeta de parada con badges de SLA, tipo de servicio y política de evidencia:**<br>• Badge de tipo de servicio según el tipo de despacho y el nivel de servicio: Express, Estándar o Recojo RMA (`warehouse_return_pickup`), con color e ícono distintos.<br>• Indicador de urgencia: calcular el tiempo restante hasta el límite superior de la franja comprometida y mostrar una cuenta regresiva cuando falten menos de 30 minutos (umbral propuesto, definido en una constante), pasando a rojo cuando la franja ya venció.<br>• Badge de la política de evidencia requerida: Firma (`hand_delivery_standard`), Foto sin contacto (`contactless_delivery`) o Código OTP (`high_value_control`).<br>• Navegación tipada con `expo-router` hacia `/pedido/[id]` pasando el ID del despacho; los parámetros mal tipados deben fallar en compilación.<br>• Criterio de terminado: pruebas de componente de cada badge, del cambio de color de la cuenta regresiva y de la navegación con el ID correcto. | Jaime | 4h | Por hacer |
| `ES-204` | `ST-28.4` | **Backend: Consulta de las asignaciones de la jornada del repartidor (GET):**<br>• Devolver los despachos de la ruta (`route_batches`) del repartidor autenticado cuya `shift_date` sea la del día, ordenados por `sequence_order` ascendente. El repartidor se toma siempre del token (rol Repartidor), nunca de un parámetro.<br>• Cada elemento incluye: ID del despacho, código de guía, `sequence_order`, estado, cliente (nombre y teléfono), dirección con referencias y coordenadas, franja comprometida, nivel de servicio, política de evidencia (`evidence_policy`), etiqueta de pago y bultos (cuando existan, `ST-49.2`).<br>• Un repartidor sin ruta para el día recibe una lista vacía (200), no un error; nunca se devuelven despachos de otras rutas ni de otros días.<br>• Incluir en la respuesta la fecha de la última modificación de la hoja de ruta, para que la app detecte cambios en el pull-to-refresh (`ES-28`, Escenario 4).<br>• Documentar con Swagger (`@ApiOperation`, 200, 401, 403) siguiendo el patrón de módulos del repositorio.<br>• Criterio de terminado: pruebas con ruta asignada, sin ruta, con otro repartidor y con despachos de otro día; `npm run build`, `npm run lint` y `npm run test` sin errores. | Joan | 4h | Por hacer |

---

#### `ES-29`: 3.3 Como Repartidor en ruta, requiero visualizar el detalle exhaustivo del pedido con acceso a navegación y teléfono del cliente [RF-U03]
* **Épica:** 3.0 Aplicación Móvil de Repartidor y Telemetría (`ES-6`)
* **Asignado (PE):** Jaime | **Story Points (Spe):** 5 | **Estado:** Por hacer (To Do)
* **Fechas:** Inicio (FI): 08 de Oct | Vencimiento (FV): 09 de Oct
* **Dependencias:** Bloquea a (B): `ES-78`, `ES-30` | Bloqueado por (BP): `ES-28`

> **Como** Repartidor o transportista en ruta  
> **Quiero** acceder al detalle exhaustivo de una orden con acceso directo a GPS, marcación telefónica, lista de bultos/productos, política de evidencia exigida y condición informativa de pago  
> **Para** verificar la carga antes de descender del vehículo, saber con exactitud qué comprobante solicitar al cliente y contactarlo de forma inmediata sin gestionar cobros.

* **Criterios de Aceptación (GWT):**
  * **Escenario 1 (Visualización estructurada de secciones del pedido - RF-U03):**  
    **Dado** que el repartidor selecciona una orden en su hoja de ruta,  
    **cuando** se abre la pantalla `/pedido/:id`,  
    **entonces** se despliegan en tarjetas ordenadas: Datos del Cliente (nombre, teléfono con botón de llamada directa), Destino de Entrega (dirección completa, referencias de fachada y botón "Navegar con GPS"), Resumen de Carga (número de bultos, peso total, lista de ítems con descripción física y cantidades) y Política de Evidencia requerida (Firma obligatoria, Foto sin contacto autorizada o Código OTP).
  * **Escenario 2 (Etiqueta informativa de pago sin gestión de cobros en la app):**  
    **Dado** un pedido visualizado en el dispositivo móvil,  
    **cuando** se inspecciona la sección de estado financiero,  
    **entonces** la app exhibe exclusivamente la etiqueta informativa «Pagado» (`paid`): en este proyecto todos los pedidos se asumen pagados, y la app no ofrece opciones de registro, cobro ni conciliación de dinero.
  * **Escenario 3 (Enlace directo a navegación satelital externa):**  
    **Dado** que el pedido cuenta con coordenadas geográficas válidas,  
    **cuando** el transportista presiona "Navegar con GPS",  
    **entonces** la aplicación abre la app nativa seleccionada (Google Maps o Waze) precargando las coordenadas de destino en modo conducción.
  * **Escenario 4 (Marcación telefónica rápida con confirmación):**  
    **Dado** que la orden contiene el teléfono verificado del cliente,  
    **cuando** el repartidor presiona el botón de llamada,  
    **entonces** el sistema invoca el marcador nativo del teléfono (`tel:+591...`) tras confirmación previa, permitiendo coordinar la entrega rápidamente.

| ID Jira | Subtarea | Descripción Técnica y Tareas a Realizar | Asignado (PE) | Estimación | Estado |
| :--- | :--- | :--- | :---: | :---: | :---: |
| `ES-133` | `ST-29.1` | **Mobile UI: Pantalla de detalle de despacho (`/pedido/[id]`) modular en React Native con NativeWind:**<br>• Crear la vista `/pedido/[id].tsx`, que lee primero de SQLite (`ST-29.3`), con tarjetas en este orden: encabezado con código de guía y badge de estado; Cliente (nombre y botón de llamada); Destino (dirección completa, referencias de fachada, mapa estático en miniatura y botón «Navegar con GPS»); Resumen de carga (número de bultos, peso total, volumen en m³ y lista de ítems con descripción física y cantidades, sin montos monetarios); Política de evidencia.<br>• Etiqueta informativa «Pagado» (todos los pedidos se asumen pagados), sin controles para registrar cobros.<br>• Banner destacado con la política de prueba de entrega que exigirá la app: firma, foto sin contacto u OTP.<br>• Botón principal contextual: «Iniciar traslado a este domicilio» (si el despacho está `out_for_delivery`, es la parada activa y no hay otro en `in_transit`), «Gestionar entrega» (si está `in_transit`) o deshabilitado con una explicación en los demás casos.<br>• Manejar textos largos y datos ausentes (sin teléfono, sin referencias) con valores alternativos legibles.<br>• Criterio de terminado: pruebas de componente de cada tarjeta, de los tres estados del botón principal y de la etiqueta de pago. | Jaime | 6h | Por hacer |
| `ES-134` | `ST-29.2` | **Mobile Native: Integración de deep linking (Google Maps/Waze) y marcador telefónico (Linking API):**<br>• Utilitario `buildNavigationUrl(app, destino)` con los esquemas: `geo:lat,lng?q=lat,lng(etiqueta)` (Android), `https://www.google.com/maps/dir/?api=1&destination=lat,lng&travelmode=driving` y `waze://?ll=lat,lng&navigate=yes`; codificar la etiqueta con `encodeURIComponent`.<br>• Detectar las apps instaladas con `Linking.canOpenURL` (declarar los esquemas en la configuración de Expo para iOS y Android) y reutilizar el selector y la persistencia de preferencia de `ST-78.1`.<br>• Llamada telefónica: sanear el número (conservar dígitos y `+`; anteponer `+591` si no tiene prefijo internacional), pedir confirmación previa («¿Llamar a [nombre]?») y abrir `tel:`; si el dispositivo no puede llamar (por ejemplo, sin SIM), capturar la excepción y mostrar el número para copiarlo.<br>• Coordenadas nulas o inválidas (latitud fuera de −90 a 90 o longitud fuera de −180 a 180): no abrir mapas con ellas y mostrar una alerta contextual que ofrezca buscar por la dirección textual.<br>• Criterio de terminado: pruebas unitarias de cada URL generada, del saneamiento del teléfono y de las coordenadas inválidas. | Joan | 4h | Por hacer |
| `ES-135` | `ST-29.3` | **Mobile Data: Tipos TypeScript, selectores de datos y validación del detalle del despacho:**<br>• Definir las interfaces `DispatchDetail`, `DispatchItem`, `DispatchPackage`, `EvidenceRequirementType` (`hand_delivery_standard` \| `contactless_delivery` \| `high_value_control`), `PaymentLabel` (`paid`) y `CustomerInfo`, alineadas con la respuesta del backend y con `local_dispatches`.<br>• Crear el selector (React Query + Zustand) que lee primero de SQLite y revalida en segundo plano si hay red, sin parpadeos: mostrar el dato local de inmediato y reemplazarlo si el servidor trae cambios.<br>• Validar la respuesta del servidor con un esquema (por ejemplo Zod) antes de guardarla; si un campo es inválido, registrar el error y conservar el dato local anterior.<br>• Probar la presentación de textos largos, referencias de dirección e instrucciones especiales sin desbordamiento.<br>• Criterio de terminado: pruebas del selector (con y sin red, con respuesta inválida) y `npm run typecheck` sin errores. | Jaime | 4h | Por hacer |

---

#### `ES-30`: 3.4 Como Repartidor en campo, necesito actualizar el estado operativo de cada despacho con soporte fuera de línea [RF-U04]
* **Épica:** 3.0 Aplicación Móvil de Repartidor y Telemetría (`ES-6`)
* **Asignado (PE):** Joan | **Story Points (Spe):** 5 | **Estado:** Por hacer (To Do)
* **Fechas:** Inicio (FI): 12 de Oct | Vencimiento (FV): 15 de Oct
* **Dependencias:** Bloquea a (B): `ES-31`, `ES-32`, `ES-33`, `ES-35`, `ES-61`, `ES-69` | Bloqueado por (BP): `ES-28`, `ES-29`, `ES-79 (✅ Finalizada en Sprint 1)`

> **Como** Repartidor en campo  
> **Quiero** actualizar el ciclo de estado de mi ruta y de cada despacho individual (`out_for_delivery`, `in_transit`, `delivered`, `not_delivered`, `returned`) directamente desde la app móvil con soporte offline y regla de traslado activo único  
> **Para** sincronizar la trazabilidad del envío en tiempo real con la torre de control y asegurar que los clientes reciban las notificaciones precisas de proximidad.

* **Criterios de Aceptación (GWT):**
  * **Escenario 1 (Salida de almacén y transición masiva a 'En Ruta'):**  
    **Dado** un conjunto de pedidos en estado `assigned` cargados en el vehículo al salir del CEDIS,  
    **cuando** el repartidor presiona el botón principal "Iniciar Ruta del Día",  
    **entonces** todos los pedidos de su hoja de ruta transicionan a `out_for_delivery`, se notifica al servidor/cola offline, el backend publica la salida física hacia Almacén (`ES-73`) y emite el evento que consumirá el módulo de notificaciones al cliente (`ES-40`, Sprint 3).
  * **Escenario 2 (Inicio de traslado puntual a parada específica y regla de unicidad de 'En Camino'):**  
    **Dado** que los pedidos están en estado `out_for_delivery` y no hay ningún otro pedido en curso,  
    **cuando** el repartidor selecciona su siguiente parada en secuencia y presiona "Iniciar Traslado a este Domicilio",  
    **entonces** ese pedido específico pasa a estado `in_transit`, se captura la coordenada GPS de partida, el backend emite el evento de proximidad para que el módulo de notificaciones (`ES-40`, Sprint 3) avise al cliente, y la app bloquea el inicio simultáneo de cualquier otro pedido hasta que este sea entregado o reportado con incidencia.
  * **Escenario 3 (Validación de precondición estricta de evidencia para entrega final):**  
    **Dado** un despacho en estado `in_transit`,  
    **cuando** el repartidor intenta transicionar el estado a `delivered`,  
    **entonces** la máquina de estados valida estrictamente que la petición incluya el payload con la evidencia digital exigida (`ES-31`); si falta la evidencia requerida (ej. firma omitida o código OTP ausente), la app rechaza la transición impidiendo el cierre del pedido.
  * **Escenario 4 (Persistencia y encolamiento FIFO offline ante pérdida de señal):**  
    **Dado** un dispositivo sin cobertura celular en el momento de actualizar un estado (`in_transit`, `delivered`, `not_delivered`),  
    **cuando** el repartidor confirma la acción,  
    **entonces** el evento se guarda de forma atómica en la tabla `local_events` de SQLite con `synced = 0`, UUID v7 y timestamp UTC, actualizando la vista local del pedido y mostrando el badge "Pendiente de sincronizar" sin interrumpir la operación del conductor.

| ID Jira | Subtarea | Descripción Técnica y Tareas a Realizar | Asignado (PE) | Estimación | Estado |
| :--- | :--- | :--- | :---: | :---: | :---: |
| `ES-147` | `ST-30.1` | **Mobile UI: Máquina de estados finita del despacho con validación de transiciones legales y unicidad de `in_transit`:**<br>• Implementar la máquina de estados en un módulo puro (sin dependencias de UI) con las transiciones legales: `assigned` → `out_for_delivery` → `in_transit` → (`delivered` \| `not_delivered` \| `incident`) → `returned` (esta última solo cuando Despacho decide devolver el pedido a almacén; no la decide el repartidor). Cualquier otra transición debe lanzar un error tipado `IllegalTransitionError`.<br>• Regla de unicidad: si existe un despacho en `in_transit`, ningún otro puede pasar a ese estado hasta que el activo llegue a un estado terminal o a `incident`; la validación se hace sobre `local_dispatches` (SQLite) para funcionar sin conexión.<br>• Botones: «Iniciar Ruta del Día» en la cabecera de «Mi Jornada» (visible solo si hay despachos `assigned`) y «Iniciar traslado a este domicilio» en la parada activa; este último queda deshabilitado, con mensaje explicativo, si ya hay un despacho en `in_transit`.<br>• Modal de confirmación antes de «Entregado» y de «No entregado», mostrando código de guía y cliente.<br>• Criterio de terminado: pruebas unitarias de la máquina de estados que cubran cada transición legal, al menos 5 transiciones ilegales y la regla de unicidad. | Jaime | 5h | Por hacer |
| `ES-148` | `ST-30.2` | **Mobile Data: Cola FIFO en SQLite (`local_events`), idempotencia y sincronizador resiliente:**<br>• Extender el repositorio local para registrar eventos de transición con `event_id` (UUID v7), `dispatch_id`, tipo de evento, nuevo estado, `lat`, `lng`, `accuracy`, `occurred_at` (ISO UTC), `synced` (0/1), `attempts` y último error. El registro del evento y la actualización de `local_dispatches` ocurren en una sola transacción atómica.<br>• Mostrar el badge «Pendiente de sincronizar» mientras `synced = 0`.<br>• Conectar con `NetInfo` para vaciar la cola en orden estricto de ocurrencia al detectar conectividad: un evento a la vez, con el backoff existente (`ST-79.2`), sin enviar el siguiente hasta recibir respuesta del anterior.<br>• Interpretar las respuestas de `ST-30.3`: 200 marca `synced = 1`; 409 (evento desfasado o estado terminal ya registrado) descarta el evento, conserva el estado del servidor y avisa al repartidor; 422 deja el evento en espera con el motivo; 5xx o sin red reintenta con backoff.<br>• Garantizar que un evento desfasado nunca sobrescriba un estado terminal del servidor.<br>• Criterio de terminado: pruebas del repositorio y del sincronizador con red intermitente, evento duplicado, 409 y 422. | Joan | 5h | Por hacer |
| `ES-149` | `ST-30.3` | **Backend: Endpoint transaccional de cambio de estado del despacho (PATCH) con auditoría y eventos en tiempo real:**<br>• Recibir `UpdateDispatchStatusDto` con: nuevo estado, `eventId` (UUID v7 generado por la app, clave de idempotencia), `occurredAt` (UTC), `lat`, `lng`, `accuracy` y, para `delivered`, la evidencia ya registrada. Documentarlo con `@ApiOperation` y `@ApiResponse` (200, 400, 403, 404, 409, 422).<br>• Validar en una transacción que la transición sea legal según el estado actual en PostgreSQL y que el despacho pertenezca a la ruta del repartidor autenticado (403 si no).<br>• Para `delivered`, verificar que exista la evidencia POD que exige la política del pedido (`ES-31`); si falta, responder 422 indicando cuál.<br>• Idempotencia y orden: si el `eventId` ya fue procesado, responder 200 con el estado actual sin reaplicar; si el evento es anterior al último cambio registrado o el estado actual ya es terminal, responder 409 sin sobrescribir (resolución de conflictos de `ST-30.2`).<br>• Tras el commit, emitir el evento interno `dispatch.status_changed` hacia `DispatchEventsGateway` (payload `{ dispatchId, status, previousStatus, version, updatedAt }`), registrar el cambio en `dispatch_events` y, al pasar a `out_for_delivery`, disparar la publicación `dispatch.shipment.departed` (`ES-73`).<br>• Criterio de terminado: pruebas e2e de transición legal, ilegal (409), evidencia faltante (422), reenvío idempotente y acceso de otro repartidor (403). | Sergio | 4h | Por hacer |

---

#### `ES-31`: 3.5 Como Repartidor en entrega, requiero registrar evidencia digital mediante fotografía, firma en pantalla o código OTP según la política del pedido [RF-U05]
* **Épica:** 3.0 Aplicación Móvil de Repartidor y Telemetría (`ES-6`)
* **Asignado (PE):** Jaime | **Story Points (Spe):** 5 | **Estado:** Por hacer (To Do)
* **Fechas:** Inicio (FI): 14 de Oct | Vencimiento (FV): 15 de Oct
* **Dependencias:** Bloquea a (B): `ES-80` | Bloqueado por (BP): `ES-30`

> **Como** Repartidor en domicilio de destino  
> **Quiero** capturar la evidencia digital de entrega requerida de forma parametrizada (firma digital en pantalla, fotografía de respaldo o validación estricta de código OTP enviado al cliente)  
> **Para** contar con una prueba de entrega (POD) fehaciente, inmutable y legalmente válida que respalde la finalización del servicio según la política de seguridad de cada pedido sin fricciones innecesarias.

* **Criterios de Aceptación (GWT):**
  * **Escenario 1 (Entrega estándar en mano con captura de firma electrónica):**  
    **Dado** un pedido configurado con política de entrega en mano estándar (`hand_delivery_standard`),  
    **cuando** el receptor firma con el dedo en el lienzo táctil y confirma sus datos (nombre y documento),  
    **entonces** la app genera el archivo PNG en almacenamiento privado del dispositivo, almacena la firma y habilita la confirmación de la entrega.
  * **Escenario 2 (Entrega sin contacto autorizada con fotografía georreferenciada):**  
    **Dado** un pedido con política de entrega sin contacto autorizada previamente por el cliente (`contactless_delivery`),  
    **cuando** el repartidor captura la fotografía del paquete depositado en el lugar acordado,  
    **entonces** la app comprime la imagen a JPEG (< 500 KB), estampa los metadatos de timestamp y coordenadas GPS, y aprueba la entrega sin exigir firma.
  * **Escenario 3 (Entrega de alta seguridad con validación obligatoria de código OTP):**  
    **Dado** un pedido de alto valor o control estricto (`high_value_control`) donde el backend emitió un OTP de 6 dígitos al teléfono verificado del cliente por SMS,  
    **cuando** el cliente proporciona el código y el repartidor lo ingresa en la app con conexión activa,  
    **entonces** el backend valida criptográficamente el código, invalida el token para un solo uso y autoriza la entrega de forma inmediata.
  * **Escenario 4 (Bloqueo estricto de entrega ante OTP offline o intentos fallidos):**  
    **Dado** un pedido que exige validación de OTP obligatoria,  
    **cuando** el dispositivo del repartidor se encuentra sin conexión a internet o el código ingresado falla más de 3 veces consecutivas,  
    **entonces** la aplicación bloquea estrictamente la confirmación de entrega, prohíbe degradar o sustituir unilateralmente el OTP por firma o foto, y exige reconectarse a la red o registrar una incidencia operativa (`ES-32`) para autorización de Despacho.
  * **Escenario 5 (Almacenamiento persistente de binarios offline y subida multipart diferida):**  
    **Dado** un pedido cuya política permite firma o foto en modo fuera de línea,  
    **cuando** se confirma la entrega en zona sin cobertura,  
    **entonces** la app almacena los binarios en almacenamiento privado (`FileSystem.documentDirectory`), encola en SQLite (`local_events`) únicamente la referencia URI local `file://...` (sin saturar la BD con Base64) y, al recuperar conectividad, el sincronizador sube el archivo mediante `multipart/form-data` con reintentos seguros antes de purgar la copia local.

| ID Jira | Subtarea | Descripción Técnica y Tareas a Realizar | Asignado (PE) | Estimación | Estado |
| :--- | :--- | :--- | :---: | :---: | :---: |
| `ES-150` | `ST-31.1` | **Mobile UI/Native: Lienzo táctil para firma electrónica y exportación a disco privado:**<br>• Integrar `react-native-signature-canvas` con trazo suave y los botones «Limpiar trazo», «Cancelar» y «Confirmar acuse».<br>• Campos obligatorios de nombre completo y carnet de identidad (CI) de quien recibe; el botón de confirmar permanece deshabilitado si el lienzo está vacío, el nombre tiene menos de 3 caracteres o el CI está vacío.<br>• Exportar la firma como PNG al almacenamiento privado de la app (`FileSystem.documentDirectory`) con el nombre `evidenceId.png`, sin guardar Base64 en SQLite.<br>• Devolver al flujo de entrega el objeto de evidencia `{ evidenceId, type: 'signature', fileUri, receiverName, receiverDocument, capturedAt, lat, lng }` para que `ST-30.2` lo encole.<br>• Criterio de terminado: pruebas de componente de las validaciones (lienzo vacío, nombre o CI faltantes) y verificación de que el PNG queda en el directorio privado. | Jaime | 5h | Por hacer |
| `ES-151` | `ST-31.2` | **Mobile Native: Módulo de cámara (`expo-camera`), compresión JPEG y metadatos de ubicación:**<br>• Solicitar el permiso de cámara con el diálogo nativo y, si el usuario lo deniega, mostrar un mensaje propio con acceso a los ajustes del sistema; mostrar un visor previo con las opciones «Reintentar» y «Usar foto».<br>• Pipeline con `expo-image-manipulator`: redimensionar a un máximo de 1280 px por lado, calidad 0.7 y formato JPEG; si el resultado supera 500 KB, repetir con menor calidad hasta cumplirlo.<br>• Obtener la ubicación al capturar (`lat`, `lng`) y la hora UTC, y guardarlas junto al archivo en el registro del evento (el JPEG recomprimido puede perder los metadatos EXIF).<br>• Guardar la foto en el almacenamiento privado (`FileSystem.documentDirectory`) y encolar en `local_events` solo la referencia `file://…` y los metadatos.<br>• Este módulo lo reutilizan la evidencia sin contacto (`ES-31`), la incidencia (`ST-32.1`) y el retiro de devoluciones (`ST-80.1`).<br>• Criterio de terminado: pruebas del pipeline (peso final menor a 500 KB y dimensiones máximas) y del permiso denegado. | Jaime | 5h | Por hacer |
| `ES-152` | `ST-31.3` | **Backend: Almacenamiento multipart de evidencia POD y validación de OTP:**<br>• Endpoint de carga de evidencia (POST `multipart/form-data` con `FileInterceptor`) con los campos: `evidenceId` (UUID v7 de la app), `type` (`signature` \| `photo`), archivo, `capturedAt`, `lat`, `lng` y, para firma, `receiverName` y `receiverDocument`. Aceptar PNG para firma y JPEG para foto, con tamaño máximo de 500 KB; rechazar con 400 (campos), 413 (tamaño) o 415 (tipo).<br>• Guardar el archivo con nombre UUID mediante un puerto de almacenamiento en `src/plugins/storage/` (disco local en desarrollo; MinIO/S3 en otros entornos) y registrar en `delivery_evidences` la URL, nunca el binario. Si el `evidenceId` ya existe, responder 200 con el registro original.<br>• Endpoint de validación de OTP (POST) que reciba el código y lo compare contra el hash generado en `ST-31.4`: si es correcto, invalidarlo (un solo uso) y registrar una evidencia `otp` sin guardar el código en claro; si falla, sumar un intento en Redis y, al tercer fallo consecutivo, bloquear nuevos intentos y responder 423 indicando que se requiere una incidencia (`ES-32`).<br>• Tras registrar cualquier evidencia, emitir el evento interno `dispatch.evidence_registered` para que la torre de control la vea; el cambio a `delivered` ocurre solo por `ST-30.3`.<br>• Criterio de terminado: pruebas e2e de firma, foto, archivo demasiado grande, tipo inválido, OTP correcto, OTP incorrecto, tercer intento fallido y reenvío del mismo `evidenceId`. | Joan | 6h | Por hacer |
| `ES-199` | `ST-31.4` | **Backend: Generación, envío simulado por SMS y vigencia del código OTP:**<br>• Cuando un despacho con política `high_value_control` pasa a `in_transit` (evento interno `dispatch.status_changed`), generar un OTP numérico de 6 dígitos con un generador criptográficamente seguro y guardar solo su hash con sal; el valor en claro nunca se persiste ni se escribe en logs.<br>• Guardar en Redis, por despacho, el hash, la vigencia (parámetro `otp_ttl_minutes` de `settings`, 60 minutos por defecto) y el contador de intentos fallidos (máximo 3); estas claves las comparte el validador de `ST-31.3`.<br>• Enviar el código al teléfono verificado del cliente mediante el puerto `SmsSender` en `src/plugins/sms/`. En este proyecto el adaptador es simulado y escribe el mensaje en un registro estructurado de desarrollo; en un entorno real se sustituye por el proveedor SMS.<br>• Reenvío controlado: máximo 2 reenvíos por despacho; cada uno invalida el código anterior y reinicia el TTL, pero no el contador de intentos.<br>• Si el despacho no tiene teléfono verificado, no generar código y marcar el despacho para que Despacho lo resuelva; el repartidor no puede degradar el OTP (`ES-31`, Escenario 4).<br>• Criterio de terminado: pruebas unitarias de generación, expiración según `otp_ttl_minutes`, bloqueo tras 3 intentos y reenvío; el adaptador simulado registra el mensaje enviado. | Sergio | 3h | Por hacer |
| `ES-200` | `ST-31.5` | **BD: Metadatos de evidencia POD, motivos de incidencia y parámetros iniciales:**<br>• Migración que agrega a `delivery_evidences` las columnas `captured_at` (TIMESTAMPTZ), `latitude` y `longitude` (NUMERIC(9,6)), `receiver_name` (VARCHAR(150)) y `receiver_document` (VARCHAR(30)), todas opcionales según el tipo de evidencia. El código OTP en claro no se persiste: `otp_code` queda nulo.<br>• Migración de datos de `incident_reasons`: marcar `requires_evidence = true` en los motivos operativos de `ES-32` y agregar el motivo «Rechazo del paquete por el cliente» con código `INC-CUST-REJECTED`.<br>• Sembrar en `settings` los parámetros iniciales: `otp_ttl_minutes` = 60, `order_reservation_ttl_minutes` = 15 e `incomplete_order_alert_minutes` = 60.<br>• `up.sql`, `down.sql` y `README.md` en `migrations/`; actualizar los `DICTIONARY.md` afectados y `db-output.sql`.<br>• Criterio de terminado: migración aplicada y revertida sin errores sobre una base con datos. | Joan | 3h | Por hacer |

---

#### `ES-32`: 3.6 Como Repartidor en ruta, necesito reportar incidencias operativas seleccionando causales tipificadas y evidencia [RF-U06]
* **Épica:** 3.0 Aplicación Móvil de Repartidor y Telemetría (`ES-6`)
* **Asignado (PE):** Jaime | **Story Points (Spe):** 3 | **Estado:** Por hacer (To Do)
* **Fechas:** Inicio (FI): 16 de Oct | Vencimiento (FV): 19 de Oct
* **Dependencias:** Bloquea a (B): `ES-66` | Bloqueado por (BP): `ES-24 (✅ Finalizada en Sprint 1)`, `ES-30`

> **Como** Repartidor ante un imprevisto o bloqueo en calle  
> **Quiero** reportar incidencias operativas seleccionando causales tipificadas (cliente ausente, dirección inaccesible, producto dañado) adjuntando observaciones y evidencia fotográfica obligatoria  
> **Para** justificar operativamente la imposibilidad de entrega en la parada programada y notificar de inmediato a la torre de control para que Despacho decida la reprogramación o ajuste de ruta.

* **Criterios de Aceptación (GWT):**
  * **Escenario 1 (Selección de causal desde catálogo tipificado y validación de foto obligatoria):**  
    **Dado** un despacho en estado `in_transit` que no puede concretarse en destino,  
    **cuando** el repartidor abre el formulario de incidencias,  
    **entonces** la app despliega el catálogo de motivos de incidencia activos, sincronizado desde el backend y administrado en `ES-24` (por ejemplo: cliente ausente `INC-CUST-ABSENT`, dirección incorrecta `INC-ADDR-INVALID`, producto dañado `INC-PROD-DAMAGED` y rechazo del paquete por el cliente `INC-CUST-REJECTED`), y exige la fotografía de respaldo (ej. fachada cerrada con número visible o bulto dañado) antes de permitir el envío; los motivos operativos de este sprint se configuran con `requires_evidence` activo.
  * **Escenario 2 (Transición a estado 'Incidencia' y liberación de ruta):**  
    **Dado** que el repartidor confirma el reporte de incidencia con su fotografía y comentarios,  
    **cuando** se procesa la transacción,  
    **entonces** el despacho pasa a estado `incident`, se remueve de la parada activa inmediata y el repartidor queda habilitado para continuar con la siguiente parada de su jornada.
  * **Escenario 3 (Almacenamiento offline de reportes de incidencia):**  
    **Dado** un repartidor sin cobertura celular en el momento de reportar una incidencia en puerta,  
    **cuando** confirma el formulario con la foto de fachada,  
    **entonces** el evento se guarda localmente en `local_events` de SQLite con bandera de sincronización pendiente y la orden se actualiza localmente permitiendo continuar la ruta sin bloqueos.

| ID Jira | Subtarea | Descripción Técnica y Tareas a Realizar | Asignado (PE) | Estimación | Estado |
| :--- | :--- | :--- | :---: | :---: | :---: |
| `ES-153` | `ST-32.1` | **Mobile UI: Formulario de reporte de incidencias con motivos del catálogo, foto y observaciones:**<br>• Diseñar una pantalla modal que lea los motivos activos desde la tabla local `incident_reasons` (sincronizada desde el catálogo de `ES-24`) y los muestre en un selector; no debe haber motivos escritos en el código.<br>• Si el motivo elegido tiene `requires_evidence`, exigir la fotografía reutilizando el pipeline de captura y compresión de `ST-31.2` y mantener deshabilitado el botón de envío mientras falte.<br>• Campo de observaciones multilínea; mínimo 10 caracteres si el motivo es «Otro».<br>• Al confirmar, encolar el evento en `local_events` y transicionar localmente el despacho a `incident` con la máquina de estados (`ST-30.1`), liberando la parada activa y habilitando la siguiente.<br>• Criterio de terminado: prueba de componente del formulario (con y sin evidencia requerida) y prueba del flujo sin conexión. | Jaime | 5h | Por hacer |
| `ES-154` | `ST-32.2` | **Backend: Registro de incidencia del despacho (POST) y alerta operativa en tiempo real:**<br>• Recibir `CreateDispatchIncidentDto` con: `incidentId` (UUID v7 de la app, clave de idempotencia), código del motivo del catálogo `incident_reasons`, observaciones (obligatorias, mínimo 10 caracteres, si el motivo es «Otro»), `occurredAt`, `lat`, `lng` y la foto ya cargada con `ST-31.3`. Validar que el motivo exista y esté activo; si tiene `requires_evidence`, exigir la foto (422 si falta).<br>• En una sola transacción: persistir en `dispatch_incidents`, pasar el despacho a `incident` (si la transición es legal desde `in_transit`) y registrar el cambio en `dispatch_events`.<br>• Tras el commit, emitir `dispatch.incident_reported` por `DispatchEventsGateway`, con criticidad alta, a la sala del coordinador y supervisor de la zona del despacho (payload `{ dispatchId, incidentId, reasonCode, reportedAt }`).<br>• Reenvíos con el mismo `incidentId` responden 200 con el registro original, sin duplicar.<br>• Documentar con Swagger (201, 200, 400, 403, 404, 409, 422).<br>• Criterio de terminado: pruebas e2e del flujo completo, motivo inactivo, foto faltante y reenvío idempotente. | Sergio | 4h | Por hacer |

---

#### `ES-33`: 3.7 Como Repartidor, requiero consultar el historial de entregas concluidas en jornadas anteriores [RF-U07]
* **Épica:** 3.0 Aplicación Móvil de Repartidor y Telemetría (`ES-6`)
* **Asignado (PE):** Joan | **Story Points (Spe):** 3 | **Estado:** Por hacer (To Do)
* **Fechas:** Inicio (FI): 16 de Oct | Vencimiento (FV): 19 de Oct
* **Dependencias:** Bloquea a (B): Ninguna | Bloqueado por (BP): `ES-30`

> **Como** Repartidor  
> **Quiero** revisar el histórico de despachos finalizados en días previos con filtros por fecha y resultado  
> **Para** auditar mis entregas realizadas, corroborar liquidaciones y responder ante cualquier consulta de ruta.

* **Criterios de Aceptación (GWT):**
  * **Escenario 1 (Filtrado por rango temporal en historial móvil):**  
    **Dado** que el repartidor ingresa al módulo "Historial",  
    **cuando** selecciona un rango de fechas (últimos 7 días, mes actual),  
    **entonces** la app muestra las órdenes cerradas organizadas cronológicamente con su comprobante de entrega y estado final (`delivered`, `returned`).
  * **Escenario 2 (Gestión de estado vacío sin registros en el rango):**  
    **Dado** un período seleccionado en el que el repartidor no registró despachos concluidos,  
    **cuando** se procesa la consulta,  
    **entonces** la vista despliega una ilustración con el mensaje "No se encontraron entregas en este período" y un botón para reajustar los filtros de fecha.
  * **Escenario 3 (Consulta de historial reciente en modo fuera de línea):**  
    **Dado** que el repartidor pierde señal en ruta,  
    **cuando** accede a consultar entregas concluidas en las últimas 48 horas,  
    **entonces** la aplicación lee los registros persistidos en la base SQLite local permitiendo revisar los acuses y detalles sin requerir internet.

| ID Jira | Subtarea | Descripción Técnica y Tareas a Realizar | Asignado (PE) | Estimación | Estado |
| :--- | :--- | :--- | :---: | :---: | :---: |
| `ES-155` | `ST-33.1` | **Mobile UI/Data: Pestaña «Historial» con paginación infinita y consulta local SQLite:**<br>• Vista de historial con buscador por código de guía (debounce de 300 ms) y selector de rango de fechas con los atajos «Últimos 7 días» y «Mes actual».<br>• Tarjeta compacta: código de guía, cliente, resultado (`delivered` o `returned`), fecha y hora de cierre y botón «Ver comprobante» que abre la evidencia.<br>• Con conexión, consultar al backend por páginas (`ST-33.2`) y cargar la siguiente al llegar al final de la lista. Sin conexión, leer de SQLite los despachos cerrados en las últimas 48 horas y mostrar un aviso de que el historial está limitado.<br>• Estado vacío ilustrado «No se encontraron entregas en este período» con un botón para reajustar los filtros.<br>• Criterio de terminado: pruebas de filtros, paginación, estado vacío y lectura sin conexión. | Joan | 4h | Por hacer |
| `ES-156` | `ST-33.2` | **Backend: Historial paginado del repartidor (GET) con filtros de fecha y resultado:**<br>• Aceptar los parámetros `page` (≥ 1), `limit` (1 a 50, por defecto 20), `from` y `to` (fechas ISO), `status` (`delivered` \| `returned`) y `search` por código de guía. El repartidor se toma siempre del token, nunca de un parámetro.<br>• Consulta TypeORM indexada por conductor (a través de la ruta del despacho) y estados terminales, ordenada por fecha de cierre descendente.<br>• Respuesta `{ data, total, page, limit }`; cada elemento incluye código de guía, cliente, estado final, fecha y hora de cierre, receptor y el identificador de la evidencia para consultar el comprobante.<br>• Documentar con Swagger; una consulta sin resultados devuelve lista vacía, no un error.<br>• Criterio de terminado: pruebas e2e de paginación, rango de fechas y aislamiento entre repartidores. | Sergio | 3h | Por hacer |

---

#### `ES-35`: 3.9 Como Repartidor, necesito visualizar el panel con mis métricas de desempeño y calificación de servicio [RF-U09]
* **Épica:** 3.0 Aplicación Móvil de Repartidor y Telemetría (`ES-6`)
* **Asignado (PE):** Jaime | **Story Points (Spe):** 2 | **Estado:** Por hacer (To Do)
* **Fechas:** Inicio (FI): 16 de Oct | Vencimiento (FV): 19 de Oct
* **Dependencias:** Bloquea a (B): Ninguna | Bloqueado por (BP): `ES-30`

> **Como** Repartidor  
> **Quiero** visualizar indicadores claros sobre mi productividad personal (entregas efectivas, porcentaje de éxito y calificación)  
> **Para** conocer mi rendimiento operativo diario y enfocarme en los estándares de calidad del servicio.

* **Criterios de Aceptación (GWT):**
  * **Escenario 1 (Despliegue de métricas en app móvil):**  
    **Dado** el perfil del repartidor,  
    **cuando** abre la sección "Mi Desempeño",  
    **entonces** se despliegan cards visuales con: total entregados hoy, entregas a tiempo (% SLA) y calificación promedio recibida.
  * **Escenario 2 (Inicialización de métricas para conductores nuevos o sin calificaciones):**  
    **Dado** un conductor nuevo o una jornada sin calificaciones externas registradas,  
    **cuando** consulta su resumen,  
    **entonces** el sistema presenta su récord de efectividad de entregas basado en las órdenes finalizadas del día y una calificación base neutra (5.0 / Excelente) con la leyenda "Calificación inicial de servicio".
  * **Escenario 3 (Selector de ventanas temporales con optimización de caché):**  
    **Dado** el panel de KPIs,  
    **cuando** el repartidor alterna entre "Hoy", "Esta Semana" y "Este Mes",  
    **entonces** la aplicación consulta el endpoint agregado optimizado con caché Redis (TTL 5 min) mostrando los porcentajes comparativos sin demoras de renderizado.

| ID Jira | Subtarea | Descripción Técnica y Tareas a Realizar | Asignado (PE) | Estimación | Estado |
| :--- | :--- | :--- | :---: | :---: | :---: |
| `ES-157` | `ST-35.1` | **Mobile UI: Panel «Mi Desempeño» con KPIs personales del conductor:**<br>• Tarjetas con barras circulares de progreso: entregas efectivas del período, porcentaje de entregas a tiempo (SLA) y total de despachos asignados.<br>• Selector de período «Hoy», «Esta Semana» y «Este Mes» con transición suave; cada cambio consulta `ST-35.2` y mantiene en pantalla los datos anteriores con un indicador de carga hasta recibir la respuesta.<br>• Tarjeta de calificación promedio con estrellas y contador de valoraciones; cuando `ratingsCount` es 0, mostrar la calificación inicial neutra «5.0 – Calificación inicial de servicio».<br>• Estados de carga (skeleton), error con reintento y sin conexión (mostrar el último dato guardado con su fecha).<br>• Criterio de terminado: pruebas de componente del selector de período, del caso sin calificaciones y del estado de error. | Jaime | 3h | Por hacer |
| `ES-158` | `ST-35.2` | **Backend: Métricas agregadas del repartidor (GET) con caché Redis:**<br>• Parámetro `period` (`today` \| `week` \| `month`); el repartidor se toma del token.<br>• Calcular con agregaciones SQL en PostgreSQL: total de entregas `delivered`, porcentaje entregado dentro de la franja comprometida (`scheduled_window_start/end`), total de despachos asignados y calificación promedio con cantidad de valoraciones (`dispatch_ratings`). Si no hay calificaciones, devolver `rating: null` y `ratingsCount: 0` para que la app muestre la calificación inicial neutra (5.0).<br>• Guardar el resultado en Redis con la clave `metrics:driver:{driverId}:{period}` y TTL de 5 minutos; invalidar las claves del repartidor cuando se confirme una entrega.<br>• Si Redis no responde, calcular directo en PostgreSQL y registrar una advertencia, sin fallar la petición.<br>• Criterio de terminado: pruebas de cálculo con datos de ejemplo, de caché (acierto, fallo, invalidación) y de repartidor sin historial. | Joan | 3h | Por hacer |

---

#### `ES-36`: 3.10 Como Repartidor en turno, requiero recibir notificaciones push nativas ante la asignación de un nuevo pedido [RF-U10]
* **Épica:** 3.0 Aplicación Móvil de Repartidor y Telemetría (`ES-6`)
* **Asignado (PE):** Joan | **Story Points (Spe):** 3 | **Estado:** Por hacer (To Do)
* **Fechas:** Inicio (FI): 18 de Oct | Vencimiento (FV): 19 de Oct
* **Dependencias:** Bloquea a (B): Ninguna | Bloqueado por (BP): `ES-28`

> **Como** Repartidor en turno activo  
> **Quiero** recibir alertas push audibles y visibles en mi smartphone en el momento exacto en que me asignan un pedido o ruta  
> **Para** enterarme inmediatamente de adiciones o ajustes a mi hoja de ruta sin necesidad de recargar constantemente la pantalla.

* **Criterios de Aceptación (GWT):**
  * **Escenario 1 (Recepción de push notification en primer o segundo plano):**  
    **Dado** un repartidor con sesión activa,  
    **cuando** el coordinador le asigna una orden en el portal web,  
    **entonces** el dispositivo móvil emite una alerta nativa sonora y muestra la notificación con el código del pedido y zona.
  * **Escenario 2 (Apertura directa al tocar la notificación mediante Deep Link):**  
    **Dado** que el repartidor presiona la notificación recibida,  
    **cuando** la app responde al intent de apertura,  
    **entonces** redirige directamente a la vista de detalle del nuevo pedido asignado (`/pedido/:id`).

| ID Jira | Subtarea | Descripción Técnica y Tareas a Realizar | Asignado (PE) | Estimación | Estado |
| :--- | :--- | :--- | :---: | :---: | :---: |
| `ES-159` | `ST-36.1` | **Mobile Native: Notificaciones push con Expo Notifications y navegación por deep link:**<br>• Solicitar el permiso de notificaciones después del primer inicio de sesión y explicar el motivo si el usuario lo deniega.<br>• Crear el canal Android `assignments` con importancia alta, patrón de vibración y sonido corporativo.<br>• Obtener el Expo Push Token y enviarlo al backend junto con el `device_id` después de cada inicio de sesión exitoso (y cuando el token cambie); al cerrar sesión, solicitar su revocación.<br>• Listener `addNotificationResponseReceivedListener`: al tocar la notificación, abrir `/pedido/[id]` con el `dispatchId` del payload, con la app en primer plano, en segundo plano o cerrada; si el pedido no existe localmente, sincronizar antes de abrirlo.<br>• Mostrar la notificación con sonido también cuando la app está en primer plano.<br>• Criterio de terminado: pruebas del registro del token, de la revocación y de la navegación con un payload de ejemplo. | Joan | 4h | Por hacer |
| `ES-160` | `ST-36.2` | **Backend: Servicio emisor de notificaciones push en NestJS:**<br>• Crear el puerto `PushSender` en `src/plugins/push/` con un adaptador para Expo Server SDK (y Firebase Admin SDK como alternativa configurable) y un adaptador simulado para pruebas.<br>• Suscribirse al evento interno `route.assigned` (`ST-51.2`) y enviar al repartidor una notificación con el payload `{ title, body, dispatchId, zoneName, priority }`: una por ruta asignada indicando la cantidad de pedidos, o una por cada pedido agregado a una ruta ya asignada.<br>• Reintentos silenciosos: hasta 3 intentos con backoff ante errores temporales del proveedor, sin bloquear la asignación de la ruta.<br>• Procesar los recibos de entrega: si el proveedor responde `DeviceNotRegistered` (token caducado o app desinstalada), revocar ese token en `push_tokens` (`ST-36.3`).<br>• Criterio de terminado: pruebas con el adaptador simulado para envío correcto, error temporal con reintento y limpieza de un token inválido. | Sergio | 4h | Por hacer |
| `ES-198` | `ST-36.3` | **BD/Backend: Entidad y persistencia de Push Tokens en PostgreSQL:**<br>• Migración en `migrations/` (con `up.sql`, `down.sql` y `README.md`) que crea la tabla `push_tokens` (`push_token_id`, `user_id` → `users`, `device_id`, `token`, `platform`, `last_used_at`, `revoked_at`, `created_at`, `updated_at`) con restricción única sobre (`user_id`, `device_id`).<br>• Entidad TypeORM y repositorio con: registrar o actualizar el token de un dispositivo en una operación atómica (`upsert`), revocar el token al cerrar sesión y listar los tokens activos de un usuario.<br>• Documentar la tabla en su `DICTIONARY.md` y actualizar `db-output.sql`.<br>• Criterio de terminado: migración aplicada y revertida sin errores y pruebas del repositorio (un registro repetido no duplica y la revocación excluye el token de los activos). | Joan | 2h | Por hacer |

---

#### `ES-78`: 3.12 Como Repartidor en ruta, necesito abrir la navegación guiada hacia la dirección de entrega en Google Maps o Waze [RF-U12]
* **Épica:** 3.0 Aplicación Móvil de Repartidor y Telemetría (`ES-6`)
* **Asignado (PE):** Jaime | **Story Points (Spe):** 5 | **Estado:** Por hacer (To Do)
* **Fechas:** Inicio (FI): 12 de Oct | Vencimiento (FV): 13 de Oct
* **Dependencias:** Bloquea a (B): Ninguna | Bloqueado por (BP): `ES-29`

> **Como** Repartidor en ruta hacia el cliente  
> **Quiero** lanzar la navegación guiada por voz en Google Maps o Waze con un solo toque desde la orden  
> **Para** conducir de forma segura optimizando el tiempo de llegada y evitando desvíos en el tráfico.

* **Criterios de Aceptación (GWT):**
  * **Escenario 1 (Lanzamiento de ruta con coordenadas validadas hacia mapas nativos - RF-U12):**  
    **Dado** un pedido con latitud y longitud válidas en el destino,  
    **cuando** se pulsa el botón "Navegar con GPS",  
    **entonces** la app abre la herramienta de mapas seleccionada del dispositivo con el destino fijado y el modo conducción activado.
  * **Escenario 2 (Selector modal y persistencia de app de navegación favorita):**  
    **Dado** un dispositivo con Google Maps y Waze instalados simultáneamente,  
    **cuando** el repartidor solicita navegación por primera vez,  
    **entonces** la app muestra un selector modal para elegir la herramienta deseada y una opción "Recordar mi elección" persistiendo la preferencia en `AsyncStorage`.
  * **Escenario 3 (Fallback ante ausencia de coordenadas precisas o apps externas):**  
    **Dado** un pedido cuya dirección carezca de coordenadas georreferenciadas o un entorno sin apps de mapas instaladas,  
    **cuando** se activa la navegación,  
    **entonces** la aplicación genera un URL Web universal de búsqueda con la dirección textual y abre el navegador del sistema sin fallar ni congelar la pantalla.

| ID Jira | Subtarea | Descripción Técnica y Tareas a Realizar | Asignado (PE) | Estimación | Estado |
| :--- | :--- | :--- | :---: | :---: | :---: |
| `ES-161` | `ST-78.1` | **Mobile Native: Módulo unificado de geonavegación externa:**<br>• Modal selector para elegir entre Google Maps, Waze o Apple Maps (este último solo en iOS), con la opción «Recordar mi elección» que guarda la preferencia en `AsyncStorage`. Si hay una preferencia guardada y la app sigue instalada, abrirla directamente sin mostrar el modal, y ofrecer cambiarla desde los ajustes.<br>• Construir las URL con las coordenadas y el nombre de la vía usando el utilitario de `ST-29.2`.<br>• Detectar las apps instaladas con `Linking.canOpenURL` y mostrar en el selector solo las disponibles.<br>• Fallback: si el destino no tiene coordenadas, o no hay ninguna app de mapas instalada, abrir en el navegador del sistema una URL web de búsqueda con la dirección textual, sin congelar la pantalla.<br>• Criterio de terminado: pruebas de preferencia guardada, ausencia de apps, coordenadas faltantes y apertura de cada URL. | Jaime | 4h | Por hacer |

---

#### `ES-80`: 3.14 Como Repartidor en ruta, requiero visualizar y gestionar órdenes de recojo para logística inversa con inspección, checklist y firma [RF-U23]
* **Épica:** 3.0 Aplicación Móvil de Repartidor y Telemetría (`ES-6`)
* **Asignado (PE):** Joan | **Story Points (Spe):** 3 | **Estado:** Por hacer (To Do)
* **Fechas:** Inicio (FI): 16 de Oct | Vencimiento (FV): 19 de Oct
* **Dependencias:** Bloquea a (B): Ninguna | Bloqueado por (BP): `ES-31`

> **Como** Repartidor ejecutando logística inversa  
> **Quiero** visualizar las órdenes de recojo en domicilio asignadas a mi recorrido, completar un checklist de inspección física obligatoria, registrar observaciones y capturar firma/foto de retiro  
> **Para** transportar mercadería devuelta con constancia digital formal (RMA) hacia el centro de distribución con total respaldo operativo.

* **Criterios de Aceptación (GWT):**
  * **Escenario 1 (Inspección física y checklist estandarizado de recojo - RF-U23):**  
    **Dado** un servicio de recojo domiciliario aprobado por Almacén (RMA),  
    **cuando** el repartidor llega al domicilio e inspecciona el producto,  
    **entonces** la app le exige completar un checklist estandarizado de 3 puntos: `[ ] Empaque original o adecuado`, `[ ] Piezas y accesorios completos`, `[ ] Sin daños por uso indebido`, además de un campo de observaciones abierto obligatorio si desmarca alguna opción.
  * **Escenario 2 (Firma de constancia y transición a 'Recogido en Tránsito'):**  
    **Dado** el checklist validado y fotografía del producto capturada,  
    **cuando** el cliente estampa su firma digital en la app confirmando la entrega del bien,  
    **entonces** la orden transiciona al estado `picked_up_in_transit`, emite el acuse digital formal de retiro y queda registrada para su posterior entrega en el depósito central.
  * **Escenario 3 (Recepción final y cierre en almacén central):**  
    **Dado** que el repartidor arriba al centro de distribución con la mercadería retirada,  
    **cuando** el encargado de almacén valida físicamente el paquete contra el ticket RMA,  
    **entonces** Despachos registra la entrega en el depósito, publica la novedad hacia Almacén y, al recibir la confirmación de recepción de Almacén (`inventory.receipt.confirmed`), transiciona la orden a `returned_to_warehouse`, liberando la custodia del chofer y cerrando el ciclo logístico inverso.

| ID Jira | Subtarea | Descripción Técnica y Tareas a Realizar | Asignado (PE) | Estimación | Estado |
| :--- | :--- | :--- | :---: | :---: | :---: |
| `ES-162` | `ST-80.1` | **Mobile UI/Flow: Flujo de recojo de devolución en domicilio con checklist:**<br>• Diferenciar en «Mi Jornada» las órdenes de recojo (`warehouse_return_pickup`) con borde ámbar e ícono de retorno.<br>• Formulario con el checklist de 3 puntos (empaque original o adecuado, piezas y accesorios completos, sin daños por uso indebido) y campo de observaciones; si algún punto está desmarcado, las observaciones son obligatorias.<br>• Captura obligatoria de la fotografía del estado físico (`ST-31.2`) y de la firma del cliente con nombre y CI (`ST-31.1`).<br>• Al confirmar, encolar el retiro en `local_events` y marcar localmente `picked_up_in_transit`, mostrando el acuse digital del retiro.<br>• Botón «Llegué al depósito» que registra la llegada; el cierre `returned_to_warehouse` lo aplica el servidor cuando Almacén confirma la recepción (`ST-80.2`) y la app lo refleja al sincronizar.<br>• Criterio de terminado: pruebas del checklist (con y sin observaciones), de los campos obligatorios y del flujo sin conexión. | Joan | 5h | Por hacer |
| `ES-163` | `ST-80.2` | **Backend: Registro del retiro en domicilio (POST) y cierre por confirmación de Almacén:**<br>• Endpoint de retiro (POST): recibir el checklist de 3 puntos (empaque, piezas y accesorios, sin daños), observaciones (obligatorias si algún punto está desmarcado) y la foto y firma ya cargadas con `ST-31.3`; validar que el despacho sea de tipo `warehouse_return_pickup` y esté en `in_transit`, y pasarlo a `picked_up_in_transit` en una transacción con registro en `dispatch_events`.<br>• Al llegar al depósito, el repartidor registra la llegada desde la app (evento encolado como cualquier otro) sin cambiar el estado; el sistema publica `dispatch.transfer.status_changed` hacia Almacén (`ES-73`).<br>• El estado `returned_to_warehouse` no lo fija el repartidor: se aplica al consumir el mensaje `inventory.receipt.confirmed`. Si Almacén reporta diferencias o daños, registrar una incidencia para que Despacho decida.<br>• Criterio de terminado: pruebas e2e de retiro con checklist incompleto sin observaciones (422), retiro válido y cierre simulado con el mensaje de recepción. | Sergio | 4h | Por hacer |

---

### ÉPICA 5.0 — Panel Administrativo de Coordinación y Despacho (`ES-8`)

#### `ES-48`: 5.1 Como Coordinador de logística, necesito visualizar la bandeja central de pedidos consolidados pendientes de despacho [RF-A40]
* **Épica:** 5.0 Panel Administrativo de Coordinación y Despacho (`ES-8`)
* **Asignado (PE):** Pardo | **Story Points (Spe):** 3 | **Estado:** Por hacer (To Do)
* **Fechas:** Inicio (FI): 05 de Oct | Vencimiento (FV): 06 de Oct
* **Dependencias:** Bloquea a (B): `ES-51` | Bloqueado por (BP): `ES-14 (✅ Finalizada en Sprint 1)`

> **Como** Coordinador de Logística  
> **Quiero** acceder a una bandeja central estructurada con todos los pedidos listos y pendientes de asignación, visualizando reservas concurrentes de otros operadores y pedidos con dirección en revisión  
> **Para** revisar la carga entrante, evitar colisiones operativas con otros coordinadores y organizar las rutas del turno de forma eficiente.

* **Criterios de Aceptación (GWT):**
  * **Escenario 1 (Visualización de la bandeja central con filtros rápidos y estados - RF-A40):**  
    **Dado** un usuario con rol `COORDINADOR` autenticado en el portal web,  
    **cuando** accede al módulo de Despachos,  
    **entonces** visualiza la tabla de pedidos consolidados en estado `pending` con columnas estructuradas (N° de Orden, Cliente, Dirección, Zona Logística, Ventana Horaria, Bultos, Peso Total y Badge de Urgencia) y filtros rápidos por zona y turno.
  * **Escenario 2 (Detección de pedidos reservados por otro coordinador en planificación):**  
    **Dado** que otro coordinador se encuentra armando una ruta preliminar con un lote de órdenes,  
    **cuando** el usuario consulta la bandeja de pedidos,  
    **entonces** dichas órdenes se exhiben deshabilitadas para selección con un badge informativo ámbar: "En planificación por [Nombre del Coordinador]", impidiendo su doble inclusión en otra ruta.
  * **Escenario 3 (Búsqueda reactiva con debounce y gestión de estado vacío):**  
    **Dado** un volumen extenso de órdenes en cola,  
    **cuando** el coordinador escribe en la barra de búsqueda,  
    **entonces** la tabla aplica un debounce de 300ms y filtra los resultados en tiempo real sin recargar la página; si no hay pedidos, despliega un estado ilustrado limpio con botón de refresco manual.

| ID Jira | Subtarea | Descripción Técnica y Tareas a Realizar | Asignado (PE) | Estimación | Estado |
| :--- | :--- | :--- | :---: | :---: | :---: |
| `ES-164` | `ST-48.1` | **Frontend Web: Bandeja central de pedidos con TanStack Table, filtros y badges de reserva:**<br>• Tabla con `@tanstack/react-table` y shadcn/ui (`DataTable`) que consume `ST-48.2`. Columnas: N° de orden, destinatario, dirección, zona, franja, bultos, peso (kg), volumen (m³) y prioridad, con ordenamiento en el servidor.<br>• Badges: «En planificación por [Nombre]» (ámbar) para los pedidos reservados por otro coordinador, con la fila deshabilitada para selección; «Dirección en revisión» (`address_review`); y badge de urgencia para `urgent`.<br>• Barra de herramientas: buscador reactivo con debounce de 300 ms, filtros por zona logística y turno, y botón de refresco manual.<br>• Estados: skeleton con la estructura de la tabla, estado vacío ilustrado con botón de refresco y mensaje de error con reintento.<br>• Respetar la regla de fronteras `pages/ → domains/ → shared/`, importar solo a través del barrel del dominio y usar `useCrud` y `createCrudService` donde aplique.<br>• Criterio de terminado: pruebas de componente de la tabla (orden, filas deshabilitadas por reserva, estado vacío) y `npm run typecheck` y `npm run lint` sin errores. | Pardo | 4h | Por hacer |
| `ES-197` | `ST-48.2` | **Backend: Bandeja paginada de pedidos pendientes (GET) con filtros y reserva suave:**<br>• Devolver los despachos `pending` y los `rescheduled` cuya fecha pactada sea hoy; excluir cualquier otro estado.<br>• Filtros opcionales: zona (`delivery_zone_id`), turno, prioridad (`urgent` \| `normal`) y búsqueda difusa por cliente o código de guía. Orden por defecto: urgentes primero y luego por vencimiento de SLA; admitir `sortBy` y `order` sobre N° de orden, destinatario, zona, franja, peso y prioridad.<br>• Respuesta `{ data, total, page, limit }` con `limit` máximo de 100; cada elemento incluye peso total y volumen de sus bultos, la franja y, si está reservado por otro coordinador, su nombre y la hora en que vence la reserva.<br>• Reserva suave: cuando un coordinador empieza a planificar una ruta con un lote de pedidos, esos pedidos quedan reservados a su nombre y los demás coordinadores los ven bloqueados («En planificación por [Nombre]»). La reserva se renueva con la actividad del coordinador y se libera sola tras `order_reservation_ttl_minutes` (clave de `settings`, 15 por defecto) sin actividad, o al confirmar o cancelar la planificación; mientras exista, el pedido se muestra como `in_planning`.<br>• Criterio de terminado: pruebas e2e de filtros, orden, paginación, visibilidad de reservas, renovación y expiración. | Joan | 4h | Por hacer |

---

#### `ES-49`: 5.2 Como Coordinador de despacho, requiero que el sistema genere automáticamente órdenes de despacho desde pedidos listos en almacén [RF-A02]
* **Épica:** 5.0 Panel Administrativo de Coordinación y Despacho (`ES-8`)
* **Asignado (PE):** Sergio | **Story Points (Spe):** 5 | **Estado:** Por hacer (To Do)
* **Fechas:** Inicio (FI): 08 de Oct | Vencimiento (FV): 11 de Oct
* **Dependencias:** Bloquea a (B): `ES-58`, `ES-50` | Bloqueado por (BP): `ES-72`, `ES-57`

> **Como** Coordinador de Despacho  
> **Quiero** que el sistema transforme automáticamente los datos de entrega recibidos de Marketplace y los pedidos empacados avisados por Almacén en órdenes de despacho consolidadas con ubicación validada y zona asignada  
> **Para** eliminar la digitación manual y disponer de las órdenes de forma inmediata en la bandeja de asignación.

* **Criterios de Aceptación (GWT):**
  * **Escenario 1 (Conversión de pedido preparado en orden de despacho - RF-A02):**  
    **Dado** la información consolidada de un pedido de venta (datos de entrega de Marketplace y bultos de Almacén) entregada por `ES-72`,  
    **cuando** el procesador ejecuta la transformación,  
    **entonces** se genera una orden en estado `pending` asignándole zona según las coordenadas recibidas y calculando la ventana horaria viable.
  * **Escenario 2 (Aplicación de reglas operativas basadas en ES-57):**  
    **Dado** un aviso de despacho entrante,  
    **cuando** se calcula la ventana de atención del pedido,  
    **entonces** el sistema aplica los parámetros de turnos (mañana/tarde) y tiempos máximos de espera configurados en `ES-57`, asignando la fecha límite calculada de SLA.
  * **Escenario 3 (Manejo de pedidos sin coordenadas válidas o fuera de cobertura):**  
    **Dado** un pedido sin coordenadas válidas o cuyo punto caiga fuera de las zonas logísticas registradas (`ES-21`),  
    **cuando** se intenta la generación automática,  
    **entonces** la orden se registra en estado `address_review` emitiendo una alerta en la bandeja del Coordinador para fijar el punto en el mapa de forma manual.

| ID Jira | Subtarea | Descripción Técnica y Tareas a Realizar | Asignado (PE) | Estimación | Estado |
| :--- | :--- | :--- | :---: | :---: | :---: |
| `ES-165` | `ST-49.1` | **Backend: Servicio de generación de órdenes de despacho a partir de pedidos consolidados:**<br>• Recibir de `ST-72.2` el conjunto consolidado (datos de entrega de Marketplace y bultos de Almacén) y crear, en una transacción, el despacho (`dispatch_type = delivery`, estado `pending`), sus `dispatch_packages` y la referencia `source_order_ref` con el ID del pedido de venta.<br>• Ubicación: usar las coordenadas recibidas de Marketplace; Despachos no geocodifica direcciones. Si las coordenadas faltan o no son válidas, o el punto cae fuera de todos los polígonos del catálogo de zonas (`ES-21`), crear la orden en estado `address_review` y generar una alerta en la bandeja del Coordinador para que fije el punto manualmente en el mapa.<br>• Asignar `delivery_zone_id` por punto dentro de polígono (GeoJSON) y calcular la ventana de atención y el SLA con los parámetros de `ES-57`.<br>• Generar el código de traslado `DSP-AAAA-NNNNN` (año y secuencia de 5 dígitos con reinicio anual) con una secuencia de PostgreSQL, único e inmutable, seguro ante concurrencia.<br>• Guardar la política de evidencia recibida de Almacén en `evidence_policy` (por defecto `hand_delivery_standard`).<br>• Guardar la etiqueta informativa de pago en `payment_status_label` con el valor `paid` (todos los pedidos se asumen pagados en este proyecto); no se valida contra ningún sistema de pagos.<br>• Criterio de terminado: pruebas unitarias de zona asignada, coordenadas faltantes o fuera de cobertura (`address_review`), unicidad del código y etiqueta de pago por defecto. | Sergio | 6h | Por hacer |
| `ES-166` | `ST-49.2` | **BD: Migraciones de estados, bultos, código de traslado e índices espaciales:**<br>• Antes de modificar, revisar los `DICTIONARY.md` de `dispatches`, `dispatch_statuses` y `dispatch_types`. Todos los cambios van en `migrations/` con `up.sql`, `down.sql` y `README.md`; nunca se editan tablas en frío.<br>• Extender `dispatch_statuses` (hoy: `pending`, `in_transit`, `delivered`, `not_delivered`, `returned`) agregando `address_review`, `in_planning`, `scheduled`, `assigned`, `out_for_delivery`, `incident`, `rescheduled`, `pickup_scheduled`, `picked_up_in_transit` y `returned_to_warehouse`. Se conserva `in_transit` (lo usan el módulo de sincronización del backend, sus pruebas y los tipos de la app móvil) con el significado «en camino a una parada concreta»; documentarlo en el `DICTIONARY.md`. `dispatch_types` ya contiene `delivery`, `supplier_pickup`, `warehouse_return_pickup` y `supplier_return`: no se crea otro campo de tipo.<br>• Crear la tabla `dispatch_packages` (`dispatch_package_id`, `dispatch_id`, `external_package_id` único por despacho, `length_cm`, `width_cm`, `height_cm`, `gross_weight_kg`, `handling_conditions`) y agregar a `dispatches` las columnas `tracking_code` (único), junto con la secuencia del código `DSP-AAAA-NNNNN`, y `evidence_policy` (VARCHAR(30), NOT NULL, por defecto `hand_delivery_standard`, restringida a `hand_delivery_standard`, `contactless_delivery` y `high_value_control`).<br>• Crear índices espaciales GiST sobre las coordenadas de destino y los polígonos de `delivery_zones`, y las restricciones de integridad referencial con los catálogos.<br>• Actualizar los `DICTIONARY.md` afectados y `db-output.sql`.<br>• Criterio de terminado: migración aplicada y revertida sin errores en una base limpia y en una con datos, y `npm run build` del proyecto `bdd` sin errores. | Sergio | 5h | Por hacer |

---

#### `ES-50`: 5.3 Como Coordinador de logística, necesito agrupar pedidos por zonas, calcular rutas preliminares y validar la capacidad de carga vehicular [RF-A03]
* **Épica:** 5.0 Panel Administrativo de Coordinación y Despacho (`ES-8`)
* **Asignado (PE):** Sergio | **Story Points (Spe):** 5 | **Estado:** Por hacer (To Do)
* **Fechas:** Inicio (FI): 12 de Oct | Vencimiento (FV): 13 de Oct
* **Dependencias:** Bloquea a (B): `ES-51` | Bloqueado por (BP): `ES-21 (✅ Finalizada en Sprint 1)`, `ES-22 (✅ Finalizada en Sprint 1)`, `ES-49`

> **Como** Coordinador de Logística  
> **Quiero** que el sistema sugiera agrupaciones de pedidos por cuadrante geográfico y fecha, calcule rutas preliminares con franjas de entrega viables y valide estrictamente los límites de peso y volumen de la unidad  
> **Para** evitar la sobrecarga de las unidades vehiculares, optimizar distancias de recorrido y comunicar al cliente su franja horaria comprometida el día anterior.

* **Criterios de Aceptación (GWT):**
  * **Escenario 1 (Validación estricta de sobrecarga vehicular en peso y cubicaje - RF-A03):**  
    **Dado** un lote de pedidos cuya sumatoria de kilogramos o metros cúbicos excede la capacidad nominal del vehículo seleccionado (`capacity_kg` o `capacity_m3` del vehículo),  
    **cuando** el coordinador intenta confirmar la consolidación de la ruta,  
    **entonces** el sistema bloquea la operación emitiendo una alerta modal detallando el exceso exacto en kg y m³.
  * **Escenario 2 (Cálculo de ruta preliminar y franjas viables para comunicación al cliente):**  
    **Dado** un conjunto de pedidos seleccionados para entrega el día siguiente,  
    **cuando** se ejecuta el optimizador de ruta preliminar,  
    **entonces** el algoritmo agrupa por cercanía minimizando tiempos de viaje y asigna a cada parada una franja horaria viable (ej. `09:00 - 11:00`), dejando la orden lista para la notificación previa al cliente (`ES-40`).
  * **Escenario 3 (Barra de progreso visual de capacidad vehicular en tiempo real):**  
    **Dado** el panel de planificación de carga,  
    **cuando** el usuario añade o quita pedidos del paquete,  
    **entonces** una barra de progreso interactiva recalcula al instante los porcentajes utilizados de peso y volumen frente a la capacidad máxima del vehículo seleccionado.
  * **Escenario 4 (Respeto de las franjas ya comunicadas al recalcular la ruta):**  
    **Dado** que existen paradas con franja ya comunicada al cliente (`ES-40`, Sprint 3),  
    **cuando** una reprogramación o un cambio obliga a recalcular la ruta,  
    **entonces** el algoritmo conserva las franjas comunicadas a los demás clientes; si alguna deja de ser viable, marca la parada para que Despacho la resuelva y avise al cliente, sin modificar la franja en silencio.

| ID Jira | Subtarea | Descripción Técnica y Tareas a Realizar | Asignado (PE) | Estimación | Estado |
| :--- | :--- | :--- | :---: | :---: | :---: |
| `ES-167` | `ST-50.1` | **Backend: Validación de peso y cubicaje vehicular con los datos reales de los bultos:**<br>• Servicio que totalice `gross_weight_kg` y el volumen (largo × ancho × alto, en m³) de los `dispatch_packages` de todos los despachos de una ruta; si un despacho aún no tiene bultos, usar `estimated_weight_kg` y marcarlo como «estimado» en la respuesta.<br>• Comparar contra `capacity_kg` y `capacity_m3` del vehículo y bloquear la asignación si el peso o el volumen superan el 100 %.<br>• Respuesta descriptiva: peso total, volumen total, porcentaje de ocupación de cada uno, margen disponible y, cuando aplique, el exceso exacto en kg y m³.<br>• Criterio de terminado: pruebas unitarias con carga justo en el límite, por encima del límite y con despachos solo estimados. | Sergio | 6h | Por hacer |
| `ES-168` | `ST-50.2` | **Backend: Clusterizador espacial y ruta preliminar con OSRM (POST de previsualización), respetando franjas comunicadas:**<br>• Agrupar los despachos seleccionados por proximidad geográfica dentro de una misma zona logística y fecha.<br>• Calcular el orden de paradas y los tiempos de viaje con el servicio OSRM (`orsm/osrm-svc`), buscando cumplir las franjas comprometidas y reducir el tiempo total de recorrido.<br>• El punto de partida de la ruta es el almacén de salida (tabla `warehouses`).<br>• Asignar a cada parada una franja viable (amplia, nunca una hora exacta) a partir de los parámetros de `ES-57`.<br>• Respetar las franjas ya comunicadas a otros clientes: dejar fijas esas paradas; si alguna deja de ser viable, marcarla con `needsReview: true` en la respuesta sin cambiarla.<br>• Es solo una previsualización: no persiste cambios. Si OSRM no responde en 5 s, responder 503 con un mensaje claro.<br>• Respuesta: paradas ordenadas con su franja, distancia y duración de cada tramo, y totales de la ruta.<br>• Criterio de terminado: pruebas con OSRM simulado (stub) para ruta válida, franja comunicada que se conserva y caída de OSRM. | Joan | 5h | Por hacer |

---

#### `ES-51`: 5.4 Como Coordinador de despacho, requiero asignar operativamente la ruta consolidada definitiva a un repartidor y vehículo con control de concurrencia [RF-A04]
* **Épica:** 5.0 Panel Administrativo de Coordinación y Despacho (`ES-8`)
* **Asignado (PE):** Pardo | **Story Points (Spe):** 5 | **Estado:** Por hacer (To Do)
* **Fechas:** Inicio (FI): 14 de Oct | Vencimiento (FV): 15 de Oct
* **Dependencias:** Bloquea a (B): `ES-53`, `ES-56`, `ES-63`, `ES-69`, `ES-52`, `ES-81` | Bloqueado por (BP): `ES-50`, `ES-58`

> **Como** Coordinador de Despacho  
> **Quiero** asignar la ruta consolidada completa a una tupla de repartidor y vehículo activo tras el cierre de planificación, previniendo colisiones de asignación entre coordinadores  
> **Para** formalizar el despacho de la carga, notificar al transportista en su app móvil y asegurar que ningún recurso sufra sobreasignación o doble jornada incompatible.

* **Criterios de Aceptación (GWT):**
  * **Escenario 1 (Asignación transaccional de ruta completa consolidada - RF-A04):**  
    **Dado** un conjunto de pedidos planificados y revisados tras la hora de corte,  
    **cuando** el coordinador selecciona un repartidor disponible y un vehículo activo y confirma la asignación,  
    **entonces** el backend vincula todas las órdenes a la ruta en una única transacción, transiciona su estado a `assigned`, genera la numeración secuencial de paradas (`sequence_order`) y emite las notificaciones push al móvil.
  * **Escenario 2 (Control de concurrencia optimista ante doble asignación simultánea):**  
    **Dado** que dos coordinadores intentan asignar simultáneamente pedidos compartidos o sobreasignar al mismo repartidor en la misma franja,  
    **cuando** el servidor procesa la segunda solicitud concurrente,  
    **entonces** el backend rechaza la transacción con código HTTP 409 Conflict basado en control de versión de entidad (`@VersionColumn`), informando el conflicto al frontend y recargando la bandeja con el estado actualizado.
  * **Escenario 3 (Recomendación inteligente de tupla Chofer + Vehículo):**  
    **Dado** el drawer de asignación,  
    **cuando** el coordinador abre el diálogo,  
    **entonces** el sistema le presenta arriba los **3 recursos recomendados** (choferes activos con vehículo compatible emparejado, capacidad de carga suficiente y turno operativo vigente).

| ID Jira | Subtarea | Descripción Técnica y Tareas a Realizar | Asignado (PE) | Estimación | Estado |
| :--- | :--- | :--- | :---: | :---: | :---: |
| `ES-169` | `ST-51.1` | **Frontend Web: Drawer de asignación de ruta completa con sugerencias de flota:**<br>• Drawer con la tabla de pedidos incluidos, totalizadores de peso y volumen y barras de llenado frente a la capacidad del vehículo seleccionado, recalculadas al añadir o quitar pedidos.<br>• Selector inteligente: mostrar arriba los 3 choferes recomendados (activos, con vehículo compatible, capacidad suficiente y turno vigente) y debajo el resto.<br>• Bloquear el botón «Confirmar asignación» si se supera el 100 % de peso o de volumen, mostrando el exceso exacto en kg y m³.<br>• Mutación con retroalimentación: indicador de envío, toast de éxito y actualización de la bandeja; ante un 409, mostrar el toast de conflicto, indicar los pedidos afectados y recargar la bandeja.<br>• Criterio de terminado: pruebas de componente de la barra de capacidad, de las recomendaciones y del manejo del 409. | Pardo | 5h | Por hacer |
| `ES-170` | `ST-51.2` | **Backend: Asignación atómica de ruta (POST) con bloqueo optimista:**<br>• Recibir la lista de despachos con la `version` esperada de cada uno, `driverId` y `vehicleId`; ejecutar todo en una transacción con `QueryRunner`.<br>• Validar antes de asignar: destino completo, bultos registrados, capacidad del vehículo (`ST-50.1`), vehículo activo, repartidor disponible sin doble jornada y que cada despacho esté en `pending`, `in_planning` o `scheduled`.<br>• Control optimista con `@VersionColumn`: si la versión de algún despacho ya cambió, abortar toda la transacción y responder 409 indicando cuáles despachos entraron en conflicto.<br>• Crear o actualizar la ruta (`route_batches`) con `driver_id` y `vehicle_id`, numerar `sequence_order` y pasar los despachos a `assigned`.<br>• Tras el commit, emitir `route.assigned` por el gateway y el evento interno que dispara la notificación push al repartidor (`ES-36`).<br>• Criterio de terminado: pruebas e2e de asignación correcta, capacidad excedida, conflicto de versión entre dos solicitudes simultáneas y vehículo o repartidor no disponibles. | Sergio | 5h | Por hacer |

---

#### `ES-52`: 5.5 Como Coordinador de despacho, necesito reprogramar entregas fallidas o rechazadas definiendo nueva fecha, motivo tipificado y reactivación automática [RF-A05]
* **Épica:** 5.0 Panel Administrativo de Coordinación y Despacho (`ES-8`)
* **Asignado (PE):** Pardo | **Story Points (Spe):** 3 | **Estado:** Por hacer (To Do)
* **Fechas:** Inicio (FI): 17 de Oct | Vencimiento (FV): 19 de Oct
* **Dependencias:** Bloquea a (B): Ninguna | Bloqueado por (BP): `ES-25 (✅ Finalizada en Sprint 1)`, `ES-51`

> **Como** Coordinador de Despacho  
> **Quiero** gestionar los pedidos que sufrieron incidencias o entregas no concretadas, asignándoles una nueva fecha pactada y motivo justificado, con control de reintentos máximos y reactivación automática en la fecha indicada  
> **Para** asegurar una segunda oportunidad de entrega sin perder la auditoría de intentos fallidos ni saturar la bandeja antes de tiempo.

* **Criterios de Aceptación (GWT):**
  * **Escenario 1 (Reprogramación con fecha pactada y causal justificada - RF-A05):**  
    **Dado** un pedido en estado `incident` o `not_delivered`,  
    **cuando** el coordinador define una nueva fecha de entrega pactada y selecciona la causal desde el catálogo (`ES-25`),  
    **entonces** el pedido transiciona a estado `rescheduled`, registra el intento en `dispatch_reschedules` (el número de intento es la cantidad de reprogramaciones registradas), se desvincula de la ruta actual y se oculta de la lista de pendientes del día.
  * **Escenario 2 (Reactivación automática en la fecha pactada de entrega):**  
    **Dado** un pedido en estado `rescheduled` cuya fecha pactada coincide con el día de la jornada activa,  
    **cuando** el coordinador consulta la bandeja de despachos del día,  
    **entonces** el sistema incluye automáticamente el pedido en la cola de asignación con un badge ámbar destacado: `Reprogramado (Reintento 2/3)` para que sea planificado en la nueva ruta.
  * **Escenario 3 (Control de umbral máximo de reintentos configurado en ES-57):**  
    **Dado** un pedido que alcanza el límite máximo de reintentos parametrizado en el sistema (ej. 3 intentos fallidos),  
    **cuando** se intenta una nueva reprogramación,  
    **entonces** el sistema bloquea la acción emitiendo una alerta crítica de "Límite de reintentos alcanzado" y fuerza la derivación del pedido a devolución formal hacia Almacén (`RMA`).

| ID Jira | Subtarea | Descripción Técnica y Tareas a Realizar | Asignado (PE) | Estimación | Estado |
| :--- | :--- | :--- | :---: | :---: | :---: |
| `ES-171` | `ST-52.1` | **Frontend Web: Modal de reprogramación con historial de intentos y evidencia:**<br>• Mostrar el historial del pedido: motivos de no entrega anteriores, notas del conductor y fotos de fachada (cargadas con URL de acceso temporal).<br>• Formulario: nueva fecha pactada (`DatePicker`, no anterior a hoy), turno o franja, motivo del catálogo `reschedule_reasons` e instrucciones especiales opcionales.<br>• Badge «Intento n de N», con N tomado del parámetro `max_reschedule_attempts`; alerta preventiva si es el último intento permitido y, si ya se alcanzó el límite, bloquear el formulario y mostrar «Límite de reintentos alcanzado» con la acción de derivar a devolución hacia Almacén.<br>• Manejar los errores 400 y 409 del backend con mensajes claros.<br>• Criterio de terminado: pruebas de componente del formulario, del último intento y del límite alcanzado. | Pardo | 4h | Por hacer |
| `ES-172` | `ST-52.2` | **Backend: Reprogramación de despacho (POST) con reglas de umbral y auditoría:**<br>• Recibir `RescheduleDispatchDto`: nueva fecha pactada (no anterior a hoy), turno o franja, código del motivo del catálogo `reschedule_reasons` (`ES-25`) e instrucciones especiales opcionales. Solo se admiten despachos en `incident` o `not_delivered`; en otro estado responder 409.<br>• Calcular el número de intento como la cantidad de registros previos en `dispatch_reschedules` más uno; si supera `max_reschedule_attempts` (clave de `settings`, `ES-57`), responder 400 indicando que debe derivarse a devolución formal hacia Almacén.<br>• En una transacción: registrar el cambio en `dispatch_reschedules` (ventana anterior y nueva, ruta anterior y usuario), desvincular el despacho de su ruta, pasarlo a `rescheduled` y registrar el evento en `dispatch_events`.<br>• Tras el commit, emitir `dispatch.status_changed` y el evento interno que el módulo de notificaciones (`ES-40`, Sprint 3) usará para avisar al cliente su nuevo compromiso.<br>• La reactivación no requiere proceso aparte: `ST-48.2` incluye los `rescheduled` cuya fecha pactada es hoy.<br>• Criterio de terminado: pruebas e2e de reprogramación válida, estado inválido, límite de intentos excedido y fecha pasada. | Joan | 4h | Por hacer |

---

#### `ES-53`: 5.6 Como Coordinador de despacho, requiero monitorear el avance de las entregas en un tablero Kanban interactivo sincronizado en tiempo real [RF-A06]
* **Épica:** 5.0 Panel Administrativo de Coordinación y Despacho (`ES-8`)
* **Asignado (PE):** Pardo | **Story Points (Spe):** 5 | **Estado:** Por hacer (To Do)
* **Fechas:** Inicio (FI): 16 de Oct | Vencimiento (FV): 17 de Oct
* **Dependencias:** Bloquea a (B): `ES-54` | Bloqueado por (BP): `ES-51`

> **Como** Coordinador de Despachos  
> **Quiero** monitorear en un tablero visual tipo Kanban las órdenes distribuidas en sus etapas operativas completas (`Pendiente`, `En Planificación`, `Programado`, `Asignado`, `En Ruta`, `En Camino`, `Entregado`, `Incidencia`, `Reprogramado`) con sincronización reactiva Socket.io y TanStack Query  
> **Para** obtener una panorámica inmediata del avance de la jornada, prevenir colisiones entre coordinadores y reaccionar al instante ante contingencias en ruta.

* **Criterios de Aceptación (GWT):**
  * **Escenario 1 (Distribución estructurada en columnas del flujo operativo completo - RF-A06):**  
    **Dado** el tablero de control de despacho,  
    **cuando** se cargan las órdenes del día,  
    **entonces** se organizan en las columnas normalizadas: `Pendiente`, `En Planificación`, `Programado`, `Asignado`, `En Ruta`, `En Camino`, `Entregado`, `Incidencia` (que agrupa `incident` y `not_delivered`) y `Reprogramado`, con contadores totales de órdenes y volumen por columna y un filtro «Logística inversa» que muestra los despachos de recojo y devolución (`pickup_scheduled`, `picked_up_in_transit`, `returned_to_warehouse`, `returned`).
  * **Escenario 2 (Interacción guiada y protección contra asignación arbitraria por arrastre):**  
    **Dado** un pedido en columna `Pendiente` o `En Planificación`,  
    **cuando** el coordinador arrastra la tarjeta hacia la columna `Asignado`,  
    **entonces** la interfaz no ejecuta una asignación a ciegas sino que abre el drawer de asignación de ruta (`ES-51`) validando capacidad y chofer antes de confirmar la transición.
  * **Escenario 3 (Actualización reactiva híbrida mediante Socket.io y TanStack Query):**  
    **Dado** que un repartidor actualiza un estado a `in_transit` o `delivered` en su app móvil,  
    **cuando** el backend procesa el cambio,  
    **entonces** emite un evento por Socket.io (`dispatch.status_changed`), el frontend invalida la consulta de TanStack Query (`invalidateQueries`) y la tarjeta se desplaza suavemente a su nueva columna sin requerir recargar la página web, respaldado por un polling pasivo de seguridad cada 15 segundos.

| ID Jira | Subtarea | Descripción Técnica y Tareas a Realizar | Asignado (PE) | Estimación | Estado |
| :--- | :--- | :--- | :---: | :---: | :---: |
| `ES-173` | `ST-53.1` | **Frontend Web: Tablero Kanban interactivo con 9 columnas, arrastre guiado y filtro de logística inversa:**<br>• Construir el tablero en React 19 con `@dnd-kit/core` y Tailwind CSS v4, con las columnas Pendiente (`pending`), En planificación (`in_planning`), Programado (`scheduled`), Asignado (`assigned`), En ruta (`out_for_delivery`), En camino (`in_transit`), Entregado (`delivered`), Incidencia (`incident` y `not_delivered`) y Reprogramado (`rescheduled`); scroll vertical independiente y contadores de órdenes y volumen en la cabecera de cada columna.<br>• Tarjeta: código de orden, transportista y vehículo asignados, zona logística, franja comprometida, badge de prioridad y, en Reprogramado, el badge «Reintento n/N».<br>• Barra de filtros: zona, conductor, tipo de despacho y un conmutador «Logística inversa» que reemplaza las columnas por las de recojo y devolución (`pickup_scheduled`, `picked_up_in_transit`, `returned_to_warehouse`, `returned`).<br>• Arrastre guiado: en este sprint solo se permite arrastrar hacia «Asignado», lo que abre el drawer de asignación (`ES-51`) sin cambiar el estado hasta confirmar; los demás arrastres quedan deshabilitados porque los estados en ruta los controla el repartidor.<br>• Criterio de terminado: pruebas de componente del tablero (columnas y contadores, arrastre permitido y arrastre bloqueado). | Pardo | 6h | Por hacer |
| `ES-174` | `ST-53.2` | **Backend/Frontend: Sincronización en vivo con Socket.io y TanStack Query:**<br>• Backend: implementar `DispatchEventsGateway` (NestJS WebSocket) como único punto de salida de eventos en tiempo real. Autenticar el handshake con el JWT, rechazar conexiones sin token válido y unir cada conexión a salas por rol (`coordinators`, `supervisors`), por zona (`zone:{id}`) y, para repartidores, por usuario (`driver:{id}`).<br>• Escuchar los eventos internos (publicados con `@nestjs/event-emitter` después de cada commit) y reemitir `dispatch.status_changed`, `dispatch.incident_reported`, `dispatch.evidence_registered`, `dispatch.change_requires_attention` y `route.assigned` con los payloads del catálogo de `docs/interoperabilidad-y-flujos.md` (sección 3).<br>• Frontend: hook `useDispatchEvents` que conecte con el cliente Socket.io, escuche esos eventos y ejecute `queryClient.invalidateQueries({ queryKey: ['dispatches-kanban'] })`, reconectando con backoff.<br>• Respaldo: `refetchInterval: 15000` activo solo mientras el socket esté desconectado.<br>• Criterio de terminado: prueba de integración que cambia un estado y verifica que el evento llega a un cliente conectado y no llega a uno sin token. | Sergio | 4h | Por hacer |

---

#### `ES-54`: 5.7 Como Coordinador de logística, necesito generar reportes de cumplimiento de entrega y tiempos frente al SLA [RF-A07]
* **Épica:** 5.0 Panel Administrativo de Coordinación y Despacho (`ES-8`)
* **Asignado (PE):** Pardo | **Story Points (Spe):** 3 | **Estado:** Por hacer (To Do)
* **Fechas:** Inicio (FI): 18 de Oct | Vencimiento (FV): 19 de Oct
* **Dependencias:** Bloquea a (B): Ninguna | Bloqueado por (BP): `ES-53`

> **Como** Coordinador de Logística  
> **Quiero** generar y exportar reportes de efectividad de entrega (% a tiempo frente a franjas comprometidas, demoras promedio, incidencias por zona)  
> **Para** evaluar el cumplimiento de los contratos de nivel de servicio (SLA) y justificar decisiones operativas de mejora continua.

* **Criterios de Aceptación (GWT):**
  * **Escenario 1 (Generación de gráficos y métricas de SLA - RF-A07):**  
    **Dado** un período seleccionado en el generador de reportes,  
    **cuando** se procesan los datos,  
    **entonces** el portal despliega el porcentaje global de entregas a tiempo frente al umbral pactado y permite exportar el consolidado.
  * **Escenario 2 (Desglose comparativo por zona logística y tipo de servicio):**  
    **Dado** el panel de métricas de SLA,  
    **cuando** se selecciona la vista detallada por zonas,  
    **entonces** se exhibe una tabla comparativa con los tiempos promedio de ciclo (despacho a entrega), tasa de cumplimiento de entregas Express vs Estándar y zonas con mayor demora.

| ID Jira | Subtarea | Descripción Técnica y Tareas a Realizar | Asignado (PE) | Estimación | Estado |
| :--- | :--- | :--- | :---: | :---: | :---: |
| `ES-175` | `ST-54.1` | **Frontend Web: Módulo de reportes de cumplimiento SLA con filtros y exportación:**<br>• Pantalla con selector de período y de agrupación (zona, nivel de servicio o conductor) que consulta `ST-54.2`.<br>• Gráficos: porcentaje global de entregas a tiempo frente al umbral pactado, barras comparativas por grupo con tiempo promedio de ciclo y demora promedio, y tabla de detalle con las zonas de mayor demora ordenadas.<br>• Exportación en el cliente a CSV y Excel del resultado mostrado, con los filtros aplicados y un nombre de archivo que incluya el período.<br>• Estados de carga, vacío («Sin datos para el período seleccionado») y error con reintento.<br>• Criterio de terminado: pruebas de componente de los filtros, del estado vacío y del contenido de las columnas del archivo exportado. | Pardo | 4h | Por hacer |
| `ES-176` | `ST-54.2` | **Backend: Reporte de cumplimiento SLA (GET) con agregaciones y caché Redis:**<br>• Parámetros: `from` y `to` (validar que `from` no sea posterior a `to`) y agrupación `zone` \| `service_level` \| `driver`.<br>• Con SQL agregado en PostgreSQL, calcular por grupo: total de despachos cerrados, porcentaje entregado dentro de la franja comprometida, demora promedio de los entregados fuera de franja y tiempo promedio de ciclo (de `out_for_delivery` a `delivered`).<br>• Cachear la respuesta en Redis con una clave derivada de los parámetros y TTL de 10 minutos; si Redis falla, consultar directo a PostgreSQL.<br>• La exportación a CSV/Excel la hace el cliente (`ST-54.1`).<br>• Criterio de terminado: pruebas con datos de ejemplo para cada agrupación, rango inválido y caché. | Sergio | 3h | Por hacer |

---

#### `ES-55`: 5.8 Como Coordinador de logística, requiero consultar el historial de despachos aplicando filtros por repartidor, vehículo y fechas [RF-A08]
* **Épica:** 5.0 Panel Administrativo de Coordinación y Despacho (`ES-8`)
* **Asignado (PE):** Pardo | **Story Points (Spe):** 3 | **Estado:** Por hacer (To Do)
* **Fechas:** Inicio (FI): 18 de Oct | Vencimiento (FV): 19 de Oct
* **Dependencias:** Bloquea a (B): Ninguna | Bloqueado por (BP): `ES-56`, `ES-51`

> **Como** Coordinador de Logística  
> **Quiero** realizar búsquedas avanzadas en el histórico de despachos aplicando múltiples filtros simultáneos (repartidor, vehículo, fechas, estado)  
> **Para** auditar pedidos anteriores y resolver dudas operativas sin fricción.

* **Criterios de Aceptación (GWT):**
  * **Escenario 1 (Filtrado multifactorial del historial - RF-A08):**  
    **Dado** el módulo de historial web,  
    **cuando** el coordinador filtra por un repartidor específico y un rango de fechas,  
    **entonces** la tabla retorna los despachos correspondientes con paginación server-side y tiempos de respuesta menores a 1 segundo.
  * **Escenario 2 (Exportación de registros históricos filtrados):**  
    **Dado** un conjunto de resultados filtrados por rango mensual en el historial,  
    **cuando** el usuario presiona "Exportar Historial",  
    **entonces** el sistema descarga un archivo estructurado en formato CSV con el detalle de las órdenes, horas de partida, entrega y repartidores participantes.

| ID Jira | Subtarea | Descripción Técnica y Tareas a Realizar | Asignado (PE) | Estimación | Estado |
| :--- | :--- | :--- | :---: | :---: | :---: |
| `ES-177` | `ST-55.1` | **Frontend Web: Tabla avanzada de historial con filtros combinados y paginación:**<br>• Tabla de despachos con paginación en el servidor que consume `ST-55.2`. Filtros combinables por rango de fechas, repartidor, placa del vehículo, estado final y búsqueda por código de guía, aplicados juntos y conservados en la URL.<br>• Columnas: código de traslado, cliente, repartidor, vehículo, hora de partida, hora de entrega y estado final.<br>• Botón «Exportar historial» que descarga el CSV generado por el backend (`ST-55.2`) con los mismos filtros activos.<br>• Enlace de cada fila a la ficha unificada 360° (`ES-56`).<br>• Estados de carga, vacío y error; respuesta percibida menor a 1 segundo.<br>• Criterio de terminado: pruebas de combinación de filtros, paginación, exportación y navegación a la ficha. | Pardo | 4h | Por hacer |
| `ES-178` | `ST-55.2` | **Backend: Consulta paginada y optimizada del histórico de despachos (GET) con exportación CSV:**<br>• QueryBuilder para búsquedas compuestas con los filtros: rango de fechas (`from`, `to`), repartidor, vehículo (placa), estado final y texto sobre el código de guía; solo despachos en estados terminales (`delivered`, `not_delivered`, `returned`, `returned_to_warehouse`).<br>• Parámetros `page`, `limit` (máximo 100), `sortBy` y `order`; respuesta `{ data, total, page, limit }`.<br>• Índices sobre la fecha de cierre y la ruta (repartidor y vehículo), verificados con `EXPLAIN`, de modo que la consulta con filtros combinados responda en menos de 1 s con 100 000 despachos de prueba.<br>• Exportación: el mismo servicio genera un CSV (UTF-8) con las columnas del detalle de órdenes (código, cliente, repartidor, vehículo, hora de partida, hora de entrega y estado), con los mismos filtros, sin paginar y enviado en streaming.<br>• Acceso solo para Coordinador y Supervisor.<br>• Criterio de terminado: pruebas e2e de filtros combinados, paginación, exportación y rol no autorizado (403). | Joan | 3h | Por hacer |

---

#### `ES-56`: 5.9 Como Coordinador de despacho, necesito visualizar una ficha unificada 360° con la trazabilidad y evidencias completas de la orden [RF-A09]
* **Épica:** 5.0 Panel Administrativo de Coordinación y Despacho (`ES-8`)
* **Asignado (PE):** Pardo | **Story Points (Spe):** 4 | **Estado:** Por hacer (To Do)
* **Fechas:** Inicio (FI): 16 de Oct | Vencimiento (FV): 17 de Oct
* **Dependencias:** Bloquea a (B): `ES-55` | Bloqueado por (BP): `ES-51`

> **Como** Coordinador o Supervisor  
> **Quiero** acceder a una vista unificada 360° con la trazabilidad completa, firmas, fotos de evidencia, tiempos e incidencias de un despacho  
> **Para** responder reclamos de clientes o auditar la calidad del servicio prestado con información fidedigna.

* **Criterios de Aceptación (GWT):**
  * **Escenario 1 (Despliegue unificado de trazabilidad y eventos - RF-A09):**  
    **Dado** un despacho finalizado o en curso,  
    **cuando** el coordinador abre su ficha unificada,  
    **entonces** se visualiza la línea de tiempo vertical con horas exactas, coordenadas GPS de entrega, receptor y operador a cargo.
  * **Escenario 2 (Inspección de evidencias fotográficas con zoom y comprobante de firma):**  
    **Dado** que el coordinador accede a la pestaña "Evidencias Digitales",  
    **cuando** hace clic en la miniatura de la foto del paquete o de la firma en pantalla,  
    **entonces** se abre un visor modal en alta resolución con metadatos de captura (fecha, hora y geolocalización de entrega).

| ID Jira | Subtarea | Descripción Técnica y Tareas a Realizar | Asignado (PE) | Estimación | Estado |
| :--- | :--- | :--- | :---: | :---: | :---: |
| `ES-179` | `ST-56.1` | **Frontend Web: Ficha 360° del despacho con línea de tiempo interactiva:**<br>• Drawer o página con pestañas: General (datos, destino, bultos, repartidor, vehículo y etiqueta de pago), Línea de tiempo, Evidencias digitales e Incidencias (incluye las reprogramaciones).<br>• Línea de tiempo vertical con la hora exacta, el estado, las coordenadas GPS de cada evento, el operador a cargo y, al final, el receptor.<br>• Visor modal de firmas y fotografías en alta resolución, con zoom y metadatos de captura (fecha, hora y geolocalización). Las URL de las evidencias son temporales: solicitar una nueva si el visor encuentra una vencida.<br>• Estados de carga y error, y mensajes claros cuando no existan evidencias o incidencias.<br>• Criterio de terminado: pruebas de componente de cada pestaña, del visor y de la renovación de una URL vencida. | Pardo | 5h | Por hacer |
| `ES-180` | `ST-56.2` | **Backend: Ficha 360° del despacho (GET) con línea de tiempo y evidencias:**<br>• Devolver en una sola respuesta: datos del despacho y destino, bultos, conductor y vehículo de la ruta, línea de tiempo ordenada (`dispatch_events` con hora, estado, coordenadas y operador), incidencias (`dispatch_incidents`), reprogramaciones (`dispatch_reschedules`) y evidencias (`delivery_evidences`).<br>• Las evidencias se devuelven con una URL de acceso de corta duración (URL firmada o ruta protegida que exige JWT) generada por el puerto de almacenamiento, nunca con la ruta física del archivo.<br>• Autorizar solo a Coordinador y Supervisor (`@Roles`); 404 si el despacho no existe.<br>• Una sola consulta con relaciones cargadas (sin N+1) y tiempo de respuesta objetivo menor a 1 s con 100 eventos.<br>• Criterio de terminado: pruebas e2e de despacho completo, sin evidencias y con rol no autorizado (403). | Joan | 4h | Por hacer |

---

#### `ES-57`: 5.10 Como Coordinador de logística, requiero configurar los parámetros de ventanas de entrega, tiempos de espera y umbral de reintentos en servicio [RF-A10]
* **Épica:** 5.0 Panel Administrativo de Coordinación y Despacho (`ES-8`)
* **Asignado (PE):** Pardo | **Story Points (Spe):** 3 | **Estado:** Por hacer (To Do)
* **Fechas:** Inicio (FI): 05 de Oct | Vencimiento (FV): 07 de Oct
* **Dependencias:** Bloquea a (B): `ES-49` | Bloqueado por (BP): `ES-14 (✅ Finalizada en Sprint 1)`

> **Como** Coordinador de Logística  
> **Quiero** parametrizar las ventanas horarias de entrega, tiempos máximos de espera en puerta, márgenes de tolerancia, umbral máximo de intentos de reprogramación y hora límite de cierre de rutas  
> **Para** sincronizar las reglas de negocio con las políticas del servicio y el cálculo de SLAs.

* **Criterios de Aceptación (GWT):**
  * **Escenario 1 (Actualización de parámetros operativos en tiempo real - RF-A10):**  
    **Dado** el formulario de configuración de parámetros,  
    **cuando** el coordinador modifica el tiempo máximo de espera a 10 minutos y guarda cambios,  
    **entonces** el backend almacena los valores y los propaga inmediatamente a las validaciones de las apps móviles.
  * **Escenario 2 (Validación estricta de coherencia en ventanas horarias y umbral de intentos):**  
    **Dado** el panel de configuración de parámetros,  
    **cuando** el usuario intenta guardar una ventana horaria inconsistente o un umbral de reintentos menor a 1,  
    **entonces** el formulario bloquea el guardado emitiendo una alerta de validación visual con React Hook Form.

| ID Jira | Subtarea | Descripción Técnica y Tareas a Realizar | Asignado (PE) | Estimación | Estado |
| :--- | :--- | :--- | :---: | :---: | :---: |
| `ES-181` | `ST-57.1` | **Frontend Web: Formulario de parametrización operativa con React Hook Form y Zod:**<br>• Campos: ventana horaria de la mañana, de la tarde y express (inicio menor que fin), tiempo máximo de espera en puerta (minutos, entero ≥ 1), margen de tolerancia (minutos ≥ 0), umbral máximo de reprogramaciones `max_reschedule_attempts` (entero ≥ 1) y hora límite de cierre de rutas (`hh:mm`).<br>• Validación con Zod: ventanas coherentes y los mínimos indicados; mostrar el error bajo cada campo y bloquear el guardado mientras existan.<br>• Cargar los valores actuales al abrir, mostrar toast de confirmación al guardar y mensaje de error con reintento si la petición falla.<br>• Criterio de terminado: pruebas de componente de validación (ventana inconsistente, umbral menor a 1, hora límite vacía) y de guardado exitoso. | Pardo | 4h | Por hacer |
| `ES-182` | `ST-57.2` | **Backend e infraestructura: Parámetros operativos (GET/PUT) y servicio Redis de caché:**<br>• Agregar el servicio Redis al `docker-compose` y un módulo de caché en `src/plugins/cache/` (puerto `CacheService` con adaptador Redis y adaptador en memoria para pruebas).<br>• Endpoints de consulta (GET) y actualización (PUT) de los parámetros operativos con DTO validado. Se guardan en la tabla `settings` (clave-valor) las claves de ventanas (mañana, tarde, express), espera máxima, tolerancia, `max_reschedule_attempts` y `route_cutoff_time`. Solo el Coordinador puede modificarlos.<br>• Cachear la lectura en Redis y reemplazar la caché en cada actualización; si Redis no responde, leer de PostgreSQL sin fallar.<br>• Al guardar, incrementar una versión de parámetros para que las apps móviles los recarguen en su próxima sincronización.<br>• Criterio de terminado: pruebas e2e de lectura, actualización válida, valores inválidos (400) y rol no autorizado (403). | Sergio | 5h | Por hacer |

---

#### `ES-58`: 5.11 Como Coordinador de despacho, necesito marcar y filtrar pedidos según su nivel de prioridad urgente o normal [RF-A11]
* **Épica:** 5.0 Panel Administrativo de Coordinación y Despacho (`ES-8`)
* **Asignado (PE):** Sergio | **Story Points (Spe):** 3 | **Estado:** Por hacer (To Do)
* **Fechas:** Inicio (FI): 12 de Oct | Vencimiento (FV): 13 de Oct
* **Dependencias:** Bloquea a (B): `ES-51` | Bloqueado por (BP): `ES-49`

> **Como** Coordinador de Logística  
> **Quiero** visualizar etiquetas cromáticas de prioridad (`urgent`, `normal`) y aplicar filtros multifactoriales en el panel web  
> **Para** priorizar los paquetes más críticos y organizar el armado de rutas sin demoras.

* **Criterios de Aceptación (GWT):**
  * **Escenario 1 (Filtrado de órdenes urgentes en panel administrativo - RF-A11):**  
    **Dado** el listado de pedidos pendientes,  
    **cuando** el usuario activa el filtro "Solo Urgentes",  
    **entonces** la vista muestra únicamente pedidos marcados como urgentes ordenados por vencimiento de SLA.
  * **Escenario 2 (Marcación manual de urgencia operativa):**  
    **Dado** un pedido estándar que por contingencia requiere prioridad inmediata,  
    **cuando** el coordinador hace clic sobre la insignia de prioridad y selecciona "URGENTE",  
    **entonces** el sistema actualiza la prioridad en base de datos, reordena la lista al tope de atención y emite toast de confirmación.

| ID Jira | Subtarea | Descripción Técnica y Tareas a Realizar | Asignado (PE) | Estimación | Estado |
| :--- | :--- | :--- | :---: | :---: | :---: |
| `ES-183` | `ST-58.1` | **Frontend Web: Insignias de prioridad y filtros combinados:**<br>• Badges de prioridad reutilizables en la bandeja y en el tablero Kanban: rojo para `urgent` («Urgente») y gris para `normal` («Normal»).<br>• Cambio rápido: al hacer clic en la insignia de un pedido, abrir un selector y confirmar el cambio con `ST-58.2`; actualizar la lista al instante, reordenar con los urgentes al tope y mostrar un toast. Si el servidor rechaza el cambio (409), revertir y avisar.<br>• Barra de filtros dinámicos (zona, prioridad, turno) con el interruptor «Solo urgentes» ordenado por vencimiento de SLA y filtros guardados en la URL.<br>• Criterio de terminado: pruebas de componente del cambio de prioridad (éxito y rechazo) y de la combinación de filtros. | Sergio | 4h | Por hacer |
| `ES-184` | `ST-58.2` | **Backend: Prioridad del despacho (columna existente) con cambio manual y ordenamiento:**<br>• Usar la columna `priority` que ya existe en `dispatches` (valores `normal` y `urgent`; no se agrega `low`).<br>• Endpoint (PATCH) para cambiar la prioridad de un despacho en `pending`, `in_planning` o `scheduled`; en otro estado responder 409. Registrar el cambio en `dispatch_events` con el usuario que lo hizo.<br>• En la consulta de la bandeja (`ST-48.2`): ordenar por defecto urgentes primero y luego por vencimiento de SLA, y permitir filtrar solo urgentes.<br>• Criterio de terminado: pruebas e2e del cambio manual, estado no permitido (409) y orden por defecto. | Joan | 3h | Por hacer |

---

#### `ES-81`: 5.12 Como Coordinador de despacho, requiero registrar y programar órdenes de recojo domiciliario para devoluciones o garantías [RF-A39]
* **Épica:** 5.0 Panel Administrativo de Coordinación y Despacho (`ES-8`)
* **Asignado (PE):** Pardo | **Story Points (Spe):** 3 | **Estado:** Por hacer (To Do)
* **Fechas:** Inicio (FI): 16 de Oct | Vencimiento (FV): 19 de Oct
* **Dependencias:** Bloquea a (B): Ninguna | Bloqueado por (BP): `ES-51`

> **Como** Coordinador de Logística  
> **Quiero** registrar y programar órdenes de recojo en domicilio para devoluciones o garantías aprobadas  
> **Para** gestionar la logística inversa desde el cliente hacia el almacén central con los mismos estándares de asignación y trazabilidad.

* **Criterios de Aceptación (GWT):**
  * **Escenario 1 (Creación de orden de recojo domiciliario - RF-A39):**  
    **Dado** un reclamo o devolución autorizada,  
    **cuando** el coordinador registra la orden de recojo con dirección del cliente, fecha y descripción de mercadería,  
    **entonces** el sistema crea la orden en estado `pickup_scheduled` lista para asignarse a la ruta de un repartidor.
  * **Escenario 2 (Validación de motivo de devolución y cancelación previa a ruta):**  
    **Dado** un pedido de recojo programado pero no despachado aún,  
    **cuando** el cliente desiste de la garantía o cancela la solicitud,  
    **entonces** el coordinador puede anular la orden registrando la causal respectiva liberando la programación de la flota.

| ID Jira | Subtarea | Descripción Técnica y Tareas a Realizar | Asignado (PE) | Estimación | Estado |
| :--- | :--- | :--- | :---: | :---: | :---: |
| `ES-185` | `ST-81.1` | **Frontend Web: Formulario de programación de órdenes de retiro domiciliario:**<br>• Modal con los campos: cliente, teléfono, dirección con ubicación en el mapa (coordenadas), fecha y franja del recojo, descripción de la mercadería, motivo de devolución del catálogo (`ES-25`) y referencia de la devolución aprobada (RMA).<br>• Validación con React Hook Form y Zod: campos obligatorios, teléfono con formato válido y fecha no anterior a hoy; mostrar el error bajo cada campo.<br>• Confirmación visual al crear (toast) y aparición de la orden en `pickup_scheduled`, lista para asignar.<br>• Anulación: desde el detalle de una orden en `pickup_scheduled`, acción «Anular recojo» con un modal de confirmación que exige la causal; deshabilitada en otros estados.<br>• Criterio de terminado: pruebas de componente de las validaciones, de la creación exitosa y de la anulación permitida y bloqueada. | Pardo | 4h | Por hacer |
| `ES-186` | `ST-81.2` | **Backend: Creación y anulación de órdenes de recojo domiciliario:**<br>• Endpoint de creación (POST) con `CreatePickupOrderDto`: cliente, teléfono, dirección con coordenadas, fecha y franja, descripción de la mercadería, código del motivo de devolución (catálogo de `ES-25`) y la referencia de la devolución aprobada (RMA) de Almacén. Validar que el motivo esté activo.<br>• Crear el despacho con tipo `warehouse_return_pickup` y estado `pickup_scheduled`, listo para asignarse a una ruta.<br>• Endpoint de anulación (PATCH): permitido solo mientras el despacho esté en `pickup_scheduled`; registra la causal y libera la programación. En otro estado responder 409.<br>• Criterio de terminado: pruebas e2e de creación, motivo inválido, anulación permitida y anulación bloqueada. | Sergio | 4h | Por hacer |

---

### ÉPICA 8.0 — Interoperabilidad con el ERP y Sistemas Externos (`ES-11`)

> **Nota de alcance:** la integración con el ERP es **asíncrona y simulada**. Los demás módulos (Marketplace y Ventas, Inventarios y Almacén, Compras y Proveedores) se representan con publicadores simulados sobre el emulador de Google Cloud Pub/Sub; los contratos (tópicos, esquemas JSON y reglas) son los de [`interoperabilidad-y-flujos.md`](../interoperabilidad-y-flujos.md).

#### `ES-72`: 8.1 Como Desarrollador de integraciones, requiero consumir los datos de entrega de Marketplace y los avisos de pedidos empacados de Almacén para consolidar órdenes de entrega [ERP-01]
* **Épica:** 8.0 Interoperabilidad con el ERP y Sistemas Externos (`ES-11`)
* **Asignado (PE):** Sergio | **Story Points (Spe):** 5 | **Estado:** Por hacer (To Do)
* **Fechas:** Inicio (FI): 05 de Oct | Vencimiento (FV): 08 de Oct
* **Dependencias:** Bloquea a (B): `ES-73`, `ES-49` | Bloqueado por (BP): `ES-14 (✅ Finalizada en Sprint 1)`

> **Como** Desarrollador de Integraciones Backend  
> **Quiero** consumir de forma asíncrona los mensajes de pedidos de venta confirmados (Marketplace y Ventas) y de pedidos empacados listos para retirar (Inventarios y Almacén), uniéndolos por el ID del pedido de venta  
> **Para** alimentar automáticamente el flujo de despachos con pedidos que ya tienen destino y bultos reales, sin digitación manual.

* **Criterios de Aceptación (GWT):**
  * **Escenario 1 (Consolidación de pedido de venta y bultos - ERP-01):**  
    **Dado** que Despachos recibió `marketplace.sales_order.confirmed` e `inventory.shipment.ready` con el mismo `salesOrderId`,  
    **cuando** el consumidor procesa el segundo de los dos mensajes,  
    **entonces** entrega la información consolidada al generador de órdenes (`ES-49`) conservando el ID del pedido de venta y el de cada bulto.
  * **Escenario 2 (Llegada en desorden o incompleta):**  
    **Dado** que solo llegó uno de los dos mensajes,  
    **cuando** se procesa,  
    **entonces** el dato queda en espera sin crear el despacho ni mostrarlo en la bandeja, y se consolida cuando llega el complementario.
  * **Escenario 3 (Rechazo por esquema inválido):**  
    **Dado** un mensaje que no cumple su JSON Schema (campo obligatorio faltante, peso o medida menor o igual a cero),  
    **cuando** se procesa,  
    **entonces** no se crea ni modifica ningún registro y el mensaje pasa al tópico dead-letter con el detalle de los errores.
  * **Escenario 4 (Idempotencia y control de versiones):**  
    **Dado** un mensaje con un `messageId` ya procesado o con un `recordVersion` menor o igual al almacenado,  
    **cuando** llega por reintento o desorden,  
    **entonces** se ignora sin generar duplicados ni retroceder datos.
  * **Escenario 5 (Cambios y cancelaciones posteriores):**  
    **Dado** un pedido en espera o en estado `pending`,  
    **cuando** llega un mensaje de actualización o cancelación de Marketplace o de Almacén,  
    **entonces** el registro se actualiza o se anula; si el pedido ya fue asignado a una ruta, no se modifica en silencio y se alerta al coordinador.
  * **Escenario 6 (Recuperación de mensajes perdidos):**  
    **Dado** un pedido que lleva más tiempo del permitido con solo uno de los dos mensajes,  
    **cuando** el detector periódico lo encuentra,  
    **entonces** Despachos solicita al origen que falta el registro completo (hasta 3 veces) y, si no llega, alerta al coordinador para que lo gestione.

| ID Jira | Subtarea | Descripción Técnica y Tareas a Realizar | Asignado (PE) | Estimación | Estado |
| :--- | :--- | :--- | :---: | :---: | :---: |
| `ES-201` | `ST-72.0` | **Backend/Infra: Infraestructura de mensajería asíncrona simulada (Pub/Sub):**<br>• Agregar el emulador de Google Cloud Pub/Sub al `docker-compose` y un script de arranque que cree los tópicos del catálogo de `docs/interoperabilidad-y-flujos.md` (sección 1.4), sus suscripciones y un tópico dead-letter por cada suscripción de entrada.<br>• Crear el puerto `MessageBus` en `src/plugins/messaging/` con las operaciones `publish(topic, envelope)` y `subscribe(topic, handler)`, un adaptador Pub/Sub (configurado por variables de entorno y apuntando al emulador en desarrollo) y un adaptador en memoria para pruebas.<br>• Implementar el sobre estándar (`messageId` UUID v7, `type`, `schemaVersion`, `source`, `occurredAt`, `data`) y un validador JSON Schema (por ejemplo Ajv) que rechace los mensajes inválidos antes de llegar al manejador; los rechazados se publican en el dead-letter con la lista de errores.<br>• Implementar el publicador de la *outbox* sobre la tabla `outbound_messages` creada en `ST-72.3`: publicar con reintentos y backoff exponencial (1 s, 2 s, 4 s), marcar el mensaje como enviado y registrar una alerta tras 3 fallos definitivos.<br>• Criterio de terminado: `docker compose up` levanta el emulador; pruebas que publican y consumen un mensaje válido, uno inválido (termina en el dead-letter) y uno repetido (se ignora por `messageId`). | Joan | 5h | Por hacer |
| `ES-187` | `ST-72.1` | **Backend: Consumidor de pedidos empacados de Almacén:**<br>• Suscribirse, mediante `MessageBus`, a `inventory.shipment.ready`, `inventory.shipment.updated` e `inventory.shipment.cancelled`.<br>• Esquema de `data` obligatorio: `salesOrderId`, `recordVersion`, `updatedAt`, `warehouseId`, `availableFrom` y `packages[]` con al menos un bulto; cada bulto con `packageId`, `lengthCm`, `widthCm`, `heightCm` (mayores a 0) y `grossWeightKg` (mayor a 0). Opcionales: `handlingConditions[]` y `evidencePolicy` (`hand_delivery_standard`, `contactless_delivery` o `high_value_control`; si no viene, se usa `hand_delivery_standard`).<br>• Idempotencia: ignorar un `messageId` ya procesado (tabla de `ST-72.3`) y un `recordVersion` menor o igual al almacenado para ese `salesOrderId`.<br>• `ready` y `updated`: guardar o reemplazar los bultos en espera y entregar el resultado al unificador de `ST-72.2`. `cancelled`: marcar el envío como cancelado y, si ya existe el despacho en `pending`, anularlo; si está asignado, alertar al coordinador sin cambiar su estado.<br>• Un mensaje inválido no escribe nada en la base de datos y se envía al dead-letter con el detalle.<br>• Criterio de terminado: pruebas con el adaptador en memoria para mensaje válido, inválido, repetido, versión antigua y cancelación. | Sergio | 6h | Por hacer |
| `ES-188` | `ST-72.2` | **Backend: Consumidor de datos de entrega de Marketplace y unión por pedido de venta:**<br>• Suscribirse a `marketplace.sales_order.confirmed`, `marketplace.sales_order.updated` y `marketplace.sales_order.cancelled`. Esquema de `data` obligatorio: `salesOrderId`, `recordVersion`, `updatedAt`, destinatario (nombre y teléfono) y dirección con referencias y coordenadas (`lat`, `lng`). Opcionales: fecha límite y preferencias de entrega. No se envía condición de pago (todos los pedidos se asumen pagados) y no se aceptan datos de facturación: los campos desconocidos se ignoran y no se guardan.<br>• Aplicar la misma idempotencia que `ST-72.1`.<br>• Unión: cuando existan los datos de entrega y los bultos del mismo `salesOrderId`, en cualquier orden de llegada, entregar el conjunto consolidado al generador de órdenes (`ST-49.1`); mientras falte uno, guardar el otro en espera sin crear el despacho ni mostrarlo en la bandeja.<br>• Cambios y cancelaciones: si el despacho ya existe en `pending`, actualizar dirección o contacto; si ya está en una ruta (`assigned` o posterior), no modificarlo y emitir `dispatch.change_requires_attention` por el gateway indicando qué cambió.<br>• Criterio de terminado: pruebas con orden de llegada A-B y B-A, mensaje incompleto, cambio sobre pedido `pending`, cambio sobre pedido asignado y cancelación. | Joan | 4h | Por hacer |
| `ES-202` | `ST-72.3` | **BD: Tablas de recepción de mensajes, datos en espera y mensajes salientes:**<br>• Crear en `migrations/` (con `up.sql`, `down.sql` y `README.md`) la tabla `inbound_messages` (`message_id` UUID único, `type`, `source`, `received_at`, `status`: `processed` \| `rejected`, `error_detail`), la tabla `pending_sales_orders` (datos de entrega por `sales_order_id` único, `record_version`, `payload` JSONB), la tabla `pending_shipments` (bultos por `sales_order_id` único, `record_version`, `payload` JSONB) y la tabla `outbound_messages` (outbox: `message_id`, `type`, `payload`, `status`, `attempts`, `next_attempt_at`).<br>• Crear índices por `sales_order_id`, `status` y `next_attempt_at`.<br>• Documentar cada tabla con su `DICTIONARY.md` y actualizar `db-output.sql`.<br>• Criterio de terminado: migración aplicada y revertida sin errores en una base limpia y `npm run build` del proyecto `bdd` sin errores. | Sergio | 2h | Por hacer |
| `ES-203` | `ST-72.4` | **Backend: Recuperación de mensajes perdidos y alerta de pedidos incompletos:**<br>• Tarea programada cada 15 minutos que busca en `pending_sales_orders` y `pending_shipments` los registros que llevan más de `incomplete_order_alert_minutes` (clave de `settings`, 60 por defecto) esperando a su complemento.<br>• Para cada uno, publicar `dispatch.sync.requested` con `{ entityType, entityId, requestedAt }` dirigido al origen que falta (Marketplace si faltan los datos de entrega; Inventarios si faltan los bultos). El origen responde publicando el registro completo con su tópico habitual, y los consumidores de `ST-72.1` y `ST-72.2` lo aceptan aunque su `recordVersion` sea igual, porque el registro local no existe.<br>• Limitar a 3 solicitudes por pedido, separadas por el mismo intervalo; después de la tercera, emitir `dispatch.change_requires_attention` con `changeType: incomplete_order` para que el Coordinador lo gestione.<br>• Agregar al simulador (`ST-73.2`) el escenario «mensaje perdido» que responde a la solicitud.<br>• Criterio de terminado: pruebas con el adaptador en memoria para un pedido incompleto que se recupera, uno que no se recupera tras 3 solicitudes y uno que se completa antes de la alerta. | Sergio | 3h | Por hacer |

---

#### `ES-73`: 8.2 Como Operador de despacho / Integrador de sistemas, requiero publicar la salida física de los bultos hacia Almacén e Inventarios [ERP-02]
* **Épica:** 8.0 Interoperabilidad con el ERP y Sistemas Externos (`ES-11`)
* **Asignado (PE):** Sergio | **Story Points (Spe):** 4 | **Estado:** Por hacer (To Do)
* **Fechas:** Inicio (FI): 09 de Oct | Vencimiento (FV): 13 de Oct
* **Dependencias:** Bloquea a (B): `ES-76` | Bloqueado por (BP): `ES-72`

> **Como** Operador de despacho / Integrador de sistemas  
> **Quiero** informar a Inventarios y Almacén, mediante un mensaje asíncrono, el momento en que los bultos salen del centro de distribución bajo custodia del repartidor  
> **Para** que Almacén actualice sus registros y libere formalmente la custodia sin bloquear la salida del transportista.

* **Criterios de Aceptación (GWT):**
  * **Escenario 1 (Publicación de la salida física - ERP-02):**  
    **Dado** que el repartidor inicia su ruta con los bultos cargados (`ES-30`),  
    **cuando** se confirma la transición a `out_for_delivery`,  
    **entonces** Despachos registra la transferencia de custodia y publica `dispatch.shipment.departed` con el pedido de venta, los IDs de los bultos y la hora de salida.
  * **Escenario 2 (Tolerancia a fallos no bloqueante):**  
    **Dado** que el broker de mensajes no está disponible,  
    **cuando** se confirma la salida,  
    **entonces** la ruta avanza con normalidad, el mensaje queda en la tabla *outbox* y se publica con reintentos y backoff exponencial; tras 3 fallos definitivos se registra una alerta en la auditoría técnica.
  * **Escenario 3 (Verificación con el simulador de Almacén):**  
    **Dado** el simulador de Almacén activo,  
    **cuando** consume el mensaje publicado,  
    **entonces** lo registra y permite verificar el contrato saliente.

| ID Jira | Subtarea | Descripción Técnica y Tareas a Realizar | Asignado (PE) | Estimación | Estado |
| :--- | :--- | :--- | :---: | :---: | :---: |
| `ES-189` | `ST-73.1` | **Backend: Publicación de la salida física con patrón outbox:**<br>• Al confirmar la transición de la ruta a `out_for_delivery` (`ST-30.3`), registrar en la misma transacción la transferencia de custodia (hora de salida, repartidor y IDs de los bultos que salen) y escribir en `outbound_messages` el mensaje `dispatch.shipment.departed` con el `salesOrderId` y la lista de `packageId` de cada pedido de la ruta.<br>• El publicador de `ST-72.0` lo envía de forma asíncrona; la operación en calle nunca espera la respuesta del broker ni de Almacén.<br>• Si el envío falla definitivamente (3 intentos), dejar el mensaje en estado `failed` y registrar una alerta en la auditoría técnica visible para el Coordinador.<br>• Criterio de terminado: prueba que simula la caída del broker, comprueba que la ruta avanza y que el mensaje se publica al restablecerse. | Sergio | 5h | Por hacer |
| `ES-190` | `ST-73.2` | **Backend: Simulador ERP para desarrollo y pruebas (Marketplace, Almacén y Compras):**<br>• Crear scripts `npm` (por ejemplo `simulate:marketplace`, `simulate:inventory` y `simulate:procurement`) que publiquen en el emulador los mensajes de cada tópico de entrada a partir de *fixtures* JSON versionados en una carpeta de pruebas del backend.<br>• Incluir *fixtures* para: pedido completo, llegada en desorden, reenvío del mismo `messageId`, `recordVersion` antiguo, mensaje inválido por campo faltante, cancelación y cambio de dirección.<br>• Crear un consumidor simulado (`simulate:listen`) que se suscriba a los tópicos que publica Despachos y escriba cada mensaje recibido en consola.<br>• Documentar en el README del backend cada comando y el comportamiento que debe observarse.<br>• Criterio de terminado: ejecutar el escenario «pedido completo» crea un despacho `pending` visible en la bandeja y el escenario «mensaje inválido» termina en el dead-letter. | Joan | 4h | Por hacer |

---

#### `ES-74`: 8.3 Como Desarrollador de integraciones, requiero consumir las solicitudes de recogida en proveedor enviadas por Compras [ERP-03]
* **Épica:** 8.0 Interoperabilidad con el ERP y Sistemas Externos (`ES-11`)
* **Asignado (PE):** Sergio | **Story Points (Spe):** 4 | **Estado:** Por hacer (To Do)
* **Fechas:** Inicio (FI): 05 de Oct | Vencimiento (FV): 09 de Oct
* **Dependencias:** Bloquea a (B): `ES-75` | Bloqueado por (BP): `ES-14 (✅ Finalizada en Sprint 1)`

> **Como** Desarrollador de Integraciones Backend  
> **Quiero** consumir los mensajes de solicitud, cambio y cancelación de recogidas ligadas a órdenes de compra aprobadas  
> **Para** registrar los traslados de abastecimiento sin digitación manual, respetando que Compras es dueño de la orden de compra.

* **Criterios de Aceptación (GWT):**
  * **Escenario 1 (Registro de la solicitud de recogida - ERP-03):**  
    **Dado** un mensaje `procurement.pickup.requested` válido,  
    **cuando** se procesa,  
    **entonces** se registra un despacho de tipo `supplier_pickup` pendiente de planificación con el ID de la orden de compra, proveedor, dirección y contacto de recogida, horario disponible, productos y cantidades esperadas, estimaciones de carga y almacén de destino.
  * **Escenario 2 (Solicitud incompleta):**  
    **Dado** un mensaje de esquema válido sin dirección de recogida, sin horario imprescindible o con un almacén de destino inexistente en el catálogo (`ES-26`),  
    **cuando** se procesa,  
    **entonces** el despacho queda en `address_review` indicando el dato faltante y no puede asignarse hasta corregirse.
  * **Escenario 3 (Rechazo por esquema inválido):**  
    **Dado** un mensaje con pesos negativos, fechas pasadas o campos obligatorios faltantes,  
    **cuando** se procesa,  
    **entonces** se deriva al dead-letter con el detalle de los errores y no se crea ningún registro.
  * **Escenario 4 (Cambios y cancelación de la solicitud):**  
    **Dado** una recogida que aún no ha sido ejecutada,  
    **cuando** llega `procurement.pickup.updated` o `procurement.pickup.cancelled`,  
    **entonces** el despacho se actualiza o se anula; Despachos nunca modifica la orden de compra.

| ID Jira | Subtarea | Descripción Técnica y Tareas a Realizar | Asignado (PE) | Estimación | Estado |
| :--- | :--- | :--- | :---: | :---: | :---: |
| `ES-191` | `ST-74.1` | **Backend: Consumidor de solicitudes de recogida de Compras:**<br>• Suscribirse a `procurement.pickup.requested`, `procurement.pickup.updated` y `procurement.pickup.cancelled`.<br>• Esquema de `data` obligatorio: `purchaseOrderId`, `recordVersion`, `updatedAt`, proveedor (nombre), dirección de recogida con referencias y coordenadas, contacto de recogida (nombre y teléfono), fecha y horario disponibles (fecha no anterior a hoy), `items[]` (código, descripción y cantidad esperada mayor a 0) y `warehouseId` de destino. Opcionales: estimaciones de cajas, peso (kg) y dimensiones (cm), y `transportConditions[]` (fragilidad, temperatura).<br>• Idempotencia igual que `ST-72.1`.<br>• Crear un despacho de tipo `supplier_pickup` con `source_order_ref = purchaseOrderId`. Si falta la dirección de recogida o el horario, o el almacén de destino no existe en el catálogo (`ES-26`), dejarlo en `address_review` indicando el dato faltante; no puede asignarse hasta corregirse.<br>• `updated` y `cancelled`: aplicar solo mientras el despacho siga en `pending`, `address_review` o `scheduled`; en otro caso, alertar al coordinador. Despachos nunca modifica la orden de compra.<br>• Criterio de terminado: pruebas con solicitud válida, almacén inexistente, esquema inválido (dead-letter), cambio y cancelación. | Sergio | 5h | Por hacer |
| `ES-192` | `ST-74.2` | **BD: Modelado de recogidas en proveedor:**<br>• Crear la tabla `dispatch_pickup_items` (`dispatch_id`, `item_code`, `description`, `expected_quantity`, `received_quantity` nullable) y agregar a `dispatches` los campos opcionales `estimated_boxes`, `estimated_volume_m3` y `transport_conditions` (texto); el peso usa el `estimated_weight_kg` existente y el tipo `supplier_pickup` ya existe en `dispatch_types`.<br>• Agregar `destination_warehouse_id` a `dispatches` con clave foránea a la tabla `warehouses`. Esa tabla es un requisito previo al sprint (corrección pendiente de `ES-26`) y no se crea en esta subtarea.<br>• Migración limpia en `migrations/` con `up.sql`, `down.sql` y `README.md`; actualizar los `DICTIONARY.md` afectados y `db-output.sql`.<br>• Criterio de terminado: migración aplicada y revertida sin errores en una base limpia. | Sergio | 3h | Por hacer |

---

#### `ES-75`: 8.4 Como Coordinador de transporte, necesito programar el recojo de mercadería en proveedores hacia el almacén destino [ERP-04]
* **Épica:** 8.0 Interoperabilidad con el ERP y Sistemas Externos (`ES-11`)
* **Asignado (PE):** Joan | **Story Points (Spe):** 4 | **Estado:** Por hacer (To Do)
* **Fechas:** Inicio (FI): 12 de Oct | Vencimiento (FV): 15 de Oct
* **Dependencias:** Bloquea a (B): `ES-76` | Bloqueado por (BP): `ES-74`

> **Como** Coordinador de Transporte  
> **Quiero** programar la recogida en el proveedor según su horario, el horario de recepción del almacén de destino y la capacidad estimada de la flota  
> **Para** integrar los flujos de abastecimiento en la planificación diaria y confirmar a Compras la fecha y franja comprometidas.

* **Criterios de Aceptación (GWT):**
  * **Escenario 1 (Programación de la recogida - ERP-04):**  
    **Dado** una solicitud de recogida completa,  
    **cuando** el coordinador la agenda,  
    **entonces** el sistema valida el horario del proveedor y el de recepción del almacén, ordena las paradas visitando primero al proveedor y publica `dispatch.pickup.scheduled` con la fecha y la franja.
  * **Escenario 2 (Reprogramación ante indisponibilidad del proveedor):**  
    **Dado** que el proveedor no tiene la carga lista en el momento pactado,  
    **cuando** se registra la incidencia,  
    **entonces** el coordinador reprograma la recogida y se publican `dispatch.transfer.incident_reported` y un nuevo `dispatch.pickup.scheduled` con el `recordVersion` incrementado.

| ID Jira | Subtarea | Descripción Técnica y Tareas a Realizar | Asignado (PE) | Estimación | Estado |
| :--- | :--- | :--- | :---: | :---: | :---: |
| `ES-193` | `ST-75.1` | **Backend: Servicio de programación de recogidas en proveedor:**<br>• Validar, antes de agendar, que el despacho `supplier_pickup` tenga origen, destino y horarios completos. Calcular la ventana de atención como la intersección entre el horario del proveedor y el horario de recepción del almacén (si el catálogo no lo define, usar la ventana general de `ES-57`) y rechazar con 409 si no existe intersección o si la capacidad estimada excede la del vehículo (`ST-50.1`).<br>• Al agendar, pasar el despacho a `scheduled` y escribir en la *outbox* el mensaje `dispatch.pickup.scheduled` (fecha, franja y `purchaseOrderId`).<br>• Reprogramación por indisponibilidad del proveedor: registrar la incidencia, reprogramar la fecha con la misma lógica de `dispatch_reschedules` y publicar `dispatch.transfer.incident_reported` y un nuevo `dispatch.pickup.scheduled` con `recordVersion` incrementado.<br>• Criterio de terminado: pruebas unitarias con ventanas que se intersectan, que no se intersectan y reprogramación. | Joan | 5h | Por hacer |
| `ES-194` | `ST-75.2` | **Backend: Pruebas de integración del circuito Compras ↔ Despachos:**<br>• Escribir pruebas con Jest que usen el adaptador de mensajería en memoria (`ST-72.0`) y cubran: solicitud válida que crea el despacho `supplier_pickup`; solicitud con almacén inexistente que termina en `address_review`; programación que publica `dispatch.pickup.scheduled`; reprogramación que publica un nuevo mensaje con `recordVersion` mayor; cambio y cancelación mientras está pendiente; mensaje inválido que termina en el dead-letter.<br>• Incluir un caso de reenvío del mismo `messageId` que no duplica el despacho.<br>• Criterio de terminado: `npm run test` ejecuta los casos y todos pasan sin depender de Docker. | Sergio | 3h | Por hacer |

---

#### `ES-76`: 8.5 Como Arquitecto de integraciones, requiero publicar el catálogo de mensajes y los esquemas JSON de los contratos asíncronos [ERP-05]
* **Épica:** 8.0 Interoperabilidad con el ERP y Sistemas Externos (`ES-11`)
* **Asignado (PE):** Joan | **Story Points (Spe):** 3 | **Estado:** Por hacer (To Do)
* **Fechas:** Inicio (FI): 16 de Oct | Vencimiento (FV): 19 de Oct *(Ajustado a 2 días hábiles)*
* **Dependencias:** Bloquea a (B): Ninguna | Bloqueado por (BP): `ES-73`, `ES-75`

> **Como** Arquitecto de Integraciones  
> **Quiero** contar con el catálogo de tópicos y un JSON Schema validado por cada mensaje intercambiado con el ERP  
> **Para** que los demás equipos y nuestros simuladores produzcan y consuman mensajes sin ambigüedades.

* **Criterios de Aceptación (GWT):**
  * **Escenario 1 (Catálogo y esquemas publicados - ERP-05):**  
    **Dado** el repositorio del backend,  
    **cuando** se consulta la carpeta de contratos,  
    **entonces** existe un JSON Schema y un ejemplo válido por cada tópico del catálogo de `docs/interoperabilidad-y-flujos.md`.
  * **Escenario 2 (Validación automática de contratos):**  
    **Dado** el comando de verificación de contratos,  
    **cuando** se ejecuta,  
    **entonces** valida todos los ejemplos y *fixtures* de los simuladores contra sus esquemas y falla indicando archivo y campo si alguno no cumple.

> **Nota:** los endpoints REST propios (web y móvil) se documentan con Swagger según `AGENTS.md` y no requieren HU propia.

| ID Jira | Subtarea | Descripción Técnica y Tareas a Realizar | Asignado (PE) | Estimación | Estado |
| :--- | :--- | :--- | :---: | :---: | :---: |
| `ES-195` | `ST-76.1` | **Backend: Consolidación de JSON Schemas y ejemplos de mensajes:**<br>• Crear la carpeta de contratos `contracts/messages/` con un archivo JSON Schema por cada tópico del catálogo (`docs/interoperabilidad-y-flujos.md`, sección 1.4) y un ejemplo válido por tópico.<br>• Cada esquema declara `schemaVersion`, campos obligatorios y opcionales, tipos, formatos (ISO-8601 UTC), unidades (kg, cm) y valores permitidos (por ejemplo `paymentCondition`).<br>• Mantener sincronizados el catálogo del documento y la carpeta: si se agrega un tópico se agregan ambos.<br>• Criterio de terminado: existe un esquema y un ejemplo por cada tópico, y los consumidores de `ST-72.1`, `ST-72.2` y `ST-74.1` usan esos mismos archivos. | Joan | 4h | Por hacer |
| `ES-196` | `ST-76.2` | **Backend: Verificación automática de contratos:**<br>• Crear el script `npm run contracts:check` que valide todos los ejemplos de `contracts/messages/` y todos los *fixtures* del simulador (`ST-73.2`) contra su esquema, y que falle indicando archivo y campo cuando alguno no cumpla.<br>• Verificar además que cada tópico del catálogo tenga esquema y ejemplo, y que cada esquema declare `schemaVersion`.<br>• Integrar el script en `npm run test` y en el pipeline de integración continua si existe.<br>• Criterio de terminado: el script pasa con los contratos actuales y falla al alterar un campo obligatorio de un ejemplo. | Sergio | 3h | Por hacer |

---

## 3. Matriz Resumen de Carga Horaria y Capacidad — Sprint 2

| Desarrollador | Roles Rotados en Sprint 2 | Subtareas Asignadas (Sprint 2) | Horas Totales Estimadas |
| :--- | :--- | :--- | :---: |
| **Jaime** | Mobile Developer & DBA | `ST-28.1` (6h), `ST-28.3` (4h), `ST-29.1` (6h), `ST-29.3` (4h), `ST-30.1` (5h), `ST-31.1` (5h), `ST-31.2` (5h), `ST-32.1` (5h), `ST-35.1` (3h), `ST-78.1` (4h) | **47 hrs** |
| **Sergio** | Backend Developer & Web Front | `ST-30.3` (4h), `ST-31.4` (3h), `ST-32.2` (4h), `ST-33.2` (3h), `ST-36.2` (4h), `ST-80.2` (4h), `ST-49.1` (6h), `ST-49.2` (5h), `ST-50.1` (6h), `ST-51.2` (5h), `ST-53.2` (4h), `ST-54.2` (3h), `ST-57.2` (5h), `ST-58.1` (4h), `ST-81.2` (4h), `ST-72.1` (6h), `ST-72.3` (2h), `ST-72.4` (3h), `ST-73.1` (5h), `ST-74.1` (5h), `ST-74.2` (3h), `ST-75.2` (3h), `ST-76.2` (3h) | **94 hrs** |
| **Pardo** | Frontend Web Lead | `ST-48.1` (4h), `ST-51.1` (5h), `ST-52.1` (4h), `ST-53.1` (6h), `ST-54.1` (4h), `ST-55.1` (4h), `ST-56.1` (5h), `ST-57.1` (4h), `ST-81.1` (4h) | **40 hrs** |
| **Joan** | Backend Developer & Mobile Architecture | `ST-28.2` (6h), `ST-28.4` (4h), `ST-29.2` (4h), `ST-30.2` (5h), `ST-31.3` (6h), `ST-31.5` (3h), `ST-33.1` (4h), `ST-35.2` (3h), `ST-36.1` (4h), `ST-36.3` (2h), `ST-80.1` (5h), `ST-48.2` (4h), `ST-50.2` (5h), `ST-52.2` (4h), `ST-55.2` (3h), `ST-56.2` (4h), `ST-58.2` (3h), `ST-72.0` (5h), `ST-72.2` (4h), `ST-73.2` (4h), `ST-75.1` (5h), `ST-76.1` (4h) | **91 hrs** |

> [!NOTE]
> **Balance y Gestión de Capacidad en el Sprint 2:**  
> Las estimaciones de horas representan asignaciones de tiempo holgadas para garantizar cobertura total. El margen de tiempo disponible por cada miembro del equipo se destina formalmente a la realización de Code Reviews cruzados (Sergio en Frontend/Mobile, Jaime en Backend), ceremonias de Scrum y colchón ante contingencias de integración con el ERP.
