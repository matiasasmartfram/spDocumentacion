---
layout: default
title: "Funcionalidad Picked Up - PedidosYa"
subtitle: "Endpoints GET/PUT para consultar y actualizar el retiro de un pedido"
---

## 1. Descripción General

Esta funcionalidad permite **saber y registrar** si un pedido de **PedidosYa** ya fue retirado por el repartidor.

Se apoya en un campo del pedido llamado **`orderPickedUp`** (booleano):

| Valor    | Significado                                              |
| -------- | -------------------------------------------------------- |
| `false`  | El pedido **todavía no** fue retirado (valor por defecto). |
| `true`   | El pedido **ya fue retirado** por el repartidor.          |

Hay dos operaciones disponibles:

1. **GET** → Consultar el estado actual de **`orderPickedUp`**.
2. **PUT** → Cambiar / actualizar el estado del pedido (marcarlo como retirado o cancelado).

<div class="note">
  <p><strong>Nota:</strong> El campo <strong>orderPickedUp</strong> vive en la raíz del documento de la colección <strong>news</strong> en MongoDB. Para el detalle del campo a nivel modelo y su comportamiento en la base de datos, ver la documentación de análisis de la variable orderPickedUp.</p>
</div>

---

## 2. Autenticación

Ambos endpoints requieren un **Bearer Token** en el header `Authorization`.

```
Authorization: Bearer <TOKEN>
```

<div class="note">
  <p><strong>Nota:</strong> Este token es el del <strong>middleware de PedidosYa</strong>. No se incluye en este documento por seguridad. Solicitarlo a quien corresponda.</p>
</div>

---

## 3. GET — Consultar si el pedido fue retirado

Devuelve el estado actual del campo **`orderPickedUp`** de un pedido.

### Endpoint

```
GET /order/remoteOrderId/{remoteOrderId}
```

- **`{remoteOrderId}`** → es el **ID interno del pedido** (`order.id`).

### Ejemplo de Request

```bash
curl --location 'https://prodpedidos.smartfran.com/order/remoteOrderId/2029405345' \
--header 'Authorization: Bearer <TOKEN>' \
--header 'Content-Type: application/json'
```

### Respuestas Posibles

**200 OK** — el pedido existe:

```json
{
  "orderId": "<originalId del pedido>",
  "orderPickedUp": true
}
```

| Campo            | Significado                                                        |
| ---------------- | ------------------------------------------------------------------ |
| `orderId`        | ID original del pedido.                                             |
| `orderPickedUp`  | `true` = ya retirado por el repartidor / `false` = aún no retirado. |

**404 Not Found** — el pedido no existe:

```json
{ "msg": "Orden no encontrada" }
```

**400 Bad Request** — falta el parámetro:

```json
{ "message": "Insuficient parameter." }
```

---

## 4. PUT — Actualizar el estado del pedido

Permite cambiar el estado de un pedido. Según el valor enviado en **`status`**, ejecuta una acción distinta.

### Endpoint

```
PUT /remoteId/{remoteId}/remoteOrder/{remoteOrderId}/posOrderStatus
```

- **`{remoteId}`** → identificador de la tienda / sucursal.
- **`{remoteOrderId}`** → ID interno del pedido (`order.id`).

### Body

```json
{
  "status": "order_picked_up"
}
```

### Ejemplo de Request (marcar como retirado)

```bash
curl --location --request PUT 'https://prodpedidos.smartfran.com/remoteId/3979/remoteOrder/1909801943/posOrderStatus' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer <TOKEN>' \
--data '{
    "status": "order_picked_up"
}'
```

### Valores de `status` que se procesan

El sistema evalúa el contenido del campo **`status`** (sin distinguir mayúsculas/minúsculas):

| Valor de `status`             | Acción que ejecuta                                                                 |
| ----------------------------- | ---------------------------------------------------------------------------------- |
| Contiene `"cancel"`           | Cancela el pedido (genera novedad de rechazo por plataforma).                      |
| Contiene `"order_picked_up"`  | Marca el pedido como retirado → setea **`orderPickedUp = true`**.                   |
| Cualquier otro valor          | No realiza ninguna acción (responde indicando que el estado no fue reconocido).    |

### Respuestas Posibles

**Pedido cancelado** (`status` contiene `cancel`) — **200 OK**:

```json
{
  "status": "ORDER_CANCELLED",
  "message": "customer cancelled order"
}
```

**Pedido retirado** (`status` contiene `order_picked_up`) — **200 OK**:

```json
{
  "status": "ORDER_DISPATCHED",
  "message": "Order dispatched"
}
```

**Estado no reconocido** — **200 OK** (no hace nada):

```json
{
  "status": "Status cancel or pick up NOT FOUND",
  "message": "Error cancel or pick up status NOT FOUND"
}
```

**Faltan parámetros** — **400 Bad Request**:

```json
{ "error": "Insuficient parameters." }
```

**Error interno** — **400 Bad Request**:

```json
{
  "reason": "Error",
  "message": "Error Update Order Status"
}
```

---

## 5. Regla de Negocio

Una vez que un pedido está marcado como **retirado** (`orderPickedUp = true`), **ya no puede ser rechazado por la sucursal**.

Cuando llega una solicitud de rechazo desde la sucursal, el sistema primero verifica el estado del pedido:

- Si **`orderPickedUp === true`** → el rechazo se **ignora** (el pedido ya está en manos del repartidor) y se registra un log informativo.
- Si **`orderPickedUp === false`** → el rechazo continúa normalmente.

Esto evita rechazar pedidos que físicamente ya salieron del local.

---

## 6. Flujo Resumido

```
1. PedidosYa notifica que el repartidor retiró el pedido
        │
        ▼
2. PUT .../posOrderStatus  con  { "status": "order_picked_up" }
        │
        ▼
3. El sistema setea  orderPickedUp = true  en el pedido
        │
        ▼
4. Se puede consultar en cualquier momento con:
   GET /order/remoteOrderId/{remoteOrderId}
        │
        ▼
5. A partir de aquí, la sucursal ya NO puede rechazar el pedido
```

---

## 7. Referencias en el Código

| Funcionalidad                         | Archivo                                                                 |
| ------------------------------------- | ----------------------------------------------------------------------- |
| Endpoint GET (`getOrder`)             | `api/src/controllers/peya.js`                                           |
| Endpoint PUT (`updateOrder`)          | `api/src/controllers/peya.js`                                           |
| Definición de rutas                   | `api/src/routes/peya.js`                                                |
| Campo `orderPickedUp` (modelo)        | `api/src/models/news.js`                                                |
| Regla de negocio (rechazo bloqueado)  | `api/src/platforms/management/strategies/orderStrategy/branchRejectStrategy.js` |
| Autenticación de las rutas            | `api/src/middlewares/route-permissions.js`                              |
