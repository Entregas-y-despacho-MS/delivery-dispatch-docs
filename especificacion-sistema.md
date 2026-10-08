# Sistema de Gestión de Entregas y Despachos (Despachos)
**Microservicio del ERP Corporativo – Supermercado / Retailer (Grupo H)**  
*Caso de estudio: HIPERMAXI, DISMAC*

---

## 👥 Equipo del Proyecto

| Nombre Completo | Rol |
|---|---|
| **Pardo Romano Alejandro Miguel** | Líder de equipo / Product Owner |
| **Riveros Soria Joan Marcelo** | Scrum Master |
| **Huaycho Clavel Jaime Ignacio** | Dev Backend |
| **Mendoza Choque Sergio Alexander** | Dev Frontend |

*Universidad Católica Boliviana "San Pablo" — La Paz, Bolivia*  
*Jira:* [Tablero de Proyecto Jira](https://joanmarceloriverossoria.atlassian.net/jira/software/projects/ES/boards/100/timeline)

---

## 1. Resumen Ejecutivo del Sistema

### 1.a. Presentación del Sistema de Información
**Despachos** es el microservicio de Gestión de Entregas y Despachos, uno de los nueve componentes que integran el ERP corporativo de una cadena de supermercados (junto con *Compras y Proveedores*, *Inventarios y Almacén*, *Recursos Humanos*, *Pagos y Facturación*, *Contabilidad*, *CRM*, *Marketplace y Ventas*, y *Producción y Logística*).

Su responsabilidad es **exclusivamente la última milla**: desde que un pedido queda listo para repartir hasta que se confirma su entrega al cliente final. Asigna repartidores y vehículos a cada pedido confirmado, da seguimiento a la entrega hasta su confirmación y gestiona las excepciones propias del reparto (cliente ausente, dirección incorrecta, producto dañado), dejando un registro auditable de cada evento.

> [!NOTE]
> Opera como un microservicio independiente, con base de datos PostgreSQL propia y API REST propia (NestJS) para sus clientes web y móvil, sin acceder directamente a las bases de datos de otros componentes del ERP. Se integra con tres microservicios mediante **mensajería asíncrona JSON (Google Cloud Pub/Sub, simulada con su emulador local durante el proyecto)**: **Marketplace y Ventas** (datos de entrega del pedido de venta), **Gestión de Inventarios y Almacén** (bultos preparados y confirmaciones de recepción) y **Gestión de Compras y Proveedores** (solicitudes de recogida en proveedor). No procesa cobros ni interactúa directamente con Pagos y Facturación. El detalle de tópicos y contratos está en [`interoperabilidad-y-flujos.md`](./interoperabilidad-y-flujos.md).

### 1.b. Propuesta de Valor

#### Para el cliente final:
* **Visibilidad total del pedido:** Estado, ubicación del repartidor y tiempo estimado de llegada (ETA) en tiempo real.
* **Comunicación proactiva:** Notificaciones automáticas por correo electrónico ante cada cambio de estado.
* **Evidencia y confianza:** Cada entrega queda respaldada con fotografía, firma electrónica o código OTP.
* **Canal directo de retroalimentación:** Calificación del servicio y reporte de reclamos.

#### Para el negocio (HIPERMAXI / DISMAC):
* **Control centralizado y trazable:** Toda la operación de última milla sin depender de coordinación manual.
* **Indicadores de cumplimiento:** SLA y tiempos de entrega para toma de decisiones operativas.
* **Escalabilidad:** Soporte de picos de operación (fin de mes, campañas comerciales) sin degradar el servicio.
* **Integración limpia:** Comunicación asíncrona por mensajes JSON con los demás módulos del ERP, conservando sus identificadores originales sin duplicar ni comprometer sus datos.

### 1.c. Funcionalidades Principales
1. Autenticación y control de acceso por roles (`Coordinador`, `Supervisor`, `Repartidor`, `Cliente`).
2. Registro automático de órdenes de despacho a partir de pedidos confirmados.
3. Asignación de repartidor y vehículo con agrupación de pedidos por zona.
4. Seguimiento en tiempo real mediante geolocalización y cálculo de ETA.
5. Registro de evidencia digital de entrega (foto, firma, OTP).
6. Gestión de incidencias y reprogramación de entregas fallidas.
7. Notificaciones push (repartidor) y por correo electrónico (cliente).
8. Calificación del servicio y gestión de reclamos.
9. Administración de flota: disponibilidad y mantenimiento de vehículos.
10. Reportes e indicadores de desempeño (cumplimiento, SLA, incidencias).
11. Auditoría y trazabilidad completa de todas las acciones y eventos del sistema.

### 1.d. Usuarios Principales
| Perfil | Plataforma | Descripción |
|---|---|---|
| **Repartidor / Transportista** | App Móvil (React Native) | Ejecuta la entrega física, actualiza estados y transmite su ubicación GPS en ruta desde la app móvil. |
| **Cliente Final** | Portal Web (Público) | Da seguimiento a su pedido mediante enlace con token único, recibe notificaciones y califica el servicio. |
| **Coordinador de Despachos / Logística** | Portal Web (Panel Admin) | Asigna pedidos a repartidores y vehículos, supervisa el cumplimiento y administra la configuración del microservicio (usuarios, catálogos, seguridad). |
| **Supervisor de Flota** | Portal Web (Panel Admin) | Monitorea vehículos y repartidores en tiempo real, gestiona disponibilidad/mantenimiento de vehículos e indicadores de flota. |

---

## 2. Arquitectura y Tecnología del Producto

### 2.a. Modelo de Despliegue por Capas

```mermaid
flowchart TD
    subgraph CapaPresentacion["1. Capa de Presentación (Frontend)"]
        WebCliente["Portal Web Cliente\n(React + Vite)"]
        WebAdmin["Panel Administrativo\n(React + Vite)"]
        AppMobile["App Móvil Repartidor\n(React Native / Expo)"]
    end

    subgraph Gateway["API Gateway del ERP"]
        APIGateway["API Gateway Común del ERP\n(HTTPS / REST)"]
    end

    subgraph CapaAplicacionDatos["2. Capa de Aplicación y Datos"]
        Backend["Backend Despachos\n(NestJS + TypeScript)\nJWT por Rol"]
        BD[("PostgreSQL\nBase de Datos Exclusiva")]
        Cache[("Redis\nCaché y OTP")]
        Backend --> BD
        Backend --> Cache
    end

    subgraph CapaIntegraciones["3. Capa de Integraciones"]
        Broker{{"Google Cloud Pub/Sub\n(emulador local)"}}
        MSMarketplace["MS Marketplace\ny Ventas"]
        MSInventario["MS Gestión de Inventarios\ny Almacén"]
        MSCompras["MS Gestión de Compras\ny Proveedores"]
        Traccar["Traccar / GPS\n(Servidor de Rastreo)"]
        Notificaciones["Servicios Externos:\nCorreo, Push y SMS"]
        Mapas["Google Maps / Waze /\nOSRM Ruteo"]
    end

    WebCliente -->|HTTPS| APIGateway
    WebAdmin -->|HTTPS| APIGateway
    AppMobile -->|HTTPS| APIGateway
    AppMobile -.->|Posiciones GPS| Traccar

    APIGateway -->|REST / JSON| Backend
    Backend <-->|Mensajes JSON| Broker
    MSMarketplace <-->|Mensajes JSON| Broker
    MSInventario <-->|Mensajes JSON| Broker
    MSCompras <-->|Mensajes JSON| Broker
    Backend <-->|REST| Traccar
    Backend --> Notificaciones
    Backend --> Mapas
```

### 2.b. Integración con Sistemas Externos

| Sistema Externo | Dirección | Contenido del Intercambio |
|---|---|---|
| **Marketplace y Ventas** | Bidireccional (asíncrona) | Marketplace envía los datos de entrega del pedido de venta confirmado (destinatario, teléfono, dirección con coordenadas, referencias y preferencias), y sus cambios o cancelaciones; todos los pedidos se asumen pagados. Despachos informa el estado y resultado de cada entrega. |
| **Gestión de Inventarios y Almacén** | Bidireccional (asíncrona) | Almacén avisa cuando un pedido de venta está empacado y listo para retirar (bultos con ID, medidas y peso real), solicita recojos RMA y confirma la recepción física de recogidas y devoluciones. Despachos informa la salida física (transferencia de custodia) de los bultos. |
| **Gestión de Compras y Proveedores** | Bidireccional (asíncrona) | Compras envía solicitudes de recogida en proveedor ligadas a una orden de compra aprobada, con sus cambios o cancelación. Despachos informa la programación, el estado del traslado y sus incidencias. |
| **Traccar (Servidor GPS)** | Entrada | La app móvil envía posiciones periódicas al servidor Traccar; el componente de seguimiento de ubicación de Despachos consulta posiciones por REST para el tablero y el cálculo de ETA. |
| **Servicio de Correo / Push / SMS** | Salida | Notificaciones automáticas al cliente (correo), al repartidor (push) y envío del código OTP de entrega (SMS, mediante adaptador simulado durante el proyecto). |
| **Google Maps / Waze / OSRM** | Salida / Interno | Apertura de navegación externa hacia la dirección de entrega mediante enlaces profundos desde la app móvil y cálculo de rutas óptimas con OSRM. |

---

## 3. Modelo Estructural de Datos Estático

La entidad central es **`Despacho`**, que representa cada servicio de traslado: entrega al cliente, recogida en proveedor y recojo de devolución (logística inversa), distinguidos por su `tipo` (`delivery`, `supplier_pickup`, `warehouse_return_pickup` y `supplier_return`, catálogo `dispatch_types`). Concentra los datos de contacto, dirección, ventana horaria, prioridad, peso y marcas de tiempo del proceso. Despachos genera su propio `codigoTraslado` y conserva los identificadores originales del ERP (`referenciaPedidoOrigen`, que guarda el ID del pedido de venta o de la orden de compra según el tipo, y el ID de cada `Bulto`) para que cada sistema relacione sus registros sin confundirlos.

```mermaid
classDiagram
    class Despacho {
        +number id
        +string codigoTraslado
        +string tipo
        +string estado
        +string referenciaPedidoOrigen
        +string prioridad
        +string direccionEntrega
        +string nombreContacto
        +string telefonoContacto
        +string correoContacto
        +string etiquetaEstadoPago
        +number pesoEstimadoKg
        +string motivoDevolucion
        +Date inicioVentanaHoraria
        +Date finVentanaHoraria
        +Date poseeSlaLimite
        +number ultimaLatitud
        +number ultimaLongitud
        +Date ultimaActualizacionUbic
        +Date confirmadoAt
        +actualizarEstado(estado)
        +actualizarUbicacion(lat, lon)
        +confirmarEntrega()
        +registrarIncidencia(motivo, descripcion)
        +calificar(puntaje, comentario)
        +reprogramar(motivo, fechaNueva)
        +calcularEta()
    }

    class Bulto {
        +number id
        +string idExterno
        +number largoCm
        +number anchoCm
        +number altoCm
        +number pesoKg
        +string condicionManejo
    }

    class Usuario {
        +number id
        +string rol
        +string nombreCompleto
        +string nombreUsuario
        +string correo
        +boolean activo
    }

    class Vehiculo {
        +number id
        +string estado
        +string tipo
        +string placa
        +number capacidad
        +cambiarEstado(estado)
        +registrarMantenimiento(incidente, descripcion)
    }

    class MantenimientoVehiculo {
        +number id
        +string descripcion
        +string estado
        +Date programadoAt
        +Date completadoAt
        +completar()
    }

    class TipoIncidenteVehiculo {
        +number id
        +string nombre
    }

    class ZonaReparto {
        +number id
        +string nombre
        +number tiempoEstimadoMin
    }

    class NivelServicio {
        +number id
        +string nombre
        +number tiempoObjetivoMin
    }

    class RutaReparto {
        +number id
        +Date fecha
        +asignarRepartidor(usuario, vehiculo)
        +agregarDespacho(despacho)
    }

    class EvidenciaEntrega {
        +string id
        +string tipo
        +string urlArchivo
        +string codigoOtp
        +Date createdAt
    }

    class IncidenciaDespacho {
        +string id
        +string descripcion
        +Date createdAt
    }

    class MotivoIncidencia {
        +number id
        +string nombre
    }

    class MotivoReprogramacion {
        +number id
        +string nombre
    }

    class Reprogramacion {
        +Date fechaAnterior
        +Date fechaNueva
        +Date createdAt
    }

    class CalificacionDespacho {
        +number id
        +number puntaje
        +string comentario
        +Date createdAt
    }

    class ReclamoDespacho {
        +number id
        +string descripcion
        +string estado
        +Date createdAt
    }

    class EventoDespacho {
        +number id
        +string tipoEvento
        +string detalle
        +Date createdAt
    }

    class LogAuditoria {
        +number id
        +string accion
        +string detalle
        +Date createdAt
    }

    Usuario "1" <-- "0..*" Despacho : asignado a (repartidor)
    Vehiculo "1" <-- "0..*" Despacho : transporta
    ZonaReparto "1" <-- "0..*" Despacho : cubre
    NivelServicio "1" <-- "0..*" Despacho : define
    RutaReparto "1" *-- "0..*" Despacho : agrupa
    RutaReparto --> Usuario : conductor
    RutaReparto --> Vehiculo : asignado
    Vehiculo "1" *-- "0..*" MantenimientoVehiculo : tiene
    MantenimientoVehiculo --> TipoIncidenteVehiculo : clasificado por
    Despacho "1" *-- "0..*" Bulto : transporta
    Despacho "1" *-- "0..*" EvidenciaEntrega : genera
    Despacho "1" *-- "0..*" IncidenciaDespacho : registra
    IncidenciaDespacho --> MotivoIncidencia : clasificada por
    Despacho "1" *-- "0..*" Reprogramacion : sufre
    Reprogramacion --> MotivoReprogramacion : justificada por
    Despacho "1" *-- "0..1" CalificacionDespacho : recibe
    Despacho "1" *-- "0..*" ReclamoDespacho : genera
    Despacho "1" *-- "0..*" EventoDespacho : audita
    Usuario "1" *-- "0..*" LogAuditoria : realiza
```

---

## 4. Pila Tecnológica del Microservicio

| Capa / Aspecto | Herramienta / Tecnología | Propósito |
|---|---|---|
| **Backend** | **NestJS (Node.js + TypeScript)** | API REST, lógica de negocio modular, WebSockets, validaciones Joi/class-validator. |
| **Frontend Web** | **React 19 + Vite 8 + TypeScript** | Portal público del cliente y Panel administrativo de despacho. Tailwind v4 + shadcn/ui. |
| **Frontend Móvil** | **React Native (Expo SDK 52)** | Aplicación nativa/híbrida para repartidores con NativeWind v4 y Expo Router. |
| **Base de Datos** | **PostgreSQL (v18+)** | Base de datos relacional independiente por microservicio con esquemas por dominio. |
| **Caché** | **Redis** | Caché de lectura con TTL corto (KPIs, reportes SLA, parámetros operativos) y control de vigencia e intentos del código OTP. |
| **Mensajería ERP** | **Google Cloud Pub/Sub (emulador local)** | Intercambio asíncrono de mensajes JSON con Marketplace, Inventarios y Compras; con adaptador en memoria para pruebas. |
| **Autenticación & Autorización** | **JWT con Refresh Tokens** | Control de acceso por roles (`Coordinador`, `Supervisor`, `Repartidor`, `Cliente`). |
| **Ruteo & ETA** | **OSRM (Open Source Routing Machine)** | Motor de ruteo self-hosted con extracto de mapas de Bolivia (Geofabrik). |
| **Rastreo GPS** | **Traccar + Geolocalización Móvil** | Transmisión de coordenadas en segundo plano desde la app móvil del repartidor. |
| **Notificaciones** | **Push (Expo/Firebase) + Mailer (Resend/SMTP) + SMS (adaptador simulado)** | Avisos a repartidores, notificaciones de estado por correo a clientes y envío del OTP de entrega. |
| **Generación de Reportes** | **Puppeteer / CSV** | Exportación de logs de auditoría, reportes de flota y cumplimiento SLA. |
