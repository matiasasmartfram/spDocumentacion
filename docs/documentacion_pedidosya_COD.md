---
layout: default
title: "Lógica de Cash on Delivery (COD) en PedidosYa"
subtitle: "Documentación técnica basada en interfaces/pedidosYa.js"
---

Esta documentación explica de manera clara y detallada cómo funciona la lógica para detectar y aplicar **Cash on Delivery** (pago contra entrega, también llamado **Cash Collection**) en los pedidos de PedidosYa. Está basada estrictamente en el código real de `api/src/platforms/interfaces/pedidosYa.js`, explicando paso a paso qué hace cada parte para que tanto técnicos como no técnicos entiendan la funcionalidad. Incluye fragmentos de código relevantes para mayor claridad.

## **¿Qué es Cash Collection (COD)?**

Cash Collection es un modelo especial de PedidosYa donde:

*   El cliente paga en efectivo al repartidor cuando recibe el pedido.
*   PedidosYa se encarga de recoger ese dinero y depositarlo para que la sucursal (franquiciado) lo cobre posteriormente, como si fuera un pago online.
*   La sucursal **NO** recibe el efectivo directamente, evitando riesgos y simplificando la contabilidad.
*   Aunque el pago original en la app sea "Efectivo", se trata internamente como un "Pago Online" para el sistema.

Esto resuelve el problema de que la sucursal no maneje dinero en efectivo, delegando esa gestión a la plataforma.

## **Detección de un Pedido COD**

Un pedido se detecta como COD cuando cumple **exactamente** estas condiciones en los datos crudos de PedidosYa:

```json
{
  "payment": {
    "type": "Efectivo"
  },
  "price": {
    "payRestaurant": "0"
  }
}
```

*   `payment.type` debe ser exactamente "Efectivo" (o variaciones como "cash", pero el código busca "efectivo" en minúsculas).
*   `price.payRestaurant` debe ser "0", indicando que la sucursal no paga nada por el servicio de entrega.

<div class="note">
  <p><strong>Nota importante:</strong> Si <code>price</code> o <code>price.payRestaurant</code> no existen en los datos, el código usa un valor por defecto de "1" para evitar fallos (mejorando la robustez del parser).</p>
</div>

En el código, esto se traduce a tratar el pago como "Pago Online" / Crédito, aunque originalmente sea efectivo.

## **Lógica Principal: Clasificación del Tipo de Pago**

Esta parte del código (función `paymentenMapper`) decide cómo clasificar el pago en el sistema interno (news). Aquí va paso a paso:

1.  **Normalización de datos** (para evitar errores por mayúsculas o espacios):
    ```javascript
    let paymentTemp = payment.type.toLowerCase().trim();
    let paymentStatusTemp = payment.status.toLowerCase().trim();
    ```

2.  **Obtención segura del valor payRestaurant**:
    ```javascript
    let payRestaurantValue = String(price?.payRestaurant ?? "1");
    ```
    *   Usa "1" por defecto si no existe, haciendo el código más resistente a datos faltantes.

3.  **Verificación si es efectivo**:
    ```javascript
    if ((paymentTemp.includes("cash") || paymentTemp.includes("efectivo")) && payRestaurantValue !== "0") {
      paymentNews.typeId = paymentType.Efectivo.paymentId; // 3
      // Cálculos de remaining y partial...
    } else {
      paymentNews.typeId = paymentType.CREDIT.paymentId; // 2
    }
    ```
    *   Si incluye "cash" o "efectivo" Y payRestaurant no es "0", se marca como **Efectivo** (tipo 3).
    *   De lo contrario, **Crédito** (tipo 2).

4.  **Determinación si es online**:
    ```javascript
    paymentNews.online = (paymentStatusTemp === 'paid' || paymentStatusTemp === 'pagado') || !((paymentTemp.includes("cash") || paymentTemp.includes("efectivo")) && payRestaurantValue !== "0");
    ```
    *   Es online si ya está pagado, o si **NO** cumple las condiciones de efectivo.

