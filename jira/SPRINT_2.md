# SPRINT_2.md — Desglose Operativo del Sprint 2: Historias de Usuario y Subtareas Técnicas

> **Proyecto:** Microservicio de Gestión de Entregas y Despachos (Grupo H / ERP Corporativo)  
> **Sprint:** 2  
> **Fuente de Trazabilidad:** Jira Cloud / GitHub  
> **Documento Padre:** [`PROJECT_BACKLOG.md`](./PROJECT_BACKLOG.md)  
> **Rotación de Roles en Sprint 2:**
> * **Backend (NestJS / REST / Integraciones ERP):** Sergio & Joan
> * **App Mobile (Expo / React Native / SQLite):** Jaime & Joan
> * **Frontend Web (React 19 / Vite / Tailwind):** Pardo & Sergio
> * **Base de Datos (PostgreSQL / Migraciones / Modelado):** Jaime

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
    **entonces** la aplicación consume `GET /api/v1/despachos/mis-asignaciones` y despliega la lista de paradas ordenadas cronológica y geográficamente (`stop_number: 1, 2, 3...`) con: número de parada, código de guía, cliente, dirección resumida, franja horaria comprometida (`14:00 - 16:00`), estado operativo (`ASIGNADO`, `EN_RUTA`, `EN_CAMINO`) y política de evidencia requerida.
  * **Escenario 2 (Inmutabilidad del orden de paradas por el conductor):**  
    **Dado** un lote de pedidos en estado `EN_RUTA`,  
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
| `ES-130` | `ST-28.1` | **Mobile UI: Pantalla "Mi Jornada" con lista virtualizada (FlashList), orden de paradas y pull-to-refresh:**<br>• Construir pantalla con listado de alto rendimiento utilizando `@shopify/flash-list`.<br>• Diseñar tarjeta de parada mostrando badge numérico de secuencia (`Parada #1`, `#2`), código de orden, nombre de cliente, franja horaria comprometida (`hh:mm - hh:mm`) y badge cromático de estado (`ASIGNADO`, `EN_RUTA`, `EN_CAMINO`).<br>• Implementar control de interacción que destaque visualmente la **parada activa inmediata** y deshabilite el inicio prematuro de paradas posteriores.<br>• Integrar soporte nativo de `RefreshControl` para pull-to-refresh y componentes de retroalimentación (Skeleton loading anatómico y estado vacío ilustrado). | Jaime | 6h | Por hacer |
| `ES-131` | `ST-28.2` | **Mobile Data: Servicio de asignaciones, repositorio SQLite (`local_dispatches`) y sincronización:**<br>• Crear servicio de consumo para `GET /api/v1/despachos/mis-asignaciones` inyectando Bearer token en Axios.<br>• Definir esquema de tabla `local_dispatches` en SQLite con índices en `stop_number`, `status` y `dispatch_id`.<br>• Implementar repositorio híbrido que retorne datos de SQLite local con fallback ante desconexión o latencia alta.<br>• Mapear banderas de sincronización (`is_synced`, `updated_at`) y manejar purga de despachos de jornadas anteriores concluidas. | Joan | 6h | Por hacer |
| `ES-132` | `ST-28.3` | **Mobile UI: Componentes de tarjeta de parada con badges de SLA, tipo de servicio y política de entrega:**<br>• Implementar badges distintivos por tipo de entrega (Express / Estándar / Recojo RMA).<br>• Añadir indicador de cuenta regresiva o badge de urgencia si la hora actual está próxima al límite superior de la franja comprometida.<br>• Incorporar badge informativo con la política de evidencia requerida por pedido (Firma, Foto sin contacto o Código OTP).<br>• Configurar navegación tipada (`expo-router`) hacia la vista de detalle pasando el UUID del despacho. | Jaime | 4h | Por hacer |

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
  * **Escenario 2 (Etiqueta informativa de pago sin cobranzas en campo):**  
    **Dado** un pedido visualizado en el dispositivo móvil,  
    **cuando** se inspecciona la sección de estado financiero,  
    **entonces** la app exhibe exclusivamente el badge informativo corporativo provisto por el ERP (`PREPAGADO`, `FACTURADO`, `CUENTA_CORRIENTE`), bloqueando cualquier opción de registro o cobro de dinero en efectivo.
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
| `ES-133` | `ST-29.1` | **Mobile UI: Pantalla de detalle de despacho modularizada en React Native con NativeWind:**<br>• Crear vista `/pedido/[id].tsx` con layout modular: encabezado con badge de estado, tarjeta de cliente con botón de acción telefónica, tarjeta de dirección y mapa estático miniatura.<br>• Diseñar sección de packing list renderizando ítems físicos, unidades, volumen en $m^3$ y peso en kg sin montos monetarios.<br>• Añadir banner destacado informando la política de prueba de entrega (POD) que exigirá la app al momento de entregar.<br>• Incorporar botón principal de acción contextual ("Iniciar Traslado a este Domicilio" o "Gestionar Entrega"). | Jaime | 6h | Por hacer |
| `ES-134` | `ST-29.2` | **Mobile Native: Integración de Deep Linking (Google Maps/Waze) y marcador telefónico (Linking API):**<br>• Construir utilitario con esquemas URL robustos (`geo:lat,lng?q=...`, `https://www.google.com/maps/dir/?api=1&destination=...`, `waze://?ll=lat,lng&navigate=yes`).<br>• Implementar modal nativo para seleccionar la aplicación de mapas preferida con persistencia en `AsyncStorage`.<br>• Configurar llamada telefónica segura con `Linking.openURL('tel:...')` capturando excepciones si el dispositivo no posee SIM.<br>• Añadir validaciones de sanitización en números telefónicos y coordenadas nulas con alerta contextual. | Joan | 4h | Por hacer |
| `ES-135` | `ST-29.3` | **Mobile Data: Modelado de tipos TypeScript, selectores SQLite y validación de esquemas ERP:**<br>• Definir interfaces TypeScript exhaustivas (`DispatchDetail`, `DispatchItem`, `EvidenceRequirementType`, `CustomerInfo`).<br>• Crear selector en Zustand / React Query que consulte primero SQLite local (`local_dispatches`) y ejecute revalidación en segundo plano si hay red.<br>• Validar presentación de textos largos, referencias de dirección e instrucciones especiales sin desbordamiento de pantalla. | Jaime | 4h | Por hacer |

---

#### `ES-30`: 3.4 Como Repartidor en campo, necesito actualizar el estado operativo de cada despacho con soporte fuera de línea [RF-U04]
* **Épica:** 3.0 Aplicación Móvil de Repartidor y Telemetría (`ES-6`)
* **Asignado (PE):** Joan | **Story Points (Spe):** 5 | **Estado:** Por hacer (To Do)
* **Fechas:** Inicio (FI): 12 de Oct | Vencimiento (FV): 15 de Oct
* **Dependencias:** Bloquea a (B): `ES-31`, `ES-32`, `ES-33`, `ES-35`, `ES-61`, `ES-69` | Bloqueado por (BP): `ES-28`, `ES-29`, `ES-79 (✅ Finalizada en Sprint 1)`

> **Como** Repartidor en campo  
> **Quiero** actualizar el ciclo de estado de mi ruta y de cada despacho individual (`EN_RUTA`, `EN_CAMINO`, `ENTREGADO`, `NO_ENTREGADO`, `DEVUELTO`) directamente desde la app móvil con soporte offline y regla de traslado activo único  
> **Para** sincronizar la trazabilidad del envío en tiempo real con la torre de control y asegurar que los clientes reciban las notificaciones precisas de proximidad.

* **Criterios de Aceptación (GWT):**
  * **Escenario 1 (Salida de almacén y transición masiva a 'En Ruta'):**  
    **Dado** un conjunto de pedidos en estado `ASIGNADO` cargados en el vehículo al salir del CEDIS,  
    **cuando** el repartidor presiona el botón principal "Iniciar Ruta del Día",  
    **entonces** todos los pedidos de su hoja de ruta transicionan a `EN_RUTA`, se notifica al servidor/cola offline y los clientes reciben la notificación de que sus pedidos salieron a reparto.
  * **Escenario 2 (Inicio de traslado puntual a parada específica y regla de unicidad de 'En Camino'):**  
    **Dado** que los pedidos están en estado `EN_RUTA` y no hay ningún otro pedido en curso,  
    **cuando** el repartidor selecciona su siguiente parada en secuencia y presiona "Iniciar Traslado a este Domicilio",  
    **entonces** ese pedido específico pasa a estado `EN_CAMINO`, se captura la coordenada GPS de partida, se notifica al cliente que el repartidor se aproxima a su domicilio, y la app bloquea el inicio simultáneo de cualquier otro pedido hasta que este sea entregado o reportado con incidencia.
  * **Escenario 3 (Validación de precondición estricta de evidencia para entrega final):**  
    **Dado** un despacho en estado `EN_CAMINO`,  
    **cuando** el repartidor intenta transicionar el estado a `ENTREGADO`,  
    **entonces** la máquina de estados valida estrictamente que la petición incluya el payload con la evidencia digital exigida (`ES-31`); si falta la evidencia requerida (ej. firma omitida o código OTP ausente), la app rechaza la transición impidiendo el cierre del pedido.
  * **Escenario 4 (Persistencia y encolamiento FIFO offline ante pérdida de señal):**  
    **Dado** un dispositivo sin cobertura celular en el momento de actualizar un estado (`EN_CAMINO`, `ENTREGADO`, `NO_ENTREGADO`),  
    **cuando** el repartidor confirma la acción,  
    **entonces** el evento se guarda de forma atómica en la tabla `local_events` de SQLite con `synced = 0`, UUID v7 y timestamp UTC, actualizando la vista local del pedido y mostrando el badge "Pendiente de sincronizar" sin interrumpir la operación del conductor.

