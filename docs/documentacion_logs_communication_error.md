---
layout: default
title: Análisis de Logs de Envío a Plataformas
subtitle: Relevamiento de logs de error en integraciones con APIs de plataformas (SmartPedidos)
---

Este documento contiene el relevamiento de los logs de error generados durante los envíos de información a las APIs de las diferentes plataformas (habitualmente vía Axios). Estos registros son fundamentales para la nueva determinación de estados y monitoreo de fallos en el servicio de **SmartPedidos**.

## 1. Rapiboy / PediGrido (`rapiboy.js`)

*   **Descripcion:** Error al informar a **SmartLoyalty** la recepción de un pedido con canje de puntos.
    *   **Log:** `Sml Communication Error: receiveOrder [ORDER_ID]`
    *   **customDataAfter:** action: `receiveOrder`, platformId: `PediGrido`, order: (8 campos), request: (url/body), response: .

*   **Descripcion:** Error al confirmar la recepción del pedido a la plataforma Rapiboy/PediGrido.
    *   **Log:** `Platform Communication Error: receiveOrder PediGrido [ORDER_ID]`
    *   **customDataAfter:** action: `receiveOrder`, platformId: `PediGrido`, order: (8 campos), request: (url/body), response: .

*   **Descripcion:** Error al informar a **SmartLoyalty** la confirmación de un pedido.
    *   **Log:** `Sml Communication Error: confirmOrder [ORDER_ID]`
    *   **customDataAfter:** action: `confirmOrder`, platformId: `PediGrido`, order: (8 campos), request: (url/body), response: .

*   **Descripcion:** Error al enviar la confirmación explícita del pedido a la plataforma Rapiboy.
    *   **Log:** `Platform Communication Error: confirmOrder PediGrido [ORDER_ID]`
    *   **customDataAfter:** action: `confirmOrder`, platformId: `PediGrido`, order: (8 campos), request: (url/body), response: .

*   **Descripcion:** Error al informar que el pedido está preparado y listo para su retiro.
    *   **Log:** `Platform Communication Error: readyOrder PediGrido [ORDER_ID]`
    *   **customDataAfter:** action: `readyOrder`, platformId: `PediGrido`, order: (8 campos), request: (url/body), response: .

*   **Descripcion:** Error al informar que el pedido ha sido despachado del local.
    *   **Log:** `Platform Communication Error: dispatchOrder PediGrido [ORDER_ID]`
    *   **customDataAfter:** action: `dispatchOrder`, platformId: `PediGrido`, order: (8 campos), request: (url/body), response: .

*   **Descripcion:** Error al informar que el pedido ha sido entregado exitosamente.
    *   **Log:** `Platform Communication Error: deliveryOrder PediGrido [ORDER_ID]`
    *   **customDataAfter:** action: `deliveryOrder`, platformId: `PediGrido`, order: (8 campos), request: (url/body), response: .

*   **Descripcion:** Error al informar a **SmartLoyalty** el rechazo de un pedido para la devolución de puntos.
    *   **Log:** `Sml Communication Error: branchRejectOrder [ORDER_ID]`
    *   **customDataAfter:** action: `branchRejectOrder`, platformId: `PediGrido`, order: (8 campos), request: (url/body), response: .

*   **Descripcion:** Error al informar el rechazo de la orden por parte del local a la plataforma.
    *   **Log:** `Platform Communication Error: branchRejectOrder PediGrido [ORDER_ID]`
    *   **customDataAfter:** action: `branchRejectOrder`, platformId: `PediGrido`, order: (8 campos), request: (url/body), response: .

<div class="note">
  <p><strong>Nota:</strong> Actualmente falta el log específico de la petición Axios en el código para el rechazo en PediGrido. Se debe implementar bajo la estructura definida.</p>
</div>

## 2. UberEats (`uberEats.js`)

*   **Descripcion:** Error al obtener el token de acceso de la API de UberEats a través de OAuth2.
    *   **Log:** `Falló uberLogin`
    *   **customDataAfter:** platformId: `UberEats`.

*   **Descripcion:** Error al notificar la aceptación del pedido a UberEats durante el flujo de recepción automática.
    *   **Log:** `Platform Communication Error: receiveOrder UberEats [ORDER_ID]`
    *   **customDataAfter:** action: `receiveOrder`, platformId: `4`, order: (5 campos), request: (url/body/header), response: .