5.  **Forzar a Crédito si es online**:
    ```javascript
    if (paymentNews.online) paymentNews.typeId = paymentType.CREDIT.paymentId;
    ```
    *   Aunque fuera efectivo, si es online, se cambia a Crédito para manejarlo como pago digital.

**Resultado para COD**: Cuando se detecta COD (efectivo + payRestaurant="0"), se marca como online y Crédito, tratándolo como pago en la app.

## **Determinación de Entrega Propia (ownDelivery)**

Para evitar activar COD en casos donde la sucursal gestiona su propia entrega (lo que causaría problemas, como en el caso de la orden 1605610275).

**Definición en código**:

```javascript
const hasDelivery = data.order.delivery != null;
const noRiderPickupTime = data.order.delivery?.riderPickupTime == null;
const hasPickup = data.order.pickup != null;
const ownDelivery = hasDelivery && noRiderPickupTime && hasPickup;
```

*   `ownDelivery` es true solo si: existe `delivery`, `riderPickupTime` es null (no hay repartidor de PedidosYa), Y existe `pickup`.

**Integración con COD**:
Aunque no hay un `if (ownDelivery) return;` explícito en el código actual, la lógica posterior impide activar Cash Collection en estos casos ajustando el pago a offline/efectivo.

## **Ajuste en las Notas del Pedido (customerComment)**

El problema: Las notas del pedido usan `customerComment` en "observaciones", y si dice "Medio de pago: Efectivo", puede confundir a la sucursal sobre quién cobra.

**Estructura en datos crudos**:

```json
"comments": {
  "customerComment": "Medio de pago: Efectivo | Paga con: 8339.0"
}
```

**Lógica de reemplazo** (solo si es COD y no ownDelivery):

```javascript
if (news.order.observations.includes('Efectivo') && data.order.price.payRestaurant === "0") {
  if (!news.order.ownDelivery) {
    news.order.observations = news.order.observations.replace(/Medio de pago:\s*Efectivo/i, "Medio de pago: Cash Collection");
  } else {
    news.order.payment.online = false;
    news.order.payment.typeId = paymentType.Efectivo.paymentId;
  }
}
```

*   Si incluye "Efectivo" y payRestaurant="0":
    *   Si **NO** es ownDelivery: cambia "Efectivo" por "Cash Collection" en las observaciones.
    *   Si es ownDelivery: deja las notas, pero marca pago como offline y efectivo (evitando COD).

Esto informa claramente a la sucursal que es un cobro gestionado por PedidosYa.

## **Ejemplos Prácticos**

| Escenario | ownDelivery | payRestaurant | customerComment Original | Resultado en News |
| :--- | :--- | :--- | :--- | :--- |
| **PedidosYa + COD válido** | false | "0" | "Medio de pago: Efectivo..." | Tipo: Crédito (online); Observaciones: "Medio de pago: Cash Collection..." |
| **Entrega Propia + Efectivo normal** | true | "1500" | "Medio de pago: Efectivo..." | Tipo: Efectivo (según lógica); Sin cambios en observaciones |
| **Entrega Propia + Intento COD** | true | "0" | "Medio de pago: Efectivo..." | Tipo: Efectivo (offline); Sin cambios en observaciones (evita COD) |
| **PedidosYa + No efectivo** | false | "0" | Sin "Efectivo" | Tipo: Crédito; Sin cambios |

## **Notas Finales**

*   **Robustez**: El código maneja datos faltantes usando valores por defecto (ej. payRestaurant ?? "1"), evitando fallos.
*   **Criterios de aceptación**: Detecta COD solo con `payment.type == "Efectivo"` y `price.payRestaurant == "0"`, registrándolo como Pago Online/Crédito.
*   **Campo adicional opcional**: Las observaciones indican que es COD gestionado por PedidosYa.
*   **Beneficio**: La sucursal cobra como pago online, sin manejar efectivo, mientras el repartidor lo deposita en el sistema.

Esta lógica asegura que COD funcione correctamente solo en entregas de PedidosYa, previniendo errores en entregas propias y manteniendo claridad en las comandas.