| ID Jira | Subtarea | Descripción Técnica y Tareas a Realizar | Asignado (PE) | Estimación | Estado |
| :--- | :--- | :--- | :---: | :---: | :---: |
| `ES-147` | `ST-30.1` | **Mobile UI: Máquina de estados finita de despacho con validación de transiciones legales y unicidad:**<br>• Implementar máquina de estados finitos para el ciclo de vida del despacho: `ASIGNADO` $\rightarrow$ `EN_RUTA` $\rightarrow$ `EN_CAMINO` $\rightarrow$ (`ENTREGADO` \| `NO_ENTREGADO` \| `INCIDENCIA`) $\rightarrow$ `DEVUELTO`.<br>• Diseñar botones de acción condicionales: botón global "Iniciar Ruta" en cabecera y botón "Rumbo a este Domicilio" deshabilitado si ya existe una orden activa en `EN_CAMINO`.<br>• Incorporar modal de confirmación antes de confirmar la entrega final o marcar imposibilidad de entrega. | Jaime | 5h | Por hacer |
| `ES-148` | `ST-30.2` | **Mobile Data: Encolamiento FIFO en SQLite (`local_events`), idempotencia y sincronizador resiliente:**<br>• Extender repositorio SQLite local para registrar eventos de transición con coordenadas GPS (`lat`, `lng`, `accuracy`), timestamp ISO UTC y UUID v7 idempotente.<br>• Conectar con `NetInfo` para disparar el vaciado secuencial de la cola FIFO en orden estricto de ocurrencia tan pronto se detecte conectividad activa.<br>• Manejar resolución de conflictos garantizando que eventos desfasados en red no sobrescriban estados terminales del servidor. | Joan | 5h | Por hacer |
| `ES-149` | `ST-30.3` | **Backend: Endpoint transaccional `PATCH /api/v1/dispatch/:id/status` con auditoría y eventos Socket.io:**<br>• Implementar endpoint en NestJS con DTO validado (`UpdateDispatchStatusDto`) verificando que la transición solicitada sea legal según el estado actual en PostgreSQL.<br>• Validar regla de negocio de precondición de evidencias POD registradas antes de permitir el cambio a `ENTREGADO`.<br>• Emitir eventos de dominio internos (`dispatch.status_changed`) hacia el Gateway de Socket.io para actualización en vivo de la torre de control y registro inmutable en auditoría. | Sergio | 4h | Por hacer |

---

#### `ES-31`: 3.5 Como Repartidor en entrega, requiero registrar evidencia digital mediante fotografía, firma en pantalla o código OTP según la política del pedido [RF-U05]
* **Épica:** 3.0 Aplicación Móvil de Repartidor y Telemetría (`ES-6`)
* **Asignado (PE):** Jaime | **Story Points (Spe):** 5 | **Estado:** Por hacer (To Do)
* **Fechas:** Inicio (FI): 16 de Oct | Vencimiento (FV): 20 de Oct
* **Dependencias:** Bloquea a (B): `ES-80` | Bloqueado por (BP): `ES-30`

> **Como** Repartidor en domicilio de destino  
> **Quiero** capturar la evidencia digital de entrega requerida de forma parametrizada (firma digital en pantalla, fotografía de respaldo o validación estricta de código OTP enviado al cliente)  
> **Para** contar con una prueba de entrega (POD) fehaciente, inmutable y legalmente válida que respalde la finalización del servicio según la política de seguridad de cada pedido sin fricciones innecesarias.

* **Criterios de Aceptación (GWT):**
  * **Escenario 1 (Entrega estándar en mano con captura de firma electrónica):**  
    **Dado** un pedido configurado con política de entrega en mano estándar (`HAND_DELIVERY_STANDARD`),  
    **cuando** el receptor firma con el dedo en el lienzo táctil y confirma sus datos (nombre y documento),  
    **entonces** la app genera el archivo PNG en almacenamiento privado del dispositivo, almacena la firma y habilita la confirmación de la entrega.
  * **Escenario 2 (Entrega sin contacto autorizada con fotografía georreferenciada):**  
    **Dado** un pedido con política de entrega sin contacto autorizada previamente por el cliente (`CONTACTLESS_DELIVERY`),  
    **cuando** el repartidor captura la fotografía del paquete depositado en el lugar acordado,  
    **entonces** la app comprime la imagen a JPEG (< 500 KB), estampa los metadatos de timestamp y coordenadas GPS, y aprueba la entrega sin exigir firma.
  * **Escenario 3 (Entrega de alta seguridad con validación obligatoria de código OTP):**  
    **Dado** un pedido de alto valor o control estricto (`HIGH_VALUE_CONTROL`) donde el backend emitió un OTP de 6 dígitos al teléfono verificado del cliente por SMS,  
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
| `ES-150` | `ST-31.1` | **Mobile UI/Native: Lienzo táctil vectorial para firma electrónica y exportación en disco privado:**<br>• Integrar componente de lienzo táctil fluido (`react-native-signature-canvas`) con trazado suave en tiempo real.<br>• Implementar botones de acción ("Limpiar trazo", "Cancelar", "Confirmar acuse") y campos obligatorios de nombre y carnet de identidad (CI) de quien recibe.<br>• Guardar la imagen vectorial exportada en formato PNG en el almacenamiento interno de la app (`expo-file-system`) sin almacenarla en Base64 en memoria SQLite. | Jaime | 5h | Por hacer |
| `ES-151` | `ST-31.2` | **Mobile Native: Módulo de cámara (`expo-camera`), compresión JPEG y metadatos EXIF:**<br>• Configurar acceso a cámara con gestión nativa de permisos y visor preliminar con opción de descartar/reintentar.<br>• Implementar pipeline de compresión automática mediante `expo-image-manipulator` (redimensionamiento a máx. 1280px y calidad 0.7, peso final < 500 KB).<br>• Inyectar metadatos de geolocalización precisa (`lat`, `lng`) y timestamp ISO en el archivo local generado.<br>• Encolar en `local_events` la referencia URI local (`file://...`) para la sincronización diferida por lotes. | Jaime | 5h | Por hacer |
| `ES-152` | `ST-31.3` | **Backend: Servicio multipart de almacenamiento POD y validador criptográfico de OTP:**<br>• Implementar endpoint `POST /api/v1/dispatch/:id/proof-of-delivery` con interceptor `FileInterceptor` para cargas binarias `multipart/form-data`.<br>• Almacenar archivos en disco parametrizado / MinIO / S3 con nombres UUID únicos y persistir registro en `delivery_evidences`.<br>• Diseñar endpoint `POST /api/v1/dispatch/:id/validate-otp` en NestJS con verificación de hash, control de TTL (10 min) y bloqueo tras 3 intentos inválidos.<br>• Emitir notificación inmediata de confirmación hacia la torre de control y portal de cliente. | Joan | 6h | Por hacer |

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
    **Dado** un despacho en estado `EN_CAMINO` que no puede concretarse en destino,  
    **cuando** el repartidor abre el formulario de incidencias,  
    **entonces** la app despliega el catálogo de causales activas (`CLIENTE_AUSENTE`, `DIRECCION_ERRONEA`, `ZONA_INACCESIBLE`, `RECHAZADO_POR_CLIENTE`, `PAQUETE_DANADO`), exigiendo fotografía obligatoria de respaldo (ej. fachada cerrada con número visible o bulto dañado) antes de permitir el envío.
  * **Escenario 2 (Transición a estado 'Incidencia' y liberación de ruta):**  
    **Dado** que el repartidor confirma el reporte de incidencia con su fotografía y comentarios,  
    **cuando** se procesa la transacción,  
    **entonces** el despacho pasa a estado `INCIDENCIA`, se remueve de la parada activa inmediata y el repartidor queda habilitado para continuar con la siguiente parada de su jornada.
  * **Escenario 3 (Almacenamiento offline de reportes de incidencia):**  
    **Dado** un repartidor sin cobertura celular en el momento de reportar una incidencia en puerta,  
    **cuando** confirma el formulario con la foto de fachada,  
    **entonces** el evento se guarda localmente en `local_events` de SQLite con bandera de sincronización pendiente y la orden se actualiza localmente permitiendo continuar la ruta sin bloqueos.