*   **Descripcion:** Error al enviar la confirmación manual del pedido a la plataforma UberEats.
    *   **Log:** `Platform Communication Error: confirmOrder UberEats [ORDER_ID]`
    *   **customDataAfter:** action: `confirmOrder`, platformId: `4`, order: (5 campos), request: (url/body/header), response: .

*   **Descripcion:** Error al solicitar la cancelación del pedido en los servidores de UberEats por rechazo del local.
    *   **Log:** `Platform Communication Error: branchRejectOrder UberEats [ORDER_ID]`
    *   **customDataAfter:** action: `branchRejectOrder`, platformId: `4`, order: (4 campos), request: (url/body/header), response: .

*   **Descripcion:** Error al informar a UberEats que el pedido se encuentra listo para ser retirado por el repartidor.
    *   **Log:** `Platform Communication Error: readyOrder UberEats [ORDER_ID]`
    *   **customDataAfter:** action: `readyOrder`, platformId: `4`, order: (4 campos), request: (url/body/header), response: .


## 3. Rappi (`rappi.js`)

*   **Descripcion:** Error al obtener el token de acceso de Rappi a través de Auth0 (OAuth2).
    *   **Log:** `Falló loginToAuth0 rappi`
    *   **customDataAfter:** platformId: `Rappi`, country: (país).

*   **Descripcion:** Error durante el proceso periódico (polling) para obtener nuevos pedidos desde la API de Rappi.
    *   **Log:** `Platform Communication Error: Rappi getOrders`
    *   **customDataAfter:** action: `getOrders`, platformId: `2`, order: `null`, request: (url/headers), response: .

*   **Descripcion:** Error al notificar la aceptación automática del pedido a la plataforma Rappi.
    *   **Log:** `Platform Communication Error: Rappi receiveOrder [ORDER_ID]`
    *   **customDataAfter:** action: `receiveOrder`, platformId: `2`, order: (5 campos), request: (url/body/header), response: .

*   **Descripcion:** Error al enviar la confirmación manual de recepción del pedido a la API de Rappi.
    *   **Log:** `Platform Communication Error: Rappi confirmOrder [ORDER_ID]`
    *   **customDataAfter:** action: `confirmOrder`, platformId: `2`, order: (5 campos), request: (url/body/header), response: .

*   **Descripcion:** Error al solicitar la cancelación (rechazo) del pedido en los servidores de Rappi.
    *   **Log:** `Platform Communication Error: Rappi branchRejectOrder [ORDER_ID]`
    *   **customDataAfter:** action: `branchRejectOrder`, platformId: `2`, order: (4 campos), request: (url/body/header), response: .


## 4. PedidosYa (`pedidosYa.js`)

*   **Descripcion:** Error al obtener el token de acceso (login) desde el servidor de PedidosYa.
    *   **Log:** `Platform Communication Error: PedidosYa getAccessToken`
    *   **customDataAfter:** action: `getAccessToken`, platformId: `1`, order: `null`, request: (url/body), response: .

*   **Descripcion:** Error al notificar la aceptación del pedido a PedidosYa mediante el flujo de recepción automática.
    *   **Log:** `Platform Communication Error: PedidosYa receiveOrder [ORDER_ID]`
    *   **customDataAfter:** action: `receiveOrder`, platformId: `1`, order: (5 campos), request: (url/body/header), response: .

*   **Descripcion:** Error al enviar la confirmación manual de aceptación del pedido a la plataforma PedidosYa.
    *   **Log:** `Platform Communication Error: PedidosYa confirmOrder [ORDER_ID]`
    *   **customDataAfter:** action: `confirmOrder`, platformId: `1`, order: (5 campos), request: (url/body/header), response: .

*   **Descripcion:** Error al solicitar el rechazo del pedido a través del callback de rechazo de PedidosYa.
    *   **Log:** `Platform Communication Error: PedidosYa branchRejectOrder [ORDER_ID]`
    *   **customDataAfter:** action: `branchRejectOrder`, platformId: `1`, order: (4 campos), request: (url/body/header), response: .

