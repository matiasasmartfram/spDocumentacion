---
layout: default
title: "Análisis Técnico: Tipo de Delivery"
subtitle: "Reglas para Pedigrido y PedidosYa"
---

Este documento detalla las reglas de negocio aplicadas para resolver la modalidad de entrega en las integraciones de **Pedigrido** (Interfaz ThirdParty) y **PedidosYa**, basándose en el mapeo de datos de entrada hacia el sistema interno.

## **1. Interfaz PediGrido / ThirdParty**

**Ubicación:** `api/src/platforms/interfaces/thirdParty.js` -> `orderMapper()`

La resolución en esta interfaz se basa en flags booleanos directos. El sistema aplica una jerarquía donde el retiro tiene prioridad sobre la logística de envío.

### **🛠️ Variables de Entrada**

| Campo Original (`data.order`) | Atributo Interno | Definición |
| :--- | :--- | :--- |
| `pickup` | `pickupOnShop` | Indica si el cliente retira por el local. |
| `logistics` | `ownDelivery` | Indica si el local se encarga del envío (Delivery Propio). |

### **📊 Matriz de Resolución**

| Combinación | `pickup` | `logistics` | `pickupOnShop` | `ownDelivery` | Resolución / Escenario |
| :---: | :---: | :---: | :---: | :---: | :--- |
| **1** | `true` | *(Cualquiera)* | ✅ **true** | ❌ **false** | **Retiro (Take Away):** Prioridad absoluta. Se fuerza false en logística. |
| **2** | `false` | `true` | ❌ **false** | ✅ **true** | **Delivery Propio:** Coordinado y realizado por el local. |
| **3** | `false` | `false` | ❌ **false** | ❌ **false** | **Delivery Plataforma:** Logística a cargo de un tercero. |

<div class="note">
  <p><strong>Regla de Prioridad:</strong> En la interfaz ThirdParty, si <strong>pickup</strong> es <strong>true</strong>, el sistema ignora el valor de <strong>logistics</strong> y marca la orden como retiro únicamente.</p>
</div>

## **2. Interfaz PedidosYa**

**Ubicación:** `api/src/platforms/interfaces/pedidosYa.js` -> `orderMapper()`

Para PedidosYa, el factor determinante no es un flag de logística, sino la **asignación de un repartidor** (Rider) por parte de la plataforma.

### **🛠️ Variables de Entrada**

| Campo Original (`data.order`) | Atributo Interno | Definición |
| :--- | :--- | :--- |
| `pickup` | `pickupOnShop` | Objeto presente solo en órdenes de Retiro. |
| `delivery.riderPickupTime` | `ownDelivery` | **Clave:** Si es **vacío (null)**, el envío es propio. |

### **📊 Matriz de Resolución (Validada por Pruebas)**

| Escenario | `pickup` (Objeto) | `riderPickupTime` | `pickupOnShop` | `ownDelivery` | Resultado / Resolución |
| :--- | :---: | :---: | :---: | :---: | :--- |
| **Retiro (Take Away)** | **Existe** | **≠ null** | ✅ **true** | ❌ **false** | **Retira cliente.** Logística de plataforma (no usada). |
| **Delivery Peya** | **No existe** | **≠ null** | ❌ **false** | ❌ **false** | **Logística Peya:** Repartidor asignado con hora de retiro. |
| **Híbrido (Propio)** | **Existe** | **null** | ✅ **true** | ✅ **true** | **Delivery Propio:** Con bandera de retiro activa en sistema. |
| **Delivery Propio** | **No existe** | **null** | ❌ **false** | ✅ **true** | **Delivery Propio:** El local gestiona la entrega. |

<div class="note">
  <p><strong>Conclusión de las Pruebas:</strong> La variable <strong>ownDelivery</strong> (Logística Propia) en PedidosYa depende <strong>exclusivamente</strong> de que <strong>riderPickupTime</strong> sea <strong>vacío (null)</strong>, independientemente de si el objeto <strong>pickup</strong> existe o no en el mensaje.</p>
</div>