| ID Jira | Subtarea | Descripción Técnica y Tareas a Realizar | Asignado (PE) | Estimación | Estado |
| :--- | :--- | :--- | :---: | :---: | :---: |
| `ES-153` | `ST-32.1` | **Mobile UI: Formulario de reporte de incidencias con selector tipificado, fotos y observaciones:**<br>• Diseñar pantalla modal de reporte de incidencias con dropdown de causales sincronizado desde SQLite (`incident_reasons`).<br>• Integrar componente de captura fotográfica obligatoria reutilizando el pipeline de compresión de `ST-31.2`.<br>• Añadir campo de texto multilínea para observaciones del conductor (mínimo 10 caracteres si la causal es "Otro").<br>• Conectar con la máquina de estados para transicionar la orden a `INCIDENCIA` y habilitar la siguiente parada en la lista. | Jaime | 5h | Por hacer |
| `ES-154` | `ST-32.2` | **Backend: Endpoint `POST /api/v1/dispatch/:id/incident` y alerta operativa Socket.io:**<br>• Crear controlador y servicio en NestJS para recibir el reporte de incidentes con DTO validado (`CreateDispatchIncidentDto`).<br>• Actualizar estado del despacho a `INCIDENCIA` y persistir registro relacional en `dispatch_incidents`.<br>• Emitir evento WebSocket (`incident.reported`) con criticidad alta para alertar visualmente al Coordinador/Supervisor en el panel web. | Sergio | 4h | Por hacer |

---

#### `ES-33`: 3.7 Como Repartidor, requiero consultar el historial de entregas concluidas en jornadas anteriores [RF-U07]
* **Épica:** 3.0 Aplicación Móvil de Repartidor y Telemetría (`ES-6`)
* **Asignado (PE):** Joan | **Story Points (Spe):** 3 | **Estado:** Por hacer (To Do)
* **Fechas:** Inicio (FI): 21 de Oct | Vencimiento (FV): 22 de Oct
* **Dependencias:** Bloquea a (B): Ninguna | Bloqueado por (BP): `ES-30`

> **Como** Repartidor  
> **Quiero** revisar el histórico de despachos finalizados en días previos con filtros por fecha y resultado  
> **Para** auditar mis entregas realizadas, corroborar liquidaciones y responder ante cualquier consulta de ruta.

* **Criterios de Aceptación (GWT):**
  * **Escenario 1 (Filtrado por rango temporal en historial móvil):**  
    **Dado** que el repartidor ingresa al módulo "Historial",  
    **cuando** selecciona un rango de fechas (últimos 7 días, mes actual),  
    **entonces** la app muestra las órdenes cerradas organizadas cronológicamente con su comprobante de entrega y estado final (`ENTREGADO`, `DEVUELTO`).
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
| `ES-155` | `ST-33.1` | **Mobile UI/Data: Pestaña "Historial" con paginación infinita y almacenamiento local SQLite:**<br>• Crear vista de historial en la app móvil con buscador rápido por código de guía y selector de rango de fechas.<br>• Diseñar tarjetas compactas con resultado de entrega (Entregado / Devuelto), fecha/hora y botón de ver comprobante POD.<br>• Conectar con tabla local de historial en SQLite para permitir consulta offline de órdenes de los últimos 3 días. | Joan | 4h | Por hacer |
| `ES-156` | `ST-33.2` | **Backend: Endpoint `GET /api/v1/dispatch/history/driver` con paginación server-side:**<br>• Implementar consulta indexada en TypeORM filtrando por `driver_id` y estados terminales (`ENTREGADO`, `DEVUELTO`).<br>• Retornar metadatos de paginación (`page`, `limit`, `total_pages`) e información condensada del receptor y comprobante. | Sergio | 3h | Por hacer |

---

#### `ES-35`: 3.9 Como Repartidor, necesito visualizar el panel con mis métricas de desempeño y calificación de servicio [RF-U09]
* **Épica:** 3.0 Aplicación Móvil de Repartidor y Telemetría (`ES-6`)
* **Asignado (PE):** Jaime | **Story Points (Spe):** 2 | **Estado:** Por hacer (To Do)
* **Fechas:** Inicio (FI): 22 de Oct | Vencimiento (FV): 23 de Oct
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
| `ES-157` | `ST-35.1` | **Mobile UI: Componente dashboard de KPIs personales del conductor:**<br>• Construir componentes visuales con barras circulares de progreso y tarjetas métricas de efectividad de entrega.<br>• Integrar selector de período ("Hoy", "Esta Semana", "Este Mes") con transiciones suaves.<br>• Diseñar tarjeta de calificación media basada en estrellas con contador de valoraciones recibidas. | Jaime | 3h | Por hacer |
| `ES-158` | `ST-35.2` | **Backend: Endpoint `GET /api/v1/dispatch/metrics/driver` con agregaciones y caché Redis:**<br>• Crear servicio de agregación rápida en PostgreSQL para calcular métricas individuales (tasa de entrega a tiempo vs SLA).<br>• Optimizar respuesta con caché Redis de corta duración (TTL 5 min) asociada al ID del repartidor. | Joan | 3h | Por hacer |

---

#### `ES-36`: 3.10 Como Repartidor en turno, requiero recibir notificaciones push nativas ante la asignación de un nuevo pedido [RF-U10]
* **Épica:** 3.0 Aplicación Móvil de Repartidor y Telemetría (`ES-6`)
* **Asignado (PE):** Joan | **Story Points (Spe):** 3 | **Estado:** Por hacer (To Do)
* **Fechas:** Inicio (FI): 19 de Oct | Vencimiento (FV): 21 de Oct
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
| `ES-159` | `ST-36.1` | **Mobile Native: Configuración de listeners de notificaciones con Expo Notifications:**<br>• Configurar canal de notificación Android con prioridad alta, patrón de vibración y sonido corporativo.<br>• Implementar servicio de registro y envío del Push Token al backend tras el inicio de sesión exitoso.<br>• Configurar listener `addNotificationResponseReceivedListener` para navegar al detalle de la orden asignada vía deep-linking. | Joan | 4h | Por hacer |
| `ES-160` | `ST-36.2` | **Backend: Servicio emisor de notificaciones push en NestJS:**<br>• Implementar servicio de emisión push (Firebase Admin SDK / Expo Server SDK) vinculado al evento de asignación de órdenes.<br>• Diseñar payload estructurado con `title`, `body`, `dispatchId`, `zoneName` y metadatos de prioridad.<br>• Manejar reintentos silenciosos y limpieza automática de tokens caducados o desinstalados. | Sergio | 4h | Por hacer |
| `ES-198` | `ST-36.3` | **BD/Backend: Entidad y persistencia de Push Tokens en PostgreSQL:**<br>• Diseñar tabla y entidad TypeORM `push_tokens` vinculada a `users` y `device_id` con timestamp de último uso.<br>• Crear repositorio para registro único, actualización atómica y revocación al cerrar sesión. | Jaime | 2h | Por hacer |

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
| `ES-161` | `ST-78.1` | **Mobile Native: Módulo unificado de geonavegación externa en React Native:**<br>• Diseñar modal selector responsivo para elegir entre Waze, Google Maps o Apple Maps.<br>• Construir URL schemes robustos parametrizados con coordenadas de destino y nombres de vía.<br>• Añadir persistencia de preferencia de aplicación favorita del conductor en `AsyncStorage`.<br>• Implementar fallback resiliente a navegador web estándar si no hay app instalada. | Jaime | 4h | Por hacer |

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
    **entonces** la orden transiciona al estado `RECOGIDO_EN_TRANSITO`, emite el acuse digital formal de retiro y queda registrada para su posterior entrega en el depósito central.
  * **Escenario 3 (Recepción final y cierre en almacén central):**  
    **Dado** que el repartidor arriba al centro de distribución con la mercadería retirada,  
    **cuando** el encargado de almacén valida físicamente el paquete contra el ticket RMA,  
    **entonces** se transiciona la orden a `DEVUELTO_ALMACEN` liberando la custodia del chofer y cerrando el ciclo logístico inverso.

