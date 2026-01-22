---
layout: default
title: Documentación del Documento "news" - SmartPedidos Platform
subtitle: Descripción General y Definición de Atributos
---

## **📋 Descripción General**

El documento **`news`** es la entidad principal del sistema SmartPedidos, utilizada para gestionar y rastrear todos los eventos relacionados con pedidos de plataformas de delivery (PedidosYa, Rappi, Glovo, UberEats, etc.) así como eventos globales del sistema (apertura/cierre de locales, advertencias).

Este documento actúa como un contenedor unificado que almacena la información completa del pedido junto con su historial de cambios de estado (trazas).

---

# **🏗️ Estructura Jerárquica del Documento**

## **📖 Definición Detallada de Atributos**

### **1. `_id` (ObjectId)**

| **Atributo** | **Tipo** | **Descripción** |
| --- | --- | --- |
| `$oid` | String | Identificador único de MongoDB para el documento news. Es el ID primario que distingue cada noticia/pedido de forma única en la base de datos. |

**Ejemplo:** `"695c154f32f48bf780078191"`

**Uso:** Utilizado para todas las operaciones de consulta, actualización y eliminación del documento. Es el campo索引 principal.

---

### **2. `order` (Objeto Principal)**

Este es el objeto más importante del documento, contiene toda la información del pedido proveniente de la plataforma externa.

#### 2.1 `order.id` (String)

| **Atributo** | **Tipo** | **Descripción** |
| --- | --- | --- |
| id | String | Identificador único del pedido asignado por el sistema SmartPedidos. Es el ID interno que referencia este pedido en nuestra plataforma. |

**Ejemplo:** `"1851195419"`

<div class="note">
  <p><strong>Notas:</strong> Este ID es generado por SmartPedidos y se mantiene consistente durante toda la vida del pedido.</p>
</div>

---

#### 2.2 `order.originalId` (String)

| **Atributo** | **Tipo** | **Descripción** |
| --- | --- | --- |
| originalId | String | Identificador original del pedido en la plataforma externa (PedidosYa, Rappi, etc.). |

**Ejemplo:** `"1851195419"`

<div class="note">
  <p><strong>Notas:</strong> Este es el ID que la plataforma de delivery asignó originalmente al pedido. En muchos casos puede ser igual a <strong>id</strong>, pero representa el origen del pedido.</p>
</div>

---

#### 2.3 `order.displayId` (String)

| **Atributo** | **Tipo** | **Descripción** |
| --- | --- | --- |
| displayId | String | Identificador público del pedido, mostrado a clientes y staff. |

**Ejemplo:** `"1851195419"`

<div class="note">
  <p><strong>Notas:</strong> Es el ID visible que se muestra en las interfaces de usuario, recibos y comunicaciones con el cliente.</p>
</div>

---

#### 2.4 `order.platformId` (Number)

| **Atributo** | **Tipo** | **Descripción** |
| --- | --- | --- |
| platformId | Number | Identificador de la plataforma de delivery de donde proviene el pedido. |

**Ejemplo:** `1`

**Valores Comunes:**

*   `1` = PedidosYa
*   `2` = Rappi
*   `3` = Glovo
*   `4` = UberEats
*   `5` = MercadoPago

<div class="note">
  <p><strong>Notas:</strong> Este campo permite identificar rápidamente qué plataforma externa generó el pedido, útil para aplicar lógicas específicas por plataforma.</p>
</div>

---

#### 2.5 `order.statusId` (Number)

| **Atributo** | **Tipo** | **Descripción** |
| --- | --- | --- |
| statusId | Number | Estado actual del pedido dentro del flujo de procesamiento. |

**Ejemplo:** `12`

**Mapeo de Estados:**

| **statusId** | **Código** | **Nombre Español** | **Descripción** |
| --- | --- | --- | --- |
| 1 | `pend` | Pendiente | Pedido nuevo, esperando acción |
| 2 | `rej` | Rechazada | Pedido rechazado por el local |
| 3 | `confirm` | Confirmada | Pedido confirmado por el local |
| 4 | `dispatch` | Despachada | Pedido entregado al repartidor |
| 5 | `delivery` | Entregada | Pedido entregado al cliente |
| 8 | `preparing` | Preparando | Pedido en preparación |
| 9 | `view` | Vista | Pedido fue visto por el local |
| 10 | `receive` | Recibida | Pedido recibido por el local |
| 11 | `rej_closed` | Restaurante Cerrado | Rechazado por restaurante cerrado |
| 12 | `ready` | Lista | Pedido listo para recoger |

