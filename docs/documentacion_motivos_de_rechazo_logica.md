---
layout: default
title: Motivos de Rechazo y Lógica de Logística
subtitle: Guía técnica sobre la interpretación de atributos logísticos y filtrado de motivos de rechazo en SmartCloud.
---

## Introducción

El objetivo de este documento es definir cómo interpretar los atributos de logística (**pickupOnShop** y **ownDelivery**) para determinar correctamente la modalidad de entrega de cada orden y cómo se filtran los **Motivos de Rechazo** en consecuencia.

## Variables de Salida (Payload de la Orden)

SmartPedidos normaliza la información de logística en dos variables booleanas clave dentro del objeto de la orden:

| Variable | Tipo | Descripción |
| :--- | :---: | :--- |
| **pickupOnShop** | `Boolean` | Indica si la orden debe ser retirada por el cliente en el local. |
| **ownDelivery** | `Boolean` | Indica si la logística de envío está a cargo del propio comercio (sucursal). |

## Matriz de Interpretación de Logística

Para determinar el **Tipo de Entrega** final, el sistema consumidor debe evaluar las combinaciones de estas dos variables en el siguiente orden de prioridad:

| Prioridad | pickupOnShop | ownDelivery | **MODALIDAD DE ENTREGA (Resolución)** | Descripción |
| :---: | :---: | :---: | :--- | :--- |
| 1️⃣ | ✅ **true** | *(Indistinto)* | 🛒 **Retira el Cliente** | El cliente acude al local. Esta condición prevalece sobre cualquier valor de **ownDelivery**. |
| 2️⃣ | ❌ **false** | ✅ **true** | 🛵 **Delivery Propio** | El envío debe ser gestionado y realizado por la logística interna de la Tienda. |
| 3️⃣ | ❌ **false** | ❌ **false** | 📱 **Delivery a Cargo de la Plataforma** | El envío es gestionado por la plataforma de origen (ej. Repartidor de PedidosYa/Rappi). |

<div class="note">
  <p><strong>Nota:</strong> Estas reglas aseguran que no haya ambigüedad en la asignación de la logística, priorizando siempre la intención del cliente de retirar en el local.</p>
</div>

### Resumen de Reglas de Negocio

1.  **Prioridad de Retiro**: Si **pickupOnShop** es verdadero, la orden **SIEMPRE** se considera "Retira el Cliente" (Take Away), ignorando cualquier otro indicador de logística.
2.  **Delivery Propio**: Se activa únicamente cuando **NO** es retiro (**pickupOnShop: false**) y el flag **ownDelivery** está encendido.
3.  **Delivery Plataforma**: Es el escenario por defecto cuando ambos indicadores son falsos (no retira el cliente y el comercio no hace el envío).

## Configuración de Motivos de Rechazo (Rapiboy/Pedigrido)

El sistema filtra los motivos de rechazo (**smartfran.rejectedMessages.json**) basándose en la correspondencia entre los atributos del motivo (**forPickup**, **forLogistics**, **forRestaurant**) y la modalidad de entrega de la orden.

Para que un motivo sea visible en el listado, debe tener su flag correspondiente en **true**.

### Definición de Flags de Rechazo

| Atributo JSON | Descripción | Relación con Logística |
| :--- | :--- | :--- |
| **isActive** | Disponibilidad global | `Boolean` |
| **forPickup** | Visible en Retiro Cliente | `Boolean` |
| **forLogistics** | Visible en Delivery Plataforma | `Boolean` |
| **forRestaurant** | Visible en Delivery Propio | `Boolean` |

### Escenario: Delivery Propio
**Condición de Orden**: `pickupOnShop: false` AND `ownDelivery: true`  
**SmartCloud**: `/api/v1/Reason/ReasonsReject/7?ownDelivery=true`  
**Filtro Aplicado**: Se muestran motivos donde **forRestaurant: true** (y **isActive: true**).