| ID Jira | Subtarea | Descripción Técnica y Tareas a Realizar | Asignado (PE) | Estimación | Estado |
| :--- | :--- | :--- | :---: | :---: | :---: |
| `ES-162` | `ST-80.1` | **Mobile UI/Flow: Flujo especializado de recolección inversa en la app móvil con checklist:**<br>• Diseñar tarjetas visualmente distintivas (borde ámbar e ícono de retorno) para diferenciar órdenes de entrega vs recojo de devolución.<br>• Construir formulario interactivo con los 3 checkboxes de inspección física y campo de observaciones multilínea.<br>• Integrar captura obligatoria de firma digital del cliente y fotografía del estado físico reutilizando el módulo POD (`ST-31.1`, `ST-31.2`). | Joan | 5h | Por hacer |
| `ES-163` | `ST-80.2` | **Backend: Endpoints transaccionales para gestión de retiro y confirmación de ingreso a almacén:**<br>• Implementar endpoint `POST /api/v1/dispatch/:id/reverse-pickup` para transicionar la orden a `RECOGIDO_EN_TRANSITO` con payload de checklist y firma.<br>• Implementar endpoint `POST /api/v1/dispatch/:id/warehouse-return` para marcar el ingreso final a depósito (`DEVUELTO_ALMACEN`).<br>• Disparar evento de webhook hacia Almacén e Inventarios notificando la custodia física. | Sergio | 4h | Por hacer |

---

### ÉPICA 5.0 — Panel Administrativo de Coordinación y Despacho (`ES-8`)

#### `ES-48`: 5.1 Como Coordinador de logística, necesito visualizar la bandeja central de pedidos consolidados pendientes de despacho [RF-A40]
* **Épica:** 5.0 Panel Administrativo de Coordinación y Despacho (`ES-8`)
* **Asignado (PE):** Pardo | **Story Points (Spe):** 3 | **Estado:** Por hacer (To Do)
* **Fechas:** Inicio (FI): 05 de Oct | Vencimiento (FV): 06 de Oct
* **Dependencias:** Bloquea a (B): `ES-51` | Bloqueado por (BP): `ES-14 (✅ Finalizada en Sprint 1)`

> **Como** Coordinador de Logística  
> **Quiero** acceder a una bandeja central estructurada con todos los pedidos listos y pendientes de asignación, visualizando reservas concurrentes de otros operadores y pedidos pendientes de geocodificación  
> **Para** revisar la carga entrante, evitar colisiones operativas con otros coordinadores y organizar las rutas del turno de forma eficiente.

* **Criterios de Aceptación (GWT):**
  * **Escenario 1 (Visualización de la bandeja central con filtros rápidos y estados - RF-A40):**  
    **Dado** un usuario con rol `COORDINADOR` autenticado en el portal web,  
    **cuando** accede al módulo de Despachos,  
    **entonces** visualiza la tabla de pedidos consolidados en estado `PENDIENTE` con columnas estructuradas (N° de Orden, Cliente, Dirección, Zona Logística, Ventana Horaria, Bultos, Peso Total y Badge de Urgencia) y filtros rápidos por zona y turno.
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
| `ES-164` | `ST-48.1` | **Frontend Web: Bandeja central de pedidos con TanStack Table, filtros y badges de reserva:**<br>• Construir tabla interactiva con `@tanstack/react-table` y componentes shadcn/ui (`DataTable`).<br>• Implementar columnas con ordenamiento server-side: N° Orden, Destinatario, Zona, Franja, Peso (kg), Volumen ($m^3$) y Prioridad.<br>• Diseñar badges cromáticos para pedidos bloqueados por reserva concurrente (`EN_PLANIFICACION`) y pedidos con dirección observada (`REVISION_DIRECCION`).<br>• Crear barra superior de herramientas con buscador reactivo (debounce 300ms), filtro por zona logística y botón de refresco. | Pardo | 4h | Por hacer |
| `ES-197` | `ST-48.2` | **Backend: Endpoint paginado `GET /api/v1/dispatch/pending` con filtros y bloqueo suave:**<br>• Implementar endpoint en NestJS para consultar órdenes disponibles para asignación (`status IN ('PENDING', 'REPROGRAMMED_TODAY')`).<br>• Incluir campos relacionales de usuario reservante en caso de bloqueo temporal por planificación concurrente.<br>• Añadir soporte de filtrado server-side por `zone_id`, `shift`, `priority` y búsqueda difusa por cliente/guía con paginación estándar `{ data, total, page, limit }`. | Joan | 4h | Por hacer |

---

#### `ES-49`: 5.2 Como Coordinador de despacho, requiero que el sistema genere automáticamente órdenes de despacho desde pedidos listos en almacén [RF-A02]
* **Épica:** 5.0 Panel Administrativo de Coordinación y Despacho (`ES-8`)
* **Asignado (PE):** Sergio | **Story Points (Spe):** 5 | **Estado:** Por hacer (To Do)
* **Fechas:** Inicio (FI): 08 de Oct | Vencimiento (FV): 11 de Oct
* **Dependencias:** Bloquea a (B): `ES-58`, `ES-50` | Bloqueado por (BP): `ES-72`, `ES-57`

> **Como** Coordinador de Despacho  
> **Quiero** que el sistema transforme automáticamente los avisos de pedidos listos recibidos desde Almacén en órdenes de despacho consolidadas con geocodificación y zona asignada  
> **Para** eliminar la digitación manual y disponer de las órdenes de forma inmediata en la bandeja de asignación.

* **Criterios de Aceptación (GWT):**
  * **Escenario 1 (Conversión de pedido preparado en orden de despacho - RF-A02):**  
    **Dado** un payload válido emitido por Almacén en el endpoint de ingesta,  
    **cuando** el procesador ejecuta la transformación,  
    **entonces** se genera una orden en estado `PENDIENTE` asignándole zona según geocodificación de dirección y cálculo de ventana horaria viable.
  * **Escenario 2 (Aplicación de reglas operativas basadas en ES-57):**  
    **Dado** un aviso de despacho entrante,  
    **cuando** se calcula la ventana de atención del pedido,  
    **entonces** el sistema aplica los parámetros de turnos (mañana/tarde) y tiempos máximos de espera configurados en `ES-57`, asignando la fecha límite calculada de SLA.
  * **Escenario 3 (Manejo de direcciones fuera de cobertura o no geocodificables):**  
    **Dado** un pedido cuya dirección textual no pueda ser resuelta automáticamente por el geocodificador o caiga fuera de las zonas logísticas registradas (`ES-21`),  
    **cuando** se intenta la generación automática,  
    **entonces** la orden se registra en estado `REVISION_DIRECCION` emitiendo una alerta en la bandeja del Coordinador para fijar el punto en el mapa de forma manual.

| ID Jira | Subtarea | Descripción Técnica y Tareas a Realizar | Asignado (PE) | Estimación | Estado |
| :--- | :--- | :--- | :---: | :---: | :---: |
| `ES-165` | `ST-49.1` | **Backend: Servicio transformador, geocodificador y persistencia de órdenes automáticas:**<br>• Crear servicio de conversión de esquemas ERP a entidades TypeORM `Dispatch` y `DispatchItem`.<br>• Conectar con servicio de geocodificación espacial para mapear coordenadas y asignar `zone_id` según polígonos GeoJSON del catálogo.<br>• Implementar fallback automático a estado `REVISION_DIRECCION` si la geocodificación no alcanza umbral de certeza (score < 0.7).<br>• Generar código de tracking único alfanumérico inmutable (ej. `DSP-2026-XXXXX`). | Sergio | 6h | Por hacer |
| `ES-166` | `ST-49.2` | **BD: Tablas, índices espaciales y disparadores de trazabilidad en PostgreSQL:**<br>• Crear índices espaciales GiST en PostgreSQL para columnas de latitud/longitud y polígonos de zonas logísticas.<br>• Configurar triggers de auditoría inicial para registrar timestamp de ingesta y usuario emisor.<br>• Añadir constraints relacionales para integridad referencial con catálogos maestros. | Jaime | 3h | Por hacer |

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
    **Dado** un lote de pedidos cuya sumatoria de kilogramos o metros cúbicos excede la capacidad nominal del vehículo seleccionado (`Vehicle.max_weight_kg` o `Vehicle.max_volume_m3`),  
    **cuando** el coordinador intenta confirmar la consolidación de la ruta,  
    **entonces** el sistema bloquea la operación emitiendo una alerta modal detallando el exceso exacto en kg y $m^3$.
  * **Escenario 2 (Cálculo de ruta preliminar y franjas viables para comunicación al cliente):**  
    **Dado** un conjunto de pedidos seleccionados para entrega el día siguiente,  
    **cuando** se ejecuta el optimizador de ruta preliminar,  
    **entonces** el algoritmo agrupa por cercanía minimizando tiempos de viaje y asigna a cada parada una franja horaria viable (ej. `09:00 - 11:00`), dejando la orden lista para la notificación previa al cliente (`ES-40`).
  * **Escenario 3 (Barra de progreso visual de capacidad vehicular en tiempo real):**  
    **Dado** el panel de planificación de carga,  
    **cuando** el usuario añade o quita pedidos del paquete,  
    **entonces** una barra de progreso interactiva recalcula al instante los porcentajes utilizados de peso y volumen frente a la capacidad máxima del vehículo seleccionado.