<div class="note">
  <p><strong>Notas:</strong> El <strong>statusId</strong> determina qué acciones están disponibles y cómo se muestra el pedido en las interfaces.</p>
</div>

---

#### 2.6 `order.orderTime` (Date)

| **Atributo** | **Tipo** | **Descripción** |
| --- | --- | --- |
| orderTime | Date | Fecha y hora exacta en que el cliente realizó el pedido en la plataforma externa. |

**Ejemplo:** `{$date: "2026-01-05T16:47:14.617Z"}`

**Formato:** ISO 8601 (UTC)

<div class="note">
  <p><strong>Notas:</strong> Esta marca de tiempo es crítica para calcular tiempos de preparación y cumplir con los SLA de las plataformas de delivery.</p>
</div>

---

#### 2.7 `order.deliveryTime` (Date)

| **Atributo** | **Tipo** | **Descripción** |
| --- | --- | --- |
| deliveryTime | Date | Fecha y hora estimada o承诺 para la entrega del pedido al cliente. |

**Ejemplo:** `{$date: "2026-01-05T16:57:14.000Z"}`

**Formato:** ISO 8601 (UTC)

<div class="note">
  <p><strong>Notas:</strong></p>
  <ul>
    <li>Es establecido por la plataforma de delivery</li>
    <li>Puede ser modificado por el local si hay retrasos</li>
    <li>Es crucial para la satisfacción del cliente y métricas de rendimiento</li>
  </ul>
</div>

---

#### 2.8 `order.pickupOnShop` (Boolean)

| **Atributo** | **Tipo** | **Descripción** |
| --- | --- | --- |
| pickupOnShop | Boolean | Indica si el cliente retirará el pedido físicamente en el local en lugar de esperar delivery. |

**Ejemplo:** `false`

**Valores:**

*   `true` = Cliente retira en local (pickup)
*   `false` = Delivery a domicilio

<div class="note">
  <p><strong>Notas:</strong> Cuando es <strong>true</strong>, el flujo de estado cambia y no se espera un repartidor.</p>
</div>

---

#### 2.9 `order.pickupDateOnShop` (Date | null)

| **Atributo** | **Tipo** | **Descripción** |
| --- | --- | --- |
| pickupDateOnShop | Date \| null | Fecha programada para el retiro en local si es un pre-pedido. |

**Ejemplo:** `null`

<div class="note">
  <p><strong>Notas:</strong> Se utiliza para pedidos anticipados (pre-orders) donde el cliente programa cuándo retirar.</p>
</div>

---

#### 2.10 `order.preOrder` (Boolean)

| **Atributo** | **Tipo** | **Descripción** |
| --- | --- | --- |
| preOrder | Boolean | Indica si el pedido es un pre-pedido (orden anticipada). |

**Ejemplo:** `false`

**Valores:**

*   `true` = Pedido programado para una fecha/hora futura
*   `false` = Pedido inmediato

<div class="note">
  <p><strong>Notas:</strong> Los pre-pedidos requieren atención especial en la planificación de inventario y personal.</p>
</div>

---

#### 2.11 `order.observations` (String)

| **Atributo** | **Tipo** | **Descripción** |
| --- | --- | --- |
| observations | String | Observaciones y notas adicionales del pedido. |

**Ejemplo:** `"#1222 | Medio de pago: VISA|"`

**Contenido Típico:**

