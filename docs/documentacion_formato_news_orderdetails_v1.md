---
layout: default
title: "Documentación de Integración: Formato de Pedidos (news) para POS - Pedidos Ya"
subtitle: "Documentación de Integración: Formato de Pedidos (news) para el POS"
---

## **Propósito del Documento**

Este documento describe la estructura de datos del objeto `news`.

<div class="note">
  <p><em>Nota: Este formato es el que está en producción y no incluye las modificaiones futras.</em></p>
</div>

## **1. Estructura General del Objeto `news`**

Cada nuevo pedido se envía encapsulado en un objeto JSON principal `news`.

#### **Ejemplo de Estructura:**

```json
{
  "_id": "...",
  "order": { ... },
  "branchId": 35,
  "extraData": { ... },
  "typeId": 1
}
```

## **2. El Objeto `order`**

Dentro de `news`, el objeto `order` contiene los datos transaccionales del pedido. Las claves principales son:

*   `id`, `originalId`, `displayId`: Identificadores del pedido.
*   `platformId`: ID interno de la plataforma (ej. 1 para PedidosYa).
*   `orderTime`, `deliveryTime`: Fechas y horas.
*   `observations`: Notas adicionales.
*   `customer`: Objeto con los datos del cliente.
*   `payment`: Objeto con el detalle del pago.
*   `details`: **Array de objetos que representa el detalle de los productos del pedido.**

## **3. El Array `order.details`:**

El contenido y la estructura del array `details` dependen de cómo está catalogado el producto en la plataforma de origen. Tiene dos comportamientos distintos:

### **Caso A: Productos "Normales" (Sin Agrupación Padre-Hijo)**

Este es el comportamiento por defecto para cualquier producto cuyo nombre **NO** comienza con la palabra "promo".

#### **Características del Mapeo:**

*   **Ítem Único:** Por cada producto pedido, se genera un solo objeto en el array `details`.
*   **`groupId` Fijo:** El campo `groupId` siempre tiene el valor estático `"0"`.
*   **Modificadores Aplanados:** Toda la información de los adicionales se concatena en una única cadena de texto en el campo `optionalText`.
*   **Asignación de SKU:** El `sku` del producto principal es sobreescrito con el `remoteCode` del **último** modificador procesado.

#### **Ejemplo:**

```json
"details": [
  {
    "productId": "64",
    "count": "1",
    "price": 11000,
    "promo": 0,
    "groupId": "0",
    "description": "1 Kilo",
    "note": "",
    "sku": "64",
    "optionalText": " Frutilla Vainilla Chocolate Limon"
  }
]
```

### **Caso B: Productos de tipo "Promoción" (Con Agrupación Padre-Hijo)**

Este comportamiento se activa específicamente cuando el nombre del producto padre comienza con "promo" (insensible a mayúsculas/minúsculas).

#### **Características del Mapeo:**

*   **Múltiples Ítems:** Un solo producto "promo" en el pedido original genera múltiples objetos en el array `details`.
*   **`groupId` para Agrupar:** Se genera un `groupId` numérico (ej: "1") que es compartido por el producto principal y todos sus componentes.
*   **Ítem Padre ("Header"):** Se crea un primer ítem que representa la promoción, identificado con el flag `promo: 2`.
*   **Ítems Hijo (Componentes):** Cada componente o elección dentro de la promoción es un ítem separado, identificado con el flag `promo: 1`.
*   **SKU Individual:** Cada ítem (padre e hijos) tiene su propio `sku` y `description` correctos.

#### **Ejemplo:**

```json
"details": [
  // --- Ítem Padre (Header de la Promo) ---
  {
    "productId": "123",
    "count": "1",
    "price": 5000,
    "promo": 2,
    "groupId": "1",
    "description": "Promo 2 1/4kg Helado",
    "sku": "123"
  },
  // --- Ítem Hijo 1 (Primer 1/4) ---
  {
    "productId": "66",
    "promo": 1,
    "groupId": "1",
    "description": "Sabores Helado 1/4 KG",
    "sku": "66"
    "optionalText": " Dulce de Leche X 1 Tramontana X 1"
  },
  // --- Ítem Hijo 2  (Primer 1/4) ---
  {
    "productId": "66",
    "promo": 1,
    "groupId": "1",
    "description": "Sabores Helado 1/4 KG",
    "sku": "66"
    "optionalText": " Chocolate X 1 Vainilla X 1"
  }
]
```

## **4. Resumen de Campos en `details`**

| Campo | Descripción |
| :--- | :--- |
| `productId` | ID del producto o modificador en la plataforma de origen. |
| `sku` | El SKU que el POS debe utilizar. |
| `description` | Nombre del producto o modificador. |
| `count` | Cantidad del ítem. |
| `price` | Precio unitario del ítem. |
| `groupId` | Usado para agrupar ítems. Es `"0"` para productos normales y un número (`"1"`, `"2"`...) para agrupar promos. |
| `promo` | Flag numérico. `0` para ítems normales, `2` para el padre de una promo, `1` para los hijos de una promo. |
| `optionalText` | Contiene una cadena de texto con todos los modificadores concatenados. |
| `note` | Comentarios específicos del producto, enviados por el cliente. |