| ID Jira | Subtarea | Descripción Técnica y Tareas a Realizar | Asignado (PE) | Estimación | Estado |
| :--- | :--- | :--- | :---: | :---: | :---: |
| `ES-167` | `ST-50.1` | **Backend: Algoritmo de validación de cubicaje y peso máximo vehicular en PostgreSQL:**<br>• Implementar servicio de cálculo que totalice `weight_kg` y `volume_m3` de los despachos asociados a una ruta.<br>• Validar estrictamente contra las capacidades de la entidad `Vehicle` bloqueando asignaciones que superen el 100% de tolerancia.<br>• Retornar respuesta descriptiva con métricas: peso total, volumen total, porcentaje de ocupación y margen disponible. | Sergio | 6h | Por hacer |
| `ES-168` | `ST-50.2` | **Backend: Servicio clusterizador espacial y cálculo de rutas preliminares con OSRM:**<br>• Desarrollar servicio que agrupe órdenes por proximidad geográfica dentro de una misma zona logística.<br>• Integrar cálculo de ruta preliminar contra el motor OSRM para ordenar paradas preliminares y estimar franjas horarias viables.<br>• Exponer endpoint `POST /api/v1/dispatch/optimize/preview` con previsualización del recorrido y franjas sugeridas. | Joan | 5h | Por hacer |

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
    **entonces** el backend vincula todas las órdenes a la ruta en una única transacción, transiciona su estado a `ASIGNADO`, genera la numeración secuencial de paradas (`stop_number`) y emite las notificaciones push al móvil.
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
| `ES-169` | `ST-51.1` | **Frontend Web: Drawer interactivo de asignación de ruta completa con sugerencias de flota:**<br>• Construir drawer con tabla de pedidos incluidos en la ruta, totalizadores de peso/volumen y barra de llenado.<br>• Implementar selector inteligente que destaque a los 3 choferes recomendados con sus vehículos emparejados del turno actual.<br>• Manejar mutación con feedback visual, toasts de error ante conflictos HTTP 409 y actualización reactiva de la bandeja. | Pardo | 5h | Por hacer |
| `ES-170` | `ST-51.2` | **Backend: Transacción atómica `POST /api/v1/dispatch/assign-route` con bloqueo optimista:**<br>• Implementar transacción de base de datos (`QueryRunner`) en NestJS para asociar la ruta a los pedidos y asignar `driver_id`, `vehicle_id` y `stop_number`.<br>• Validar control de concurrencia optimista (`@VersionColumn`) impidiendo que pedidos reservados o asignados por otro hilo sean sobrescritos.<br>• Disparar evento de dominio `route.assigned` que emite notificación push masiva al repartidor y actualiza el socket de la torre de control. | Sergio | 5h | Por hacer |

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
    **Dado** un pedido en estado `INCIDENCIA` o `NO_ENTREGADO`,  
    **cuando** el coordinador define una nueva fecha de entrega pactada y selecciona la causal desde el catálogo (`ES-25`),  
    **entonces** el pedido transiciona a estado `REPROGRAMADO`, incrementa su contador de reintentos (`attempt_count`), se desvincula de la ruta actual y se oculta de la lista de pendientes del día.
  * **Escenario 2 (Reactivación automática en la fecha pactada de entrega):**  
    **Dado** un pedido en estado `REPROGRAMADO` cuya fecha pactada coincide con el día de la jornada activa,  
    **cuando** el coordinador consulta la bandeja de despachos del día,  
    **entonces** el sistema incluye automáticamente el pedido en la cola de asignación con un badge ámbar destacado: `REPROGRAMADO (Reintento 2/3)` para que sea planificado en la nueva ruta.
  * **Escenario 3 (Control de umbral máximo de reintentos configurado en ES-57):**  
    **Dado** un pedido que alcanza el límite máximo de reintentos parametrizado en el sistema (ej. 3 intentos fallidos),  
    **cuando** se intenta una nueva reprogramación,  
    **entonces** el sistema bloquea la acción emitiendo una alerta crítica de "Límite de reintentos alcanzado" y fuerza la derivación del pedido a devolución formal hacia Almacén (`RMA`).

| ID Jira | Subtarea | Descripción Técnica y Tareas a Realizar | Asignado (PE) | Estimación | Estado |
| :--- | :--- | :--- | :---: | :---: | :---: |
| `ES-171` | `ST-52.1` | **Frontend Web: Modal de reprogramación con historial de intentos y fotos de evidencia:**<br>• Diseñar diálogo interactivo mostrando las causales previas de no entrega, notas del conductor y fotos de fachada.<br>• Integrar selector de nueva fecha pactada (`DatePicker`), turno y campo para instrucciones especiales de reintento.<br>• Incorporar badge con el número de intento actual (`Intento 1 de 3`) y alerta preventiva si es el último reintento permitido. | Pardo | 4h | Por hacer |
| `ES-172` | `ST-52.2` | **Backend: Endpoint transaccional `POST /api/v1/dispatch/:id/reschedule` y reglas de umbral:**<br>• Actualizar estado a `REPROGRAMADO`, persistir `reprogrammed_date` e incrementar el contador `attempt_count` en PostgreSQL.<br>• Validar el parámetro `max_reschedule_attempts` configurado en `ES-57`; si se supera, rechazar con HTTP 400 exigiendo derivación a RMA.<br>• Registrar evento inmutable en la bitácora histórica de auditoría y notificar al cliente el nuevo compromiso de entrega. | Joan | 4h | Por hacer |

---

#### `ES-53`: 5.6 Como Coordinador de despacho, requiero monitorear el avance de las entregas en un tablero Kanban interactivo sincronizado en tiempo real [RF-A06]
* **Épica:** 5.0 Panel Administrativo de Coordinación y Despacho (`ES-8`)
* **Asignado (PE):** Pardo | **Story Points (Spe):** 5 | **Estado:** Por hacer (To Do)
* **Fechas:** Inicio (FI): 16 de Oct | Vencimiento (FV): 17 de Oct
* **Dependencias:** Bloquea a (B): `ES-54` | Bloqueado por (BP): `ES-51`

> **Como** Coordinador de Despachos  
> **Quiero** monitorear en un tablero visual tipo Kanban las órdenes distribuidas en sus etapas operativas completas (`Pendiente`, `En Planificación`, `Programado`, `Asignado`, `En Ruta`, `Entregado`, `Incidencia`) con sincronización reactiva Socket.io y TanStack Query  
> **Para** obtener una panorámica inmediata del avance de la jornada, prevenir colisiones entre coordinadores y reaccionar al instante ante contingencias en ruta.

* **Criterios de Aceptación (GWT):**
  * **Escenario 1 (Distribución estructurada en columnas del flujo operativo completo - RF-A06):**  
    **Dado** el tablero de control de despacho,  
    **cuando** se cargan las órdenes del día,  
    **entonces** se organizan en las columnas normalizadas: `Pendiente`, `En Planificación`, `Programado`, `Asignado`, `En Ruta`, `Entregado` e `Incidencia`, con contadores totales de órdenes y volumen por columna.
  * **Escenario 2 (Interacción guiada y protección contra asignación arbitraria por arrastre):**  
    **Dado** un pedido en columna `Pendiente` o `En Planificación`,  
    **cuando** el coordinador arrastra la tarjeta hacia la columna `Asignado`,  
    **entonces** la interfaz no ejecuta una asignación a ciegas sino que abre el drawer de asignación de ruta (`ES-51`) validando capacidad y chofer antes de confirmar la transición.
  * **Escenario 3 (Actualización reactiva híbrida mediante Socket.io y TanStack Query):**  
    **Dado** que un repartidor actualiza un estado a `EN_CAMINO` o `ENTREGADO` en su app móvil,  
    **cuando** el backend procesa el cambio,  
    **entonces** emite un evento por Socket.io (`dispatch.updated`), el frontend invalida la consulta de TanStack Query (`invalidateQueries`) y la tarjeta se desplaza suavemente a su nueva columna sin requerir recargar la página web, respaldado por un polling pasivo de seguridad cada 15 segundos.