*   **Descripcion:** Error al informar a PedidosYa que el pedido se encuentra preparado y listo para su entrega.
    *   **Log:** `Platform Communication Error: PedidosYa readyOrder [ORDER_ID]`
    *   **customDataAfter:** action: `readyOrder`, platformId: `1`, order: (4 campos), request: (url/body/header), response: .

*   **Descripcion:** Error al informar el despacho (retiro) del pedido, habitualmente para logística propia o pickup.
    *   **Log:** `Platform Communication Error: UberEats receiveOrder [ORDER_ID]`
    *   **customDataAfter:** action: `dispatchOrder`, platformId: `1`, order: (4 campos), request: (url/body/header), response: .

<div class="note">
  <p><strong>Nota (Bug Detectado):</strong> En el código de PedidosYa, el mensaje de error menciona "UberEats receiveOrder" pero la acción real es <strong>dispatchOrder</strong> de PedidosYa.</p>
</div>

## 5. ThirdParty / I+D (`thirdParty.js`)

*   **Descripcion:** Error al intentar obtener el token de acceso para el servicio de **SmartLoyalty** (Login SML).
    *   **Log:** `Falló SMLLogin`
    *   **customDataAfter:** platformId: `thridParty`, request: (url/body).

*   **Descripcion:** Error al informar el estado de recepción del pedido al servicio de **SmartLoyalty**.
    *   **Log:** `Sml Communication Error: SML receiveOrder [ORDER_ID]`
    *   **customDataAfter:** action: `receiveOrder`, platformId: `12`, order: (5 campos), request: (url/bodySml/headerssml), response: .

*   **Descripcion:** Error al notificar la recepción del pedido a la plataforma de integración I+D.
    *   **Log:** `Platform Communication Error: I+D receiveOrder [ORDER_ID]`
    *   **customDataAfter:** action: `receiveOrder`, platformId: `12`, order: (5 campos), request: (url/body/header), response: .

*   **Descripcion:** Error al informar que el pedido ha sido visualizado (viewed) en la plataforma I+D.
    *   **Log:** `Platform Communication Error: I+D viewOrder [ORDER_ID]`
    *   **customDataAfter:** action: `viewOrder`, platformId: `12`, order: (4 campos), request: (url/body/headers), response: .

*   **Descripcion:** Error al informar la confirmación del pedido al servicio de **SmartLoyalty**.
    *   **Log:** `Sml Communication Error: SML confirmOrder [ORDER_ID]`
    *   **customDataAfter:** action: `confirmOrder`, platformId: `12`, order: (5 campos), request: (url/bodySml/headerssml), response: .

*   **Descripcion:** Error al enviar la confirmación del pedido a la plataforma I+D.
    *   **Log:** `Platform Communication Error: I+D confirmOrder [ORDER_ID]`
    *   **customDataAfter:** action: `confirmOrder`, platformId: `12`, order: (5 campos), request: (url/body/header), response: .

*   **Descripcion:** Error al informar el despacho (enviado) del pedido a la plataforma I+D.
    *   **Log:** `Platform Communication Error: I+D dispatchOrder [ORDER_ID]`
    *   **customDataAfter:** action: `dispatchOrder`, platformId: `12`, order: (4 campos), request: (url/body/headers), response: .

*   **Descripcion:** Error al informar que el pedido ha sido entregado exitosamente a la plataforma I+D.
    *   **Log:** `Platform Communication Error: I+D deliveryOrder [ORDER_ID]`
    *   **customDataAfter:** action: `deliveryOrder`, platformId: `12`, order: (4 campos), request: (url/body/headers), response: .

*   **Descripcion:** Error al notificar el rechazo del pedido al servicio de **SmartLoyalty** para la gestión de puntos.
    *   **Log:** `Sml Communication Error: SML branchRejectOrder [ORDER_ID]`
    *   **customDataAfter:** action: `branchRejectOrder`, platformId: `12`, order: (4 campos), request: (url/bodySml/headerssml), response: .

*   **Descripcion:** Error al solicitar la cancelación del pedido en la plataforma I+D por rechazo del local.
    *   **Log:** `Platform Communication Error: I+D branchRejectOrder [ORDER_ID]`
    *   **customDataAfter:** action: `branchRejectOrder`, platformId: `12`, order: (4 campos), request: (url/body/headers), response: .


## 6. MercadoPago (`mercadoPago.js`)

