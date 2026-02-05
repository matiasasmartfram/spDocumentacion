---
layout: default
title: "Análisis Técnico: Proceso de Rechazo Automático (orderRejClosed)"
subtitle: "Funcionamiento del cron de rechazo automático en el Concentrador"
---

Este documento detalla el funcionamiento del cron de rechazo automático en el Concentrador, diseñado para gestionar pedidos que no pudieron ser entregados al punto de venta (POS) debido a desconexión o cierre del local.

## **1. Objetivo del Proceso**

El proceso `orderRejClosed` actúa como una red de seguridad de última instancia. Su función es evitar que un pedido quede en un "limbo" de espera cuando el Concentrador lo ha recibido pero la tienda (POS) no lo ha procesado ni reclamado.

## **2. Flujo de Ejecución**

El proceso se divide en dos fases: la **captura** del pedido y la **ejecución** del rechazo.

### **Fase A: Registro del Pendiente (`Captura`)**

Cuando el sistema detecta una anomalía (ej. tienda cerrada o error de comunicación), se crea un registro en la colección `orderrejcloseds`.

*   **Ubicación en el código:** `api/src/controllers/branch.js` (Línea ~335)
*   **Lógica:**

```javascript
await orderRejClosedModel.create({ 
    newId: result._id, 
    type: "ordersToSendPOS" // Marca el pedido para revisión posterior
});
```

### **Fase B: El Cron de Limpieza (`Ejecución`)**

Un cron job revisa periódicamente estos registros para determinar si deben ser rechazados definitivamente.

*   **Frecuencia de revisión:** Cada **30 segundos**.
*   **Configuración del Cron:** `*/30 * * * * *` (Línea 968 de `branch.js`).
*   **Función Principal:** `orderRejClosed()` (Línea 1327).

## **3. Lógica del Tiempo de Gracia (+1 minuto)**

Existe una diferencia entre el tiempo configurado en la base de datos (`orderRejClosedTime`) y el tiempo real de espera.

*   **Configuración en `configs`:** `orderRejClosedTime: 2` (minutos).
*   **Fórmula en el Código (Línea 1331):**

```javascript
const date2minutes = new Date(dateNow.getTime() - (orderRejClosedTime.orderRejClosedTime + 1) * 60 * 1000);
```

**¿Por qué espera 3 minutos si la configuración dice 2?**
El código suma un minuto extra (`+ 1`) como margen de seguridad. Esto garantiza que la tienda tenga el tiempo configurado íntegro para reconectarse antes de que el Concentrador tome la decisión de rechazar.

### **Ejemplo Práctico:**

| Evento | Hora | Acción del Sistema |
| :--- | :--- | :--- |
| **Ingreso del Pedido** | 10:00:00 | Se crea `orderrejcloseds` con `createdAt: 10:00:00`. |
| **Configuración** | - | `orderRejClosedTime = 2`. Umbral = 2 + 1 = **3 min**. |
| **Ejecución Cron 1** | 10:02:15 | Calcula umbral (10:02:15 - 3m = 09:59:15). 10:00 no es menor a 09:59. **No hace nada.** |
| **Ejecución Cron 2** | 10:03:05 | Calcula umbral (10:03:05 - 3m = 10:00:05). 10:00 **es menor** a 10:00:05. |
| **Rechazo Final** | 10:03:05 | El pedido se rechaza automáticamente. |

## **4. Detalles del Rechazo Automático**

Cuando el cron decide ejecutar el rechazo, realiza las siguientes acciones en la colección `news` y hacia la plataforma:

1.  **Estado del Pedido:** Se cambia a `statusId: 2` (Rechazado).
2.  **Motivo del Rechazo:** Se asigna el ID `-3` (Línea 1352).
    *   Este motivo corresponde internamente a fallas del Concentrador o Local Cerrado.
3.  **Identificación de Entidad:** El log se marca con `entity: 'CONCENTRADOR'`.
    *   Esto permite distinguir en los reportes que el rechazo no fue manual por un usuario, sino automático por el sistema.
4.  **Notificación Externa:** Se dispara la función `sendReject(element, platformId)` para informar a PedidosYa/Rappi/Uber.

## **5. Relación Crítica con `lastGetNews`**

Este proceso depende directamente de la salud de la conexión de la tienda, la cual se mide mediante el campo `lastGetNews`.

1.  **Monitor de conexión:** Las funciones `checkOfflinePOS...` (como `checkOfflinePOSpedigrido`) comparan continuamente el `lastGetNews` con la hora actual.
2.  **Cierre por Inactividad:** Si una tienda no realiza pedidos o no consulta novedades por más de 1 minuto, el sistema la marca como offline (`alreadyClosedPediGrido: true`).
3.  **Encadenamiento:** Una vez que la tienda está marcada como offline, cualquier pedido nuevo que ingrese caerá automáticamente en la lógica de `orderrejcloseds`, iniciando la cuenta regresiva de los 3 minutos descritos anteriormente.

<div class="note">
  <p><strong>Nota Técnica:</strong> Este mecanismo asegura la experiencia del cliente final, garantizando que si una tienda perdió internet o cerró su software sin avisar, el pedido no quede "colgado" y sea devuelto a la plataforma para que el cliente sea notificado del rechazo.</p>
</div>