| ID Jira | Subtarea | Descripción Técnica y Tareas a Realizar | Asignado (PE) | Estimación | Estado |
| :--- | :--- | :--- | :---: | :---: | :---: |
| `ES-173` | `ST-53.1` | **Frontend Web: Tablero Kanban interactivo con drag-and-drop y visualización de etapas:**<br>• Construir tablero Kanban modular en React 19 con `@dnd-kit/core` y Tailwind CSS v4.<br>• Diseñar las 7 columnas del flujo con scroll independiente y contadores de cabecera.<br>• Diseñar tarjetas visuales enriquecidas: código de orden, transportista/vehículo asignado, zona logística, franja horaria comprometida y badge de prioridad.<br>• Integrar barra de filtros reactivos por zona, conductor y tipo de servicio en el encabezado. | Pardo | 6h | Por hacer |
| `ES-174` | `ST-53.2` | **Frontend/Backend: Sincronización en vivo con Socket.io Gateway y TanStack Query cache invalidation:**<br>• Configurar WebSocket Gateway en NestJS (`DispatchEventsGateway`) que emita eventos ante cambios de estado de órdenes y rutas.<br>• Implementar hook de React `useDispatchEvents` que escuche los eventos vía Socket.io client e invoque `queryClient.invalidateQueries({ queryKey: ['dispatches-kanban'] })`.<br>• Configurar polling de respaldo pasivo (`refetchInterval: 15000`) como mecanismo de contingencia ante desconexiones de red. | Sergio | 4h | Por hacer |

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
| `ES-175` | `ST-54.1` | **Frontend Web: Módulo visual de reportes con filtros y exportación:**<br>• Diseñar pantalla con gráficos de cumplimiento de entregas (% a tiempo vs demoras).<br>• Añadir botón de exportación rápida a formato CSV/Excel en el cliente. | Pardo | 4h | Por hacer |
| `ES-176` | `ST-54.2` | **Backend: Endpoint `GET /api/v1/dispatch/reports/sla` de estadísticas de SLA:**<br>• Implementar agregaciones SQL en PostgreSQL para comparar tiempos reales vs tiempos teóricos de entrega según ventanas pactadas.<br>• Retornar métricas por zona, servicio y transportista con caché Redis (TTL 10 min). | Sergio | 3h | Por hacer |

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
| `ES-177` | `ST-55.1` | **Frontend Web: Tabla avanzada de historial con filtros combinados y paginación:**<br>• Implementar vista de tabla con filtros por fecha, repartidor, placa vehicular y resultado final.<br>• Añadir enlace directo a la vista unificada 360° de cada despacho. | Pardo | 4h | Por hacer |
| `ES-178` | `ST-55.2` | **Backend: Consulta paginada y optimizada de histórico en PostgreSQL:**<br>• Crear QueryBuilder para búsquedas compuestas sobre tablas históricas con índices de fecha y conductor.<br>• Estructurar respuesta estándar `{ data, total, page }`. | Joan | 3h | Por hacer |

---

#### `ES-56`: 5.9 Como Coordinador de despacho, necesito visualizar una ficha unificada 360° con la trazabilidad y evidencias completas de la orden [RF-A09]
* **Épica:** 5.0 Panel Administrativo de Coordinación y Despacho (`ES-8`)
* **Asignado (PE):** Pardo | **Story Points (Spe):** 4 | **Estado:** Por hacer (To Do)
* **Fechas:** Inicio (FI): 19 de Oct | Vencimiento (FV): 21 de Oct
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
| `ES-179` | `ST-56.1` | **Frontend Web: Drawer/Página de vista 360° del despacho con timeline interactivo:**<br>• Desarrollar vista integral estructurada en tabs: General, Línea de Tiempo, Evidencias Digitales e Incidencias.<br>• Incorporar visor modal para inspeccionar firmas y fotografías en alta resolución. | Pardo | 5h | Por hacer |
| `ES-180` | `ST-56.2` | **Backend: Endpoint agregado `GET /api/v1/dispatch/:id/full-summary`:**<br>• Construir consulta relacional uniendo despacho, eventos, conductor y URLs de evidencias multimedia.<br>• Exponer recursos multimedia mediante rutas protegidas o URLs firmadas desde el plugin de almacenamiento.<br>• Proteger la consulta con control de acceso por roles. | Joan | 4h | Por hacer |

---

#### `ES-57`: 5.10 Como Coordinador de logística, requiero configurar los parámetros de ventanas de entrega, tiempos de espera y umbral de reintentos en servicio [RF-A10]
* **Épica:** 5.0 Panel Administrativo de Coordinación y Despacho (`ES-8`)
* **Asignado (PE):** Pardo | **Story Points (Spe):** 3 | **Estado:** Por hacer (To Do)
* **Fechas:** Inicio (FI): 05 de Oct | Vencimiento (FV): 07 de Oct
* **Dependencias:** Bloquea a (B): `ES-49` | Bloqueado por (BP): `ES-14 (✅ Finalizada en Sprint 1)`

> **Como** Coordinador de Logística  
> **Quiero** parametrizar las ventanas horarias de entrega, tiempos máximos de espera en puerta, márgenes de tolerancia y umbral máximo de intentos de reprogramación  
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
| `ES-181` | `ST-57.1` | **Frontend Web: Formulario de parametrización operativa con React Hook Form y Zod:**<br>• Diseñar vista de configuración con inputs para ventanas horarias (mañana, tarde, express), tiempos de espera y límite de reprogramaciones (`max_reschedule_attempts`).<br>• Aplicar validación numérica y toasts de confirmación. | Pardo | 4h | Por hacer |
| `ES-182` | `ST-57.2` | **Backend: Endpoints de gestión de parámetros operativos y caché Redis:**<br>• Implementar `GET` y `PUT /api/v1/dispatch/settings/operational` con DTO validado.<br>• Actualizar caché en Redis para lectura de alto rendimiento en cálculo de rutas y reprogramaciones. | Sergio | 4h | Por hacer |

---

#### `ES-58`: 5.11 Como Coordinador de despacho, necesito marcar y filtrar pedidos según su nivel de prioridad urgente o normal [RF-A11]
* **Épica:** 5.0 Panel Administrativo de Coordinación y Despacho (`ES-8`)
* **Asignado (PE):** Sergio | **Story Points (Spe):** 3 | **Estado:** Por hacer (To Do)
* **Fechas:** Inicio (FI): 12 de Oct | Vencimiento (FV): 13 de Oct
* **Dependencias:** Bloquea a (B): `ES-51` | Bloqueado por (BP): `ES-49`

> **Como** Coordinador de Logística  
> **Quiero** visualizar etiquetas cromáticas de prioridad (`URGENTE`, `NORMAL`) y aplicar filtros multifactoriales en el panel web  
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
| `ES-183` | `ST-58.1` | **Frontend Web: Componentes de insignias de prioridad y filtros combinados:**<br>• Desarrollar badges visuales (Rojo: Urgente, Gris: Normal) con selector de cambio rápido de prioridad.<br>• Implementar barra de filtros dinámicos (Zona, Prioridad, Turno) con actualización fluida. | Sergio | 4h | Por hacer |
| `ES-184` | `ST-58.2` | **Backend: Soporte para campo de prioridad y ordenamiento en consultas REST:**<br>• Añadir columna `priority` en entidad de despachos (`LOW`, `NORMAL`, `URGENT`).<br>• Implementar lógica de ordenamiento en QueryBuilder priorizando despachos urgentes por defecto. | Joan | 3h | Por hacer |

---

#### `ES-81`: 5.12 Como Coordinador de despacho, requiero registrar y programar órdenes de recojo domiciliario para devoluciones o garantías [RF-A39]
* **Épica:** 5.0 Panel Administrativo de Coordinación y Despacho (`ES-8`)
* **Asignado (PE):** Pardo | **Story Points (Spe):** 3 | **Estado:** Por hacer (To Do)
* **Fechas:** Inicio (FI): 20 de Oct | Vencimiento (FV): 22 de Oct
* **Dependencias:** Bloquea a (B): Ninguna | Bloqueado por (BP): `ES-51`

> **Como** Coordinador de Logística  
> **Quiero** registrar y programar órdenes de recojo en domicilio para devoluciones o garantías aprobadas  
> **Para** gestionar la logística inversa desde el cliente hacia el almacén central con los mismos estándares de asignación y trazabilidad.

* **Criterios de Aceptación (GWT):**
  * **Escenario 1 (Creación de orden de recojo domiciliario - RF-A39):**  
    **Dado** un reclamo o devolución autorizada,  
    **cuando** el coordinador registra la orden de recojo con dirección del cliente, fecha y descripción de mercadería,  
    **entonces** el sistema crea la orden en estado `RECOJO_PROGRAMADO` lista para asignarse a la ruta de un repartidor.
  * **Escenario 2 (Validación de motivo de devolución y cancelación previa a ruta):**  
    **Dado** un pedido de recojo programado pero no despachado aún,  
    **cuando** el cliente desiste de la garantía o cancela la solicitud,  
    **entonces** el coordinador puede anular la orden registrando la causal respectiva liberando la programación de la flota.