*   **Descripcion:** Error crítico al intentar obtener el token de acceso de **MercadoPago** desde la infraestructura Cloud propia.
    *   **Log:** `Error fetching token from Cloud`
    *   **customDataAfter:** platformId: `MercadoPago`, details: (error.message).

*   **Descripcion:** Error al notificar la aceptación del pedido a **MercadoPago** durante el flujo de recepción automática.
    *   **Log:** `Platform Communication Error: MercadoPago receiveOrder [ORDER_ID]`
    *   **customDataAfter:** action: `receiveOrder`, platformId: `8`, order: (5 campos), request: (url/body/header), response: .

*   **Descripcion:** Error al enviar la confirmación manual de aceptación del pedido a la plataforma **MercadoPago**.
    *   **Log:** `Platform Communication Error: MercadoPago confirmOrder [ORDER_ID]`
    *   **customDataAfter:** action: `confirmOrder`, platformId: `8`, order: (5 campos), request: (url/body/header), response: .

*   **Descripcion:** Error al solicitar la cancelación (rechazo) del pedido en los servidores de **MercadoPago** por parte del local.
    *   **Log:** `Platform Communication Error: MercadoPago branchRejectOrder [ORDER_ID]`
    *   **customDataAfter:** action: `branchRejectOrder`, platformId: `8`, order: (5 campos), request: (url/body/header), response: .

---

## Propuesta de Estandarización: Estructura "Metadata-First"

Para optimizar la observabilidad y facilitar las consultas en MongoDB, se propone estandarizar los logs bajo una estructura donde la metadata crítica sea de primer nivel dentro del objeto `customDataAfter`.

### Objetivo
Permitir filtrados rápidos por tipo de acción, plataforma u orden sin depender de análisis de texto (Regex) sobre el campo `message`.

### Estructura Propuesta del Log

```json
{
  "message": "Platform Communication Error: [ACTION] [PLATFORM] [ORDER_ID]", 
  "error": {
    "message": "Mensaje de error original (ej: timeout o status 400)",
    "customDataAfter": {
      "action": "actionName", // Identificador único: confirmOrder, dispatchOrder, etc.
      "platformId": "platformId", // Nombre de la plataforma en minúsculas
      "order": {
        "id": "",
        "originalId": "",
        "statusId": 0,
        "platformId": 0,
        "ownDelivery": false,
        "preOrder": false,
        "branchId": 0,
        "country": ""
      },
      "request": {
        "url": "URL de la peticion",
        "method": "POST/PUT",
        "body": { "datos": "enviados" }
      },
      "response": {
        "status": 0,
        "data": { "respuesta": "de la plataforma" }
      }
    }
  }
}
```

### Ejemplo Real de Log Estandarizado en MongoDB

```json
{
  "_id": {
    "$oid": "69a0a0aabbedbfab3babf4e7"
  },
  "message": "Platform Communication Error: PediGrido branchRejectOrder 4541267",
  "error": {
    "error": "Error: Request failed with status code 400",
    "message": "Request failed with status code 400",
    "stack": "Error: Request failed with status code 400\n    at createError (/usr/src/app/node_modules/axios/lib/core/createError.js:16:15)\n    at settle (/usr/src/app/node_modules/axios/lib/core/settle.js:17:12)\n    at IncomingMessage.handleStreamEnd (/usr/src/app/node_modules/axios/lib/adapters/http.js:269:11)\n    at IncomingMessage.emit (events.js:412:35)\n    at IncomingMessage.emit (domain.js:475:12)\n    at endReadableNT (internal/streams/readable.js:1333:12)\n    at processTicksAndRejections (internal/process/task_queues.js:82:21)",
    "customDataAfter": {
      "action": "branchRejectOrder",
      "platformId": 7,
      "order": {
        "id": "4541267",
        "originalId": "4541267",
        "statusId": 5,
        "branchId": 4808
      },
      "request": {
        "url": "https://app.pedigrido.com/v1/Grido/CancelarPedido",
        "body": {
          "Token": "262E87DA4933F471",
          "IdPedido": "4541267",
          "Motivo": 19
        },
        "header": {
          "Content-Type": "application/json"
        }
      }
    }
  },
  "createdAt": {
    "$date": "2026-02-26T19:36:10.657Z"
  }
}
```
