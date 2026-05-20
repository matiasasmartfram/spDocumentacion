---
layout: default
title: DeliveryType — Reglas de Negocio
subtitle: Descripción de los valores del campo deliveryType y su impacto en la facturación del costo de envío.
---

## Valores posibles de `deliveryType`

El campo **`deliveryType`** define quién es responsable de gestionar la entrega del pedido. Existen tres valores posibles:

| Valor | Significado |
|---|---|
| **`forPickup`** | El cliente retira el pedido en el local. No hay delivery. |
| **`forRestaurant`** | El delivery es gestionado por el local. La tienda se hace cargo del envío. |
| **`forLogistics`** | El delivery es gestionado por la plataforma (PedidosYa, Rappi, etc.). |

---

## `order.payment.shipping` — Quién factura el costo de envío

El campo **`order.payment.shipping`** indica el costo de delivery que **el local debe facturar al usuario final**. Su valor depende directamente del **`deliveryType`** asignado al pedido.

### `forLogistics`

La plataforma externa gestiona el delivery y le cobra el envío directamente al usuario.

**`order.payment.shipping` siempre es `0`.**

El local no tiene nada que facturar por ese concepto. El costo es absorbido y cobrado íntegramente por la plataforma (PedidosYa, Rappi, etc.).

### `forRestaurant`

El local gestiona su propio delivery. La plataforma **no** cobra el envío al usuario; esa responsabilidad recae sobre el local.

**`order.payment.shipping` puede ser mayor a `0`.**

Cuando el valor es mayor a `0`, ese monto es el costo de delivery que **el local debe facturar al cliente**. Es responsabilidad de la tienda gestionar y cobrar ese monto.

<div class="note">
  <p><strong>Nota:</strong> Un pedido <strong>forRestaurant</strong> puede tener <strong>order.payment.shipping = 0</strong>. Esto ocurre en situaciones puntuales, por ejemplo cuando el local o la plataforma define envío gratuito para esa orden. No invalida la regla general.</p>
</div>

### `forPickup`

No hay delivery. El cliente retira el pedido en el local.

**`order.payment.shipping` es `0`.**

---

## Resumen

| **`deliveryType`** | **`order.payment.shipping`** | **Quién factura el envío** |
|---|---|---|
| `forLogistics` | Siempre `0` | La plataforma (directamente al usuario) |
| `forRestaurant` | **Puede ser `> 0`** (caso excepcional: `0`) | **El local** |
| `forPickup` | Siempre `0` | Nadie (no hay delivery) |