| ID Jira | Subtarea | Descripción Técnica y Tareas a Realizar | Asignado (PE) | Estimación | Estado |
| :--- | :--- | :--- | :---: | :---: | :---: |
| `ES-185` | `ST-81.1` | **Frontend Web: Formulario de programación de órdenes de retiro domiciliario:**<br>• Diseñar formulario modal para registro de recojos (cliente, teléfono, dirección, motivo de devolución).<br>• Integrar validación con React Hook Form y confirmación visual. | Pardo | 4h | Por hacer |
| `ES-186` | `ST-81.2` | **Backend: Endpoint `POST /api/v1/dispatch/reverse-logistics/pickup`:**<br>• Implementar servicio en NestJS para crear órdenes marcadas como logística inversa (`direction: RETURN`).<br>• Integrar validación con los catálogos de motivos de devolución (`ES-25`). | Sergio | 4h | Por hacer |

---

### ÉPICA 8.0 — Interoperabilidad con el ERP y Sistemas Externos (`ES-11`)

#### `ES-72`: 8.1 Como Desarrollador de integraciones, requiero exponer un endpoint REST seguro para recibir pedidos preparados desde Almacén [ERP-01]
* **Épica:** 8.0 Interoperabilidad con el ERP y Sistemas Externos (`ES-11`)
* **Asignado (PE):** Sergio | **Story Points (Spe):** 5 | **Estado:** Por hacer (To Do)
* **Fechas:** Inicio (FI): 05 de Oct | Vencimiento (FV): 08 de Oct
* **Dependencias:** Bloquea a (B): `ES-73`, `ES-49` | Bloqueado por (BP): `ES-14 (✅ Finalizada en Sprint 1)`

> **Como** Desarrollador de Integraciones Backend  
> **Quiero** exponer un endpoint REST seguro y documentado para recibir órdenes preparadas desde el microservicio de Inventarios y Almacén  
> **Para** alimentar automáticamente el flujo de despachos con los paquetes físicos listos para salir a ruta.

* **Criterios de Aceptación (GWT):**
  * **Escenario 1 (Recepción exitosa de pedido preparado - ERP-01):**  
    **Dado** un microservicio de Almacén autenticado mediante credenciales de servicio,  
    **cuando** envía un JSON a `POST /api/v1/erp/orders/ready` con el id de pedido, peso, volumen, ventana y destinatario,  
    **entonces** el microservicio responde con código 201 Created y el identificador de despacho asignado.
  * **Escenario 2 (Validación estricta de esquema y rechazo ante campos obligatorios faltantes):**  
    **Dado** un payload incompleto o con datos inválidos,  
    **cuando** el endpoint procesa la solicitud,  
    **entonces** responde con código 422 Unprocessable Entity detallando los campos faltantes.
  * **Escenario 3 (Manejo idempotente ante reenvíos de la misma orden):**  
    **Dado** un aviso de pedido preparado ya registrado previamente en el sistema,  
    **cuando** Almacén reenvía la misma carga por reintento de red,  
    **entonces** el endpoint detecta el identificador externo existente y responde de forma idempotente con HTTP 200 OK retornando la orden original sin generar duplicados.

| ID Jira | Subtarea | Descripción Técnica y Tareas a Realizar | Asignado (PE) | Estimación | Estado |
| :--- | :--- | :--- | :---: | :---: | :---: |
| `ES-187` | `ST-72.1` | **Backend: Endpoint REST `POST /api/v1/erp/orders/ready` y validación DTO:**<br>• Crear controlador y servicio en NestJS para la ingesta de órdenes de almacén.<br>• Diseñar `CreateErpOrderReadyDto` con validaciones estrictas (`class-validator`) para pesos, dimensiones y direcciones.<br>• Manejar idempotencia transaccional ante reenvíos de la misma orden. | Sergio | 6h | Por hacer |
| `ES-188` | `ST-72.2` | **Backend: Middleware de autenticación M2M y autorización de servicios ERP:**<br>• Implementar guard de autenticación M2M (`M2mAuthGuard`) con validación de API Keys específicas por servicio.<br>• Registrar intentos de comunicación externa y accesos inter-servicios en logs estructurados. | Joan | 4h | Por hacer |

---

#### `ES-73`: 8.2 Como Operador de despacho / Integrador de sistemas, requiero emitir la notificación de salida física hacia Almacén e Inventarios [ERP-02]
* **Épica:** 8.0 Interoperabilidad con el ERP y Sistemas Externos (`ES-11`)
* **Asignado (PE):** Sergio | **Story Points (Spe):** 4 | **Estado:** Por hacer (To Do)
* **Fechas:** Inicio (FI): 09 de Oct | Vencimiento (FV): 13 de Oct
* **Dependencias:** Bloquea a (B): `ES-76` | Bloqueado por (BP): `ES-72`

> **Como** Operador de despacho / Integrador de sistemas  
> **Quiero** notificar al microservicio de Inventarios y Almacén en el instante en que el vehículo sale físicamente del centro de distribución  
> **Para** que Almacén actualice el estado de los bultos a "En Tránsito" y libere formalmente la custodia de la mercadería sin bloquear la salida del transportista.

* **Criterios de Aceptación (GWT):**
  * **Escenario 1 (Notificación exitosa de salida física - ERP-02):**  
    **Dado** que el vehículo inicia su ruta y abandona la garita del CEDIS,  
    **cuando** el sistema procesa la salida de almacén,  
    **entonces** invoca síncronamente el endpoint de Almacén confirmando la partida y registrando la respuesta 200 en auditoría.
  * **Escenario 2 (Tolerancia a fallos no bloqueante mediante cola de reintentos):**  
    **Dado** que el microservicio de Almacén presenta intermitencia o timeout (> 2.5s),  
    **cuando** la confirmación directa no recibe respuesta inmediata,  
    **entonces** el sistema registra la salida física localmente permitiendo el avance de la ruta en calle y encola el evento en una cola de reintentos con backoff exponencial.

| ID Jira | Subtarea | Descripción Técnica y Tareas a Realizar | Asignado (PE) | Estimación | Estado |
| :--- | :--- | :--- | :---: | :---: | :---: |
| `ES-189` | `ST-73.1` | **Backend: Cliente HTTP y servicio de confirmación síncrona hacia Almacén:**<br>• Implementar servicio con `@nestjs/axios` para enviar payload de salida física de pedidos.<br>• Añadir política de timeouts ajustados (< 2.5s) para no bloquear la operación física de despacho. | Sergio | 5h | Por hacer |
| `ES-190` | `ST-73.2` | **Backend: Cola de contingencia y reintentos ante caídas de Almacén:**<br>• Configurar tabla o cola Redis para reintentar la notificación ante errores 5xx del microservicio externo.<br>• Alertar a la auditoría técnica en caso de fallos definitivos tras 3 reintentos. | Joan | 4h | Por hacer |

---

#### `ES-74`: 8.3 Como Desarrollador de integraciones, requiero interfaces REST para recibir solicitudes de transporte y logística inversa desde Compras [ERP-03]
* **Épica:** 8.0 Interoperabilidad con el ERP y Sistemas Externos (`ES-11`)
* **Asignado (PE):** Sergio | **Story Points (Spe):** 4 | **Estado:** Por hacer (To Do)
* **Fechas:** Inicio (FI): 05 de Oct | Vencimiento (FV): 09 de Oct
* **Dependencias:** Bloquea a (B): `ES-75` | Bloqueado por (BP): `ES-14 (✅ Finalizada en Sprint 1)`

> **Como** Desarrollador de Integraciones Backend  
> **Quiero** implementar las interfaces REST para recibir solicitudes de transporte y logística inversa desde Gestión de Compras y Proveedores  
> **Para** atender transferencias de mercadería y devoluciones hacia proveedores externos de forma automatizada.

* **Criterios de Aceptación (GWT):**
  * **Escenario 1 (Recepción de orden de transporte de Compras - ERP-03):**  
    **Dado** un requerimiento de transporte emitido por Compras,  
    **cuando** se consume `POST /api/v1/erp/procurement/orders`,  
    **entonces** el microservicio valida el contrato y genera la orden de recojo correspondiente.
  * **Escenario 2 (Rechazo y tipificación ante cargas o direcciones inválidas):**  
    **Dado** una orden de compras con pesos negativos, fechas pasadas o sin punto de retiro en proveedor,  
    **cuando** el endpoint procesa la solicitud,  
    **entonces** rechaza la petición con HTTP 422 Unprocessable Entity especificando los errores de esquema.
  * **Escenario 3 (Autenticación M2M obligatoria):**  
    **Dado** una llamada externa que omite la API Key o presenta credenciales M2M inválidas,  
    **cuando** intenta consumir la interfaz de Compras,  
    **entonces** el `M2mAuthGuard` rechaza la solicitud retornando código HTTP 401 Unauthorized.