| ID | Motivo | Configuración JSON (platformId: 7) | Análisis |
| :--- | :--- | :--- | :--- |
| **1** | Sin Producto/Variedad | `forRestaurant: true` | ✅ Habilitado |
| **19** | Pedido repetido | `forRestaurant: true` | ✅ Habilitado |
| **12** | Cliente solicita anular el Pedido | `forRestaurant: true` | ✅ Habilitado |
| **10** | Zona No Corresponde | `forRestaurant: true`, `forPickup: false` | ✅ Habilitado (Exclusivo Delivery) |
| **9** | No sale Nadie | `forRestaurant: true`, `forPickup: false` | ✅ Habilitado (Exclusivo Delivery) |
| **11** | Zona fuera de cobertura | `forRestaurant: true`, `forPickup: false` | ✅ Habilitado (Exclusivo Delivery) |
| **4** | Repartidor Accidentado | `forRestaurant: true`, `forPickup: false` | ✅ Habilitado (Exclusivo Delivery) |
| **3** | Sin repartidor | `forRestaurant: true`, `forPickup: false` | ✅ Habilitado (Exclusivo Delivery) |
| **2** | Domicilio Erroneo | `forRestaurant: true`, `forPickup: false` | ✅ Habilitado (Exclusivo Delivery) |

### Escenario: Retira el Cliente (Take Away)
**Condición de Orden**: `pickupOnShop: true`  
**SmartCloud**: `/api/v1/Reason/ReasonsReject/7?pickupOnShop=true` (Sugerido)  
**Filtro Aplicado**: Se muestran motivos donde **forPickup: true** (y **isActive: true**).

| ID | Motivo | Configuración JSON (platformId: 7) | Análisis |
| :--- | :--- | :--- | :--- |
| **1** | Sin Producto/Variedad | `forPickup: true` | ✅ Habilitado |
| **19** | Pedido repetido | `forPickup: true` | ✅ Habilitado |
| **12** | Cliente solicita anular el Pedido | `forPickup: true` | ✅ Habilitado |

<div class="note">
  <p><strong>Nota:</strong> Motivos como "Sin repartidor" (ID 3) tienen **forPickup: false**, por lo que correctamente no aparecen en órdenes de retiro.</p>
</div>

## Configuración de Motivos de Rechazo (PedidosYa)

A continuación se detallan los motivos habilitados para cada escenario logístico en PedidosYa (**platformId: 1**).

### Escenario: Retira el Cliente (Take Away)
**Condición de Orden**: `pickupOnShop: true`  
**Filtro Aplicado**: Se muestran motivos donde **forPickup: true**.

| ID | Motivo | Configuración JSON (platformId: 1) | Análisis |
| :--- | :--- | :--- | :--- |
| **103** | Falta de producto | `forPickup: true` | ✅ Habilitado |
| **107** | Hay mucha demanda en el local | `forPickup: true` | ✅ Habilitado |

<div class="note">
  <p><strong>Nota:</strong> Motivos logísticos como "Mal clima" (ID 101) tienen **forPickup: false**, por lo que no se muestran en este escenario.</p>
</div>

### Escenario: Delivery Propio (Logística del Comercio)
**Condición de Orden**: `pickupOnShop: false` AND `ownDelivery: true`  
**Filtro Aplicado**: Se muestran motivos donde **forRestaurant: true**.

| ID | Motivo | Configuración JSON (platformId: 1) | Análisis |
| :--- | :--- | :--- | :--- |
| **103** | Falta de producto | `forRestaurant: true` | ✅ Habilitado |
| **107** | Hay mucha demanda en el local | `forRestaurant: true` | ✅ Habilitado |
| **106** | Fuera de zona de entrega | `forRestaurant: true`, `forPickup: false`, `forLogistics: false` | ✅ Habilitado (Exclusivo Propio) |
| **105** | No hay repartidor disponible | `forRestaurant: true`, `forPickup: false`, `forLogistics: false` | ✅ Habilitado (Exclusivo Propio) |
| **101** | Mal clima | `forRestaurant: true` | ✅ Habilitado |

### Escenario: Delivery a Cargo de la Plataforma
**Condición de Orden**: `pickupOnShop: false` AND `ownDelivery: false`  
**Filtro Aplicado**: Se muestran motivos donde **forLogistics: true**.

| ID | Motivo | Configuración JSON (platformId: 1) | Análisis |
| :--- | :--- | :--- | :--- |
| **103** | Falta de producto | `forLogistics: true` | ✅ Habilitado |
| **107** | Hay mucha demanda en el local | `forLogistics: true` | ✅ Habilitado |
| **101** | Mal clima | `forLogistics: true` | ✅ Habilitado* |

<div class="note">
  <p><strong>Nota sobre ID 101 (Mal Clima):</strong> Aunque "Mal Clima" tiene **forLogistics: true**, es importante verificar si debe mostrarse efectivamente en este escenario de plataforma.</p>
</div>

---
*Documento generado para la estandarización de procesos logísticos en integraciones con SmartCloud.*
