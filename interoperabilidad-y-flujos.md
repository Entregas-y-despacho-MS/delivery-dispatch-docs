# Interoperabilidad y Flujos de Información (Escenarios de Negocio)
**Microservicio de Gestión de Entregas y Despachos (Grupo H)**

Este documento detalla los contratos de comunicación y los diagramas de secuencia de los cuatro procesos clave del microservicio.

---

## 1. Matriz de Integración con el ERP Corporativo

El microservicio de Despachos se comunica de forma **síncrona (REST/JSON)** a través del API Gateway común con únicamente dos microservicios externos:

```mermaid
flowchart LR
    Inventario["Gestión de Inventarios y Almacén"] <-->|REST / JSON| Despachos["Microservicio de Despachos\n(NestJS)"]
    Compras["Gestión de Compras y Proveedores"] <-->|REST / JSON| Despachos
    Despachos <-->|REST / GPS| Traccar["Servidor Traccar (GPS)"]
```

| Microservicio | Dirección | Eventos / Flujo de Información |
|---|---|---|
| **Gestión de Inventarios y Almacén** | Bidireccional | • Almacén notifica pedido preparado (`picking`/`packing`) o solicitud de devolución aprobada (RMA) con datos del cliente.<br>• Despachos confirma salida física de almacén o recolección en domicilio.<br>• Despachos reporta reingreso de mercadería por devoluciones/cancelaciones. |
| **Gestión de Compras y Proveedores** | Bidireccional | • Compras notifica órdenes de despacho y recojo de mercadería desde proveedores hacia tiendas/almacén central.<br>• Despachos reporta logística inversa (productos dañados devueltos al proveedor) y confirmación de recepción en destino. |

> [!IMPORTANT]
> **Despachos no procesa ni registra cobros.** La gestión financiera y cobros es responsabilidad exclusiva del microservicio de *Gestión de Pagos y Facturación*, con el cual Despachos no tiene interfaz directa.

---

## 2. Escenarios de Negocio y Diagramas de Secuencia

---

### 📦 Escenario 1: Entrega de Pedido a Cliente (Logística Directa)

```mermaid
sequenceDiagram
    autonumber
    actor Almacen as Gestión Inventarios y Almacén
    participant Backend as MS Despachos (Backend)
    actor Coordinador as Coordinador de Logística (Web)
    actor Repartidor as App Móvil Repartidor
    participant Traccar as Traccar (Servidor GPS)
    actor Cliente as Cliente Final (Web / Tracking)

    Almacen->>Backend: notificarPreparacion(pedido, pesoEstimado, ventanaHoraria)
    Backend->>Backend: registrarOrdenDespacho(estado = PRE_AVISO)
    Backend-->>Coordinador: mostrarOrdenEnTablero()
    
    Coordinador->>Backend: incluirEnLote(zona, turnoTarde)
    Almacen->>Backend: notificarPaqueteListo(pedido)
    Backend->>Backend: actualizarEstado(LISTO_PARA_RECOJO)
    
    Coordinador->>Backend: consolidarRuta(turno)
    Coordinador->>Backend: asignarRepartidor(ruta, repartidor, vehiculo)
    Backend->>Repartidor: enviarListaEntregas(ruta)
    
    loop Mientras haya despacho activo
        Repartidor->>Traccar: enviarPosicion(lat, lng)
        Backend->>Traccar: consultarPosiciones(dispositivo)
        Traccar-->>Backend: ultimaUbicacion
    end

    Cliente->>Backend: abrirSeguimiento(tokenSeguro)
    Backend-->>Cliente: estado + ultimaUbicacion + ETA
    
    Repartidor->>Backend: registrarEvidencia(foto, firmaDigital, OTP)
    Repartidor->>Backend: actualizarEstado(ENTREGADO)
    Backend->>Cliente: notificar(ENTREGADO por Correo)
    Backend->>Almacen: confirmarEntrega(pedido)
    Almacen->>Almacen: cerrarOrden(pedido)
```

---

### 🔄 Escenario 2: Devolución de Cliente en Domicilio (Logística Inversa)