| ID Jira | Subtarea | Descripción Técnica y Tareas a Realizar | Asignado (PE) | Estimación | Estado |
| :--- | :--- | :--- | :---: | :---: | :---: |
| `ES-191` | `ST-74.1` | **Backend: Endpoint REST `POST /api/v1/erp/procurement/orders`:**<br>• Crear controlador y DTOs para la recepción de requerimientos del módulo de Compras.<br>• Validar puntos de retiro, datos de contacto del proveedor y especificaciones de carga.<br>• Proteger el controlador con `M2mAuthGuard`. | Sergio | 5h | Por hacer |
| `ES-192` | `ST-74.2` | **BD: Modelado relacional para operaciones de Compras y Proveedores:**<br>• Crear relaciones y campos en PostgreSQL para tipificar despachos originados por compras externas.<br>• Aplicar migraciones TypeORM limpias y versionadas. | Jaime | 3h | Por hacer |

---

#### `ES-75`: 8.4 Como Coordinador de transporte, necesito programar y gestionar el recojo de mercadería en proveedores hacia almacenes centrales [ERP-04]
* **Épica:** 8.0 Interoperabilidad con el ERP y Sistemas Externos (`ES-11`)
* **Asignado (PE):** Joan | **Story Points (Spe):** 4 | **Estado:** Por hacer (To Do)
* **Fechas:** Inicio (FI): 12 de Oct | Vencimiento (FV): 15 de Oct
* **Dependencias:** Bloquea a (B): `ES-76` | Bloqueado por (BP): `ES-74`

> **Como** Coordinador de Transporte  
> **Quiero** coordinar y registrar el recojo de mercadería en instalaciones de proveedores para su posterior traslado a tiendas o almacenes centrales  
> **Para** integrar los flujos de abastecimiento externo en la programación diaria de la flota vehicular.

* **Criterios de Aceptación (GWT):**
  * **Escenario 1 (Programación de recojo en proveedor - ERP-04):**  
    **Dado** un pedido de compras validado,  
    **cuando** se agenda el retiro en las instalaciones del proveedor,  
    **entonces** el sistema calcula la ventana de atención y lo integra en la planificación operativa del Coordinador.
  * **Escenario 2 (Reprogramación ante rechazo o indisponibilidad del proveedor):**  
    **Dado** que el proveedor no cuenta con la carga lista al momento pactado,  
    **cuando** se reporta la incidencia,  
    **entonces** el coordinador puede reprogramar la fecha de retiro emitiendo un evento de actualización hacia Compras.

| ID Jira | Subtarea | Descripción Técnica y Tareas a Realizar | Asignado (PE) | Estimación | Estado |
| :--- | :--- | :--- | :---: | :---: | :---: |
| `ES-193` | `ST-75.1` | **Backend: Servicio de programación de recojo en proveedores externos:**<br>• Implementar servicio de lógica de negocio para crear órdenes de retiro en instalaciones de terceros.<br>• Notificar de vuelta a Compras la confirmación de la fecha y franja horaria programada. | Joan | 5h | Por hacer |
| `ES-194` | `ST-75.2` | **Backend: Pruebas de integración E2E del circuito de Compras $\leftrightarrow$ Despachos:**<br>• Construir tests en Jest/Supertest simulando el flujo completo de solicitud, recojo y confirmación mediante stubs/mocks de API. | Sergio | 3h | Por hacer |

---

#### `ES-76`: 8.5 Como Arquitecto de integraciones, requiero publicar la documentación OpenAPI/Swagger de los contratos para el API Gateway [ERP-05]
* **Épica:** 8.0 Interoperabilidad con el ERP y Sistemas Externos (`ES-11`)
* **Asignado (PE):** Joan | **Story Points (Spe):** 3 | **Estado:** Por hacer (To Do)
* **Fechas:** Inicio (FI): 20 de Oct | Vencimiento (FV): 21 de Oct *(Ajustado a 2 días hábiles)*
* **Dependencias:** Bloquea a (B): Ninguna | Bloqueado por (BP): `ES-73`, `ES-75`

> **Como** Arquitecto de Integraciones  
> **Quiero** acceder a la especificación OpenAPI / Swagger completamente documentada de los endpoints de interoperabilidad  
> **Para** integrar de forma estándar el API Gateway corporativo y validar los contratos de datos sin ambigüedades.

* **Criterios de Aceptación (GWT):**
  * **Escenario 1 (Acceso a documentación interactiva Swagger - ERP-05):**  
    **Dado** el microservicio de Despachos en ejecución,  
    **cuando** se consulta `/api/v1/docs`,  
    **entonces** se despliega la interfaz Swagger UI con todos los esquemas, ejemplos de petición/respuesta y códigos HTTP normalizados.
  * **Escenario 2 (Exportación del contrato estático OpenAPI para API Gateway):**  
    **Dado** el comando de exportación de contratos,  
    **cuando** se ejecuta el script de compilación,  
    **entonces** se genera el archivo estático `openapi.json` con las especificaciones de seguridad Bearer y API Key listo para el Gateway corporativo.

| ID Jira | Subtarea | Descripción Técnica y Tareas a Realizar | Asignado (PE) | Estimación | Estado |
| :--- | :--- | :--- | :---: | :---: | :---: |
| `ES-195` | `ST-76.1` | **Backend: Decoración exhaustiva con decoradores Swagger (`@nestjs/swagger`):**<br>• Documentar todos los endpoints de interoperabilidad con `@ApiOperation`, `@ApiResponse` y `@ApiProperty`.<br>• Definir ejemplos reales de payloads JSON para Almacén y Compras. | Joan | 4h | Por hacer |
| `ES-196` | `ST-76.2` | **Backend: Generación de archivo estático OpenAPI JSON/YAML para el API Gateway:**<br>• Configurar script para compilar y exportar el esquema OpenAPI estático listo para el Gateway central.<br>• Validar cumplimiento de estándares de seguridad y tipificación. | Sergio | 3h | Por hacer |

---

## 3. Matriz Resumen de Carga Horaria y Capacidad — Sprint 2

| Desarrollador | Roles Rotados en Sprint 2 | Subtareas Asignadas (Sprint 2) | Horas Totales Estimadas |
| :--- | :--- | :--- | :---: |
| **Jaime** | Mobile Developer & DBA | `ST-28.1` (6h), `ST-28.3` (4h), `ST-29.1` (6h), `ST-29.3` (4h), `ST-30.1` (5h), `ST-31.1` (5h), `ST-31.2` (5h), `ST-32.1` (5h), `ST-35.1` (3h), `ST-36.3` (2h), `ST-78.1` (4h), `ST-49.2` (3h), `ST-74.2` (3h) | **55 hrs** |
| **Sergio** | Backend Developer & Web Front | `ST-30.3` (4h), `ST-32.2` (4h), `ST-33.2` (3h), `ST-36.2` (4h), `ST-80.2` (4h), `ST-57.2` (4h), `ST-49.1` (6h), `ST-58.1` (4h), `ST-50.1` (6h), `ST-51.2` (5h), `ST-53.2` (4h), `ST-54.2` (3h), `ST-81.2` (4h), `ST-72.1` (6h), `ST-73.1` (5h), `ST-74.1` (5h), `ST-75.2` (3h), `ST-76.2` (3h) | **77 hrs** |
| **Pardo** | Frontend Web Lead | `ST-48.1` (4h), `ST-57.1` (4h), `ST-51.1` (5h), `ST-52.1` (4h), `ST-53.1` (6h), `ST-56.1` (5h), `ST-54.1` (4h), `ST-55.1` (4h), `ST-81.1` (4h) | **40 hrs** |
| **Joan** | Backend Developer & Mobile Architecture | `ST-28.2` (6h), `ST-29.2` (4h), `ST-30.2` (5h), `ST-31.3` (6h), `ST-33.1` (4h), `ST-35.2` (3h), `ST-36.1` (4h), `ST-80.1` (5h), `ST-48.2` (4h), `ST-58.2` (3h), `ST-50.2` (5h), `ST-52.2` (4h), `ST-56.2` (4h), `ST-55.2` (3h), `ST-72.2` (4h), `ST-73.2` (4h), `ST-75.1` (5h), `ST-76.1` (4h) | **77 hrs** |

> [!NOTE]
> **Balance y Gestión de Capacidad en el Sprint 2:**  
> Las estimaciones de horas representan asignaciones de tiempo holgadas para garantizar cobertura total. El margen de tiempo disponible por cada miembro del equipo se destina formalmente a la realización de Code Reviews cruzados (Sergio en Frontend/Mobile, Jaime en Backend), ceremonias de Scrum y colchón ante contingencias de integración con el ERP.
