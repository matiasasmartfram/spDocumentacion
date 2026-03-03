---
layout: default
title: Sistema de Reintento de Órdenes
subtitle: Documentación del proceso asíncrono de confirmación y rechazo de pedidos
---

## Propósito
Su objetivo es garantizar que las confirmaciones o rechazos de pedidos que no pudieron procesarse en tiempo real (En el Platform) (debido a fallos de red, timeouts o procesos asíncronos deliberados) se completen de manera efectiva.

## Ubicación y Ejecución
- **Base de Datos**: MongoDB, colección `rejectPeyas`.
- **Lógica de Control**: Función `rejectOrdersPeya()`. (Concentrador)
- **Programador de Tareas (Cron Job)**: Se ejecuta cada **2 minutos** (`*/2 * * * *`).

## Lógica de Funcionamiento
El proceso se dispara automáticamente y sigue un flujo para garantizar que cada registro sea procesado una única vez.

### 1. Selección de Registros
El sistema consulta la colección `rejectPeyas` buscando todos los documentos donde el campo `send` sea `false`.

### 2. Procesamiento por Plataforma
Dependiendo del `platformId` asociado al registro, el sistema construye los encabezados y el cuerpo del mensaje específicos requeridos por la API de la plataforma:

| Platform ID | Plataforma | Descripción de Lógica |
| :--- | :--- | :--- |
| **1** | **PedidosYa** | Utiliza `acceptanceTime` y envía el estado `order_accepted`. |
| **2** | **Rappi** | Requiere el token específico por país (`country`). |
| **4** | **UberEats** | Envía el campo `ready_for_pickup_time`. |
| **7** | **PediGrido** | Incluye `Token`, `IdPedido` y `Demora`. Si falla y hay canje, dispara devolución. |
| **8** | **Mercado Pago** | Realiza un flujo complejo de obtención y desencriptación de tokens vía Cloud. |
| **12** | **I+D** | Similar a PediGrido, utiliza `tokenID` y gestiona devoluciones de canje. |

### 3. Finalización
Una vez que se intenta la comunicación con la plataforma (sea exitosa o falle), el sistema marca el registro con `send: true`. Esto es **importante** para evitar que un pedido con datos corruptos o errores persistentes genere un bucle infinito en el cron job.

<div class="note">
  <p><strong>Nota:</strong> En casos donde la comunicación falla para plataformas como <strong>PediGrido (7)</strong> o <strong>I+D (12)</strong>, y el pedido original contiene puntos de canje, el sistema ejecuta automáticamente la función <code>sendRejectsToCanje</code> para revertir la operación en SmartLoyalty y envia un "Fallo de comunicación con la plataforma" (id:-10) a la plataforma y se informa el rechazo al POS.</p>
</div>

## Estructura de Datos (rejectPeyas)
A continuación se detallan los campos clave que debe contener un registro para ser procesado correctamente:

```json
{
  "platformId": 1, // ID de la plataforma
  "url": "https://api.pedidosya.com/v1/orders/123/confirm", // URL del endpoint para confirmar en la Plataforma.
  "orderId": "123456", // ID del pedido
  "send": false, // Estado del registro
  "extraData": {
    "readyForPickup": "2023-10-27T10:00:00Z", // Para PedidosYa
    "Demora": 30, // Para PediGrido
    "country": "AR" // Para Rappi
  }
}
```