```mermaid
sequenceDiagram
    autonumber
    actor Almacen as Gestión Inventarios y Almacén
    participant Backend as MS Despachos (Backend)
    actor Coordinador as Coordinador de Logística (Web)
    actor Repartidor as App Móvil Repartidor

    Almacen->>Almacen: aprobarSolicitudGarantia(RMA)
    Almacen->>Backend: solicitarRecojo(RMA, domicilioCliente, contacto)
    Backend->>Backend: crearOrdenRecojo(estado = PENDIENTE)
    
    Coordinador->>Backend: asignarRecojoEnLote(repartidor, barrio, turnoTarde)
    Backend->>Repartidor: enviarOrdenRecojo(domicilio)
    
    Repartidor->>Repartidor: inspeccionarProducto(piezas completas, estado físico)
    Repartidor->>Backend: registrarEvidencia(fotos, firmaComprobanteRetiro)
    Repartidor->>Backend: actualizarEstado(PRODUCTO_RETIRADO)
    
    Repartidor->>Backend: registrarDescargaEnDeposito()
    Backend->>Almacen: notificarReingresoMercaderia(RMA)
    Almacen->>Almacen: derivar a revision tecnica de inventario
```

---

### 🚚 Escenario 3: Traslado de Mercadería desde Proveedor (Abastecimiento)

```mermaid
sequenceDiagram
    autonumber
    actor Compras as Gestión Compras y Proveedores
    participant Backend as MS Despachos (Backend)
    actor Coordinador as Coordinador de Logística (Web)
    actor Repartidor as App Móvil Transportista
    participant Traccar as Traccar (GPS)
    actor Proveedor as Proveedor
    actor Cedis as Centro de Distribución (Tienda)

    Compras->>Backend: solicitarTransporte(carga, origen = Proveedor, tipo = PESADO)
    Backend->>Backend: crearOrdenTransporte(estado = PENDIENTE)
    
    Coordinador->>Backend: asignarVehiculoGranVolumen(transportista)
    Backend->>Repartidor: enviarOrdenTransporte(origen, destino)
    
    Repartidor->>Proveedor: presentarse y cargar productos
    Repartidor->>Backend: registrarSalidaOrigen()
    
    loop En viaje
        Repartidor->>Traccar: enviarPosicion(lat, lng)
        Backend->>Traccar: consultarPosiciones(dispositivo)
    end
    
    Repartidor->>Cedis: entregar carga
    Cedis->>Repartidor: firma digital de recepcion
    Repartidor->>Backend: actualizarEstado(ENTREGADO_COMPLETO)
    Backend->>Compras: confirmarRecepcion(carga, completa)
    Compras->>Compras: cerrarOrdenDeCompra()
```

---

### ⚠️ Escenario 4: Devolución de Mercadería Defectuosa a Proveedor

```mermaid
sequenceDiagram
    autonumber
    actor Compras as Gestión Compras y Proveedores
    participant Backend as MS Despachos (Backend)
    actor Coordinador as Coordinador de Logística (Web)
    actor Repartidor as App Móvil Transportista
    actor Almacen as Almacén Central
    actor Proveedor as Proveedor

    Compras->>Backend: solicitarTrasladoSalida(loteDefectuoso, destino = Proveedor)
    Backend->>Backend: crearOrdenLogisticaInversa(estado = PENDIENTE)
    
    Coordinador->>Backend: asignarRutaRetorno(camionFlota, transportista)
    Backend->>Repartidor: enviarOrdenRecojo(en Almacen, entrega a Proveedor)
    
    Repartidor->>Almacen: cargar paletas defectuosas
    Repartidor->>Backend: registrarSalidaAlmacen()
    
    Repartidor->>Proveedor: entregar carga mermada
    Proveedor->>Repartidor: firma del encargado de recepcion
    Repartidor->>Backend: actualizarEstado(ENTREGADO_A_PROVEEDOR)
    
    Backend->>Compras: confirmarDevolucion(lote)
    Compras->>Compras: procesar nota de credito / reembolso
```