*   Número de mesa (#1222)
*   Medio de pago utilizado
*   Instrucciones especiales del cliente
*   Notas del repartidor

<div class="note">
  <p><strong>Notas:</strong> Este campo concatena múltiples tipos de información separada por pipes <strong>|</strong>. Es importante parsear correctamente.</p>
</div>

---

#### 2.12 `order.ownDelivery` (Boolean)

| **Atributo** | **Tipo** | **Descripción** |
| --- | --- | --- |
| ownDelivery | Boolean | Indica si el pedido será entregado con flota propia del local en lugar de la plataforma. |

**Ejemplo:** `false`

**Valores:**

*   `true` = Delivery con personal propio del local
*   `false` = Delivery gestionado por la plataforma

<div class="note">
  <p><strong>Notas:</strong> Afecta el flujo de asignación de repartidores y cálculo de comisiones.</p>
</div>

---

#### 2.13 `order.tenant` (Objeto)

Contiene información del cliente/organización que utiliza el sistema SmartPedidos.

```json
tenant: {
  "name": "grido",
  "orgId": "org_g4qPlLbxcJZx5e7U",
  "tenantId": "d3186bc6d7b2"
}

| **Atributo** | **Tipo** | **Descripción** |
| --- | --- | --- |
| `name` | String | Nombre comercial del tenant (cadena/restaurante). |
| `orgId` | String | Identificador de organización en sistemas externos. |
| `tenantId` | String | Identificador único del tenant en SmartPedidos. |

<div class="note">
  <p><strong>Notas:</strong> Permite la multi-tenancy del sistema, separando datos de diferentes clientes.</p>
</div>

#### 2.14 order.customer (Objeto)

Información del cliente que realizó el pedido.

```json
customer: {
  "id": "41965339",
  "name": "Mica Cattarozzi",
  "address": "Sin especificar",
  "phone": "+543514597169",
  "email": "",
  "dni": null
}
```

| **Atributo** | **Tipo** | **Descripción** |
| --- | --- | --- |
| `id` | String | Identificador del cliente en la plataforma de delivery. |
| `name` | String | Nombre completo del cliente. |
| `address` | String | Dirección de entrega del pedido. |
| `phone` | String | Número de teléfono del cliente. |
| `email` | String | Correo electrónico del cliente. |
| `dni` | String \| null | Documento Nacional de Identidad (para Argentina). |

<div class="note">
  <p><strong>Notas:</strong></p>
  <ul>
    <li>Algunos campos pueden estar vacíos según la plataforma</li>
    <li>El <strong>address</strong> puede ser "Sin especificar" si el cliente tiene direcciones guardadas</li>
    <li>Es importante para comunicación y seguimiento de entrega</li>
  </ul>
</div>

#### 2.15 order.details (Array)

Lista de productos/items que conforman el pedido.

```json
details: [
  {
    "productId": "191259688",
    "count": "1",
    "price": "10200",
    "promo": 2,
    "groupId": 1,
    "discount": "",
    "description": "PROMO 2 Potes de Helado 1/4 Kg al 20% OFF",
    "sku": "6844",
    "note": "",
    "canje": 0
  }
  // ... más productos
]
```

| **Atributo** | **Tipo** | **Descripción** |
| --- | --- | --- |
| `productId` | String | Identificador único del producto en el sistema. |
| `count` | String | Cantidad solicitada del producto. |
| `price` | String | Precio unitario del producto. |
| `promo` | Number | Tipo de promoción aplicada (0=sin promo, 1=incluido en promo, 2=promo principal). |
| `groupId` | Number | Grupo de productos (útil para bundles/promociones). |
| `discount` | String | Descuento aplicado al producto. |
| `description` | String | Descripción/nombre del producto. |
| `sku` | String | Código SKU del producto. |
| `note` | String | Notas especiales para el producto. |
| `optionalText` | String | Detalles de personalización (sabores, ingredientes, etc.). |
| `canje` | Number | Indicador de canje (0=no es canje, 1=es canje). |

**Ejemplo de Producto con Sabores:**

```json
{
  "productId": "41361630",
  "promo": 1,
  "groupId": 1,
  "description": "Sabores Helado 1/4 KG",
  "sku": "66",
  "optionalText": " Mascarpone con Frutos del Bosque X 1 Super Dulce de Leche X 1"
}
```

<div class="note">
  <p><strong>Notas:</strong></p>
  <ul>
    <li>Los productos con <strong>promo: 1</strong> son componentes de una promoción</li>
    <li><strong>optionalText</strong> contiene las personalizaciones (sabores, cantidades por sabor)</li>
    <li>Los grupos (<strong>groupId</strong>) conectan productos que pertenecen a la misma promoción</li>
  </ul>
</div>

#### 2.16 order.payment (Objeto)

Información de pago del pedido.

```json
payment: {
  "discount": 2040,
  "voucher": "",
  "remaining": 0,
  "partial": 0,
  "typeId": 2,
  "online": true,
  "shipping": 0,
  "subtotal": 19000,
  "currency": "$",
  "note": ""
}
```

| **Atributo** | **Tipo** | **Descripción** |
| --- | --- | --- |
| `discount` | Number | Monto total de descuento aplicado. |
| `voucher` | String | Código de voucher utilizado. |
| `remaining` | Number | Saldo pendiente de pago. |
| `partial` | Number | Indica si es pago parcial. |
| `typeId` | Number | Tipo de método de pago. |
| `online` | Boolean | Indica si el pago fue realizado online. |
| `shipping` | Number | Costo de envío. |
| `subtotal` | Number | Subtotal del pedido (sin descuentos). |
| `currency` | String | Símbolo de la moneda. |
| `note` | String | Notas sobre el pago. |

**Tipos de Pago (`typeId`):**

*   `1` = Efectivo
*   `2` = Tarjeta de Crédito/Débito
*   `3` = MercadoPago
*   `4` = Otro

**Cálculo del Total:**

`totalAmount = subtotal - discount + shipping`

<div class="note">
  <p><strong>Notas:</strong></p>
  <ul>
    <li><strong>online: true</strong> indica que el pago ya fue procesado por la plataforma</li>
    <li>El <strong>discount</strong> ya está incluido en el <strong>totalAmount</strong></li>
  </ul>
</div>

#### 2.17 order.driver (Objeto)

Información del repartidor asignado (cuando está disponible).

```json
driver: {
  "pickUpDatetime": "2026-01-05T19:52:27.253Z",
  "estimatedDeliveryDate": "2026-01-05T19:57:14.000Z"
}
```

| **Atributo** | **Tipo** | **Descripción** |
| --- | --- | --- |
| `pickUpDatetime` | Date | Fecha y hora en que el repartidor recogió el pedido. |
| `estimatedDeliveryDate` | Date | Fecha estimada de entrega al cliente. |

<div class="note">
  <p><strong>Notas:</strong></p>
  <ul>
    <li>Este objeto puede estar vacío si no hay repartidor asignado aún</li>
    <li><strong>pickUpDatetime</strong> solo se completa cuando el repartidor confirma el retiro</li>
  </ul>
</div>

#### 2.18 order.totalAmount (Number)

| **Atributo** | **Tipo** | **Descripción** |
| --- | --- | --- |
| `totalAmount` | Number | Monto total final del pedido (con descuentos incluidos). |

**Ejemplo:** `16960`

**Unidad:** Moneda local (indicada en `payment.currency`)

<div class="note">
  <p><strong>Notas:</strong></p>
  <ul>
    <li>Es el monto que el cliente paga finalmente</li>
    <li>Ya incluye el descuento de promociones</li>
    <li>No incluye shipping si es adicional (ver <strong>payment.shipping</strong>)</li>
  </ul>
</div>

### 3. __v (Number)

| **Atributo** | **Tipo** | **Descripción** |
| --- | --- | --- |
| `__v` | Number | Versión del documento MongoDB (utilizada por Mongoose para control de concurrencia). |

**Ejemplo:** `0`

<div class="note">
  <p><strong>Notas:</strong> Campo interno de Mongoose, generalmente no se utiliza directamente en la lógica de negocio.</p>
</div>

### 4. branchId (Number)

| **Atributo** | **Tipo** | **Descripción** |
| --- | --- | --- |
| `branchId` | Number | Identificador de la sucursal/local que debe procesar el pedido. |

**Ejemplo:** `3002`

<div class="note">
  <p><strong>Notas:</strong></p>
  <ul>
    <li>Es el campo clave para direccionar el pedido al local correcto</li>
    <li>Se utiliza para filtrar pedidos en las vistas por local</li>
    <li>Generalmente coincide con el código de punto de venta</li>
  </ul>
</div>

### 5. createdAt (Date)

| **Atributo** | **Tipo** | **Descripción** |
| --- | --- | --- |
| `createdAt` | Date | Fecha y hora de creación del documento en MongoDB. |

**Ejemplo:** `{$date: "2026-01-05T19:47:27.515Z"}`

**Formato:** ISO 8601 (UTC)

<div class="note">
  <p><strong>Notas:</strong></p>
  <ul>
    <li>Se genera automáticamente al insertar el documento</li>
    <li>Útil para auditoria y métricas de rendimiento</li>
    <li>Puede diferir de <strong>order.orderTime</strong> si hay delay en el procesamiento</li>
  </ul>
</div>

### 6. extraData (Objeto)

Metadatos adicionales sobre el contexto del pedido.

```json
extraData: {
  "branch": "COLÓN",
  "chain": "Grido",
  "platform": "PedidosYa",
  "client": "Progreso J y T SA",
  "country": "Argentina"
}
```

| **Atributo** | **Tipo** | **Descripción** |
| --- | --- | --- |
| `branch` | String | Nombre de la sucursal. |
| `chain` | String | Nombre de la cadena/franquicia. |
| `platform` | String | Nombre de la plataforma de delivery. |
| `client` | String | Nombre del cliente tenant. |
| `country` | String | País donde se procesa el pedido. |

<div class="note">
  <p><strong>Notas:</strong></p>
  <ul>
    <li>Datos desnormalizados para facilitar consultas y reportes</li>
    <li>Evita necesidad de joins con otras colecciones</li>
    <li>Útil para filtrar y categorizar en dashboards</li>
  </ul>
</div>

### 7. traces (Array)

Historial de cambios de estado del pedido. Es un array de objetos que registran cada actualización.

```json
traces: [
  {
    "update": {
      "typeId": 1,
      "orderStatusId": 1
    },
    "_id": {"$oid": "695c154f4b9d26ad640a7a5f"},
    "entity": "PLATFORM",
    "createdAt": {"$date": "2026-01-05T19:47:27.518Z"}
  }
  // ... más trazas
]
```

#### Estructura de cada traza:

| **Atributo** | **Tipo** | **Descripción** |
| --- | --- | --- |
| `update` | Object | Contiene los datos de la actualización. |
| `update.typeId` | Number | Tipo de actualización realizada. |
| `update.orderStatusId` | Number | Estado del pedido después de la actualización. |
| `update.isValid` | Boolean | Si la actualización fue válida. |
| `update.platformResult` | Boolean | Resultado de la operación en la plataforma. |
| `update.updatedAt` | Date | Fecha de la actualización. |
| `update.typeIdPrev` | Number | Tipo de actualización anterior. |
| `update.deliveryTimeId` | Number | ID del tiempo de entrega. |
| `_id` | ObjectId | ID de la traza. |
| `entity` | String | Quién realizó la actualización (PLATFORM, BRANCH). |
| `createdAt` | Date | Fecha de creación de la traza. |

#### Tipos de Entidades (`entity`):

| **Entity** | **Descripción** |
| --- | --- |
| `PLATFORM` | Actualización recibida de la plataforma de delivery. |
| `BRANCH` | Actualización realizada por el local/restaurante. |

#### Flujo Típico de Trazas:

<div class="note">
  <p><strong>Notas:</strong></p>
  <ul>
    <li>Las trazas son inmutables, solo se agregan nuevas</li>
    <li>Permiten reconstruir el historial completo del pedido</li>
    <li>Útiles para debugging y resolución de conflictos</li>
  </ul>
</div>

### 8. typeId (Number)

| **Atributo** | **Tipo** | **Descripción** |
| --- | --- | --- |
| `typeId` | Number | Tipo de evento/news que representa este documento. |

**Ejemplo:** `15`

**Mapeo de Tipos:**

| **typeId** | **Código** | **Nombre** | **Descripción** |
| --- | --- | --- | --- |
| 1 | `new_ord` | newOrder | Nuevo pedido recibido |
| 2 | `rej_ord` | rejected | Pedido rechazado |
| 3 | `disp_ord` | dispatchedOrder | Pedido despachado |
| 4 | `platform_rej_ord` | platformRejected | Rechazado por plataforma |
| 5 | `confirm_ord` | acceptedOrder | Pedido confirmado |
| 6 | `open_branch` | branchOpened | Local abierto |
| 7 | `close_branch` | branchClosed | Local cerrado |
| 8 | `warning` | branchWarning | Advertencia del local |
| 9 | `upd_ord` | updatedOrder | Pedido actualizado |
| 10 | `receive_ord` | receivedOrder | Pedido recibido |
| 11 | `view_ord` | viewedOrder | Pedido visto |
| 12 | `driver_ord` | retriveDriver | Repartidor asignado |
| 13 | `rej_closed_ord` | rejectedClosedRestaurant | Rechazado por local cerrado |
| 14 | `deliv_ord` | deliveredOrder | Pedido entregado |
| 15 | `ready_ord` | readyOrder | Pedido listo |

<div class="note">
  <p><strong>Notas:</strong></p>
  <ul>
    <li>Determina el flujo de procesamiento del documento</li>
    <li>Se combina con <strong>order.statusId</strong> para determinar estado completo</li>
    <li>Tipos con prefijo <strong>order_</strong> son eventos de pedidos</li>
    <li>Tipos con prefijo <strong>branch_</strong> son eventos globales</li>
  </ul>
</div>

### 9. updatedAt (Date)

| **Atributo** | **Tipo** | **Descripción** |
| --- | --- | --- |
| `updatedAt` | Date | Fecha y hora de la última actualización del documento. |

**Ejemplo:** `{$date: "2026-01-05T19:48:12.098Z"}`

**Formato:** ISO 8601 (UTC)

<div class="note">
  <p><strong>Notas:</strong></p>
  <ul>
    <li>Se actualiza automáticamente en cada save()</li>
    <li>Útil para identificar documentos que necesitan sincronización</li>
    <li>Base para métricas de tiempo de respuesta</li>
  </ul>
</div>

### 10. viewed (Date)

| **Atributo** | **Tipo** | **Descripción** |
| --- | --- | --- |
| `viewed` | Date | Fecha y hora en que el pedido fue marcado como visto por el local. |

**Ejemplo:** `{$date: "2026-01-05T19:47:28.367Z"}`

**Formato:** ISO 8601 (UTC)

<div class="note">
  <p><strong>Notas:</strong></p>
  <ul>
    <li>Puede ser <strong>null</strong> si el pedido no ha sido visto</li>
    <li>Se setea cuando el local abre el detalle del pedido</li>
    <li>Importante para métricas de tiempo de respuesta</li>
  </ul>
</div>

## 🔄 Flujo de Vida del Pedido

Unable to Render Diagram

## 📊 Ejemplo Completo de Documento

```json
{
  "_id": {
    "$oid": "695c154f32f48bf780078191"
  },
  "order": {
    "id": "1851195419",
    "originalId": "1851195419",
    "displayId": "1851195419",
    "platformId": 1,
    "statusId": 12,
    "orderTime": { "$date": "2026-01-05T16:47:14.617Z" },
    "deliveryTime": { "$date": "2026-01-05T16:57:14.000Z" },
    "pickupOnShop": false,
    "pickupDateOnShop": null,
    "preOrder": false,
    "observations": "#1222 | Medio de pago: VISA|",
    "ownDelivery": false,
    "tenant": {
      "name": "grido",
      "orgId": "org_g4qPlLbxcJZx5e7U",
      "tenantId": "d3186bc6d7b2"
    },
    "customer": {
      "id": "41965339",
      "name": "Mica Cattarozzi",
      "address": "Sin especificar",
      "phone": "+543514597169",
      "email": "",
      "dni": null
    },
    "details": [
      {
        "productId": "191259688",
        "count": "1",
        "price": "10200",
        "promo": 2,
        "groupId": 1,
        "discount": "",
        "description": "PROMO 2 Potes de Helado 1/4 Kg al 20% OFF",
        "sku": "6844",
        "note": "",
        "canje": 0
      }
    ],
    "payment": {
      "discount": 2040,
      "voucher": "",
      "remaining": 0,
      "partial": 0,
      "typeId": 2,
      "online": true,
      "shipping": 0,
      "subtotal": 19000,
      "currency": "$",
      "note": ""
    },
    "driver": {
      "pickUpDatetime": "2026-01-05T19:52:27.253Z",
      "estimatedDeliveryDate": "2026-01-05T19:57:14.000Z"
    },
    "totalAmount": 16960
  },
  "__v": 0,
  "branchId": 3002,
  "createdAt": { "$date": "2026-01-05T19:47:27.515Z" },
  "extraData": {
    "branch": "COLÓN",
    "chain": "Grido",
    "platform": "PedidosYa",
    "client": "Progreso J y T SA",
    "country": "Argentina"
  },
  "traces": [
    {
      "update": { "typeId": 1, "orderStatusId": 1 },
      "_id": { "$oid": "695c154f4b9d26ad640a7a5f" },
      "entity": "PLATFORM",
      "createdAt": { "$date": "2026-01-05T19:47:27.518Z" }
    }
  ],
  "typeId": 15,
  "updatedAt": { "$date": "2026-01-05T19:48:12.098Z" },
  "viewed": { "$date": "2026-01-05T19:47:28.367Z" }
}
```

