---
layout: default
title: "Integración OpenClose (POS Online/Offline) - Guía para Terceros"
subtitle: "Lógica y requerimientos para la integración"
---

## **Introducción**

Este documento detalla la lógica y los requerimientos para que plataformas de delivery se integren con SmartPedidos. El objetivo es asegurar que una tienda solo reciba pedidos cuando el POS esté conectado y operativo.

## **Mecanismo de Funcionamiento**

Nuestro sistema monitorea constantemente la conexión del POS para sincronizar su estado de apertura y cierre:

1.  **Heartbeat del POS:** Mientras el POS está funcionando, envía señales periódicas a SmartPedidos.
2.  **Sincronización de Estado:** SmartPedidos detecta cambios en la conexión y envía automáticamente las actualizaciones de estado (OPEN o CLOSED) a su plataforma.
3.  **Tiempos de Respuesta:**
    *   **Para Apertura (OPEN):** El envío de la notificación se realiza en un lapso de **1 minuto** desde que el POS recupera la conexión.
    *   **Para Cierre (CLOSED):** El envío de la notificación se realiza en un intervalo de **mínimo 1 minuto y máximo 2 minutos** desde que se detecta la pérdida de señal.
4.  **Apertura de Tienda:** En cuanto el POS inicia su actividad o recupera la comunicación, SmartPedidos envía la orden de apertura (OPEN).
5.  **Cierre de Tienda:** Si el POS pierde conexión o se apaga, SmartPedidos envía la orden de cierre (CLOSED) tras cumplirse el tiempo de chequeo definido.

## **Requerimientos de Infraestructura y Escalabilidad**

<div class="note">
  <p>Es <strong>muy importante</strong> que la infraestructura de la plataforma tercera esté preparada para recibir un volumen considerable de solicitudes.</p>
  <p>A diferencia de un cierre manual programado, este proceso es dinámico y depende de la estabilidad de la conexión. Esto puede generar ráfagas de solicitudes de cambio de estado (aperturas y cierres seguidos) si una conexión es inestable. Su API debe ser capaz de procesar estas actualizaciones en tiempo real y a escala.</p>
</div>

## **Documentación Técnica de Referencia**

Para implementar esta integración, el tercero debe exponer los siguientes servicios (URLs y payloads a modo de ejemplo):

### **1. Consulta de Estado (GET)**

Permite a SmartPedidos verificar el estado actual de una sucursal en su sistema antes de realizar un cambio.

*   **Endpoint de ejemplo:** `GET /api/branch/status/{BranchId}`

**Respuesta Exitosa:**

```json
{
  "branchId": "5001",
  "availabilityState": "OPEN"
}
```

*(Valores posibles para availabilityState: "OPEN" o "CLOSED")*

### **2. Actualización de Estado (PUT)**

Permite a SmartPedidos informar un cambio de disponibilidad basado en el estado del POS.

*   **Endpoint de ejemplo:** `PUT /api/branch/status/{BranchId}`

**Cuerpo de la solicitud (Payload):**

```json
{
  "availabilityState": "CLOSED"
}
```

**Respuesta Esperada:**

```json
{
  "branchId": "5001",
  "availabilityState": "CLOSED"
}
```

## **Casos de Uso**

### **Escenario A: Apertura**

1.  El POS se activa o recupera internet.
2.  SmartPedidos envía un **PUT** con `"availabilityState": "OPEN"` en un lapso de **1 minuto**.
3.  La tienda queda habilitada para recibir pedidos.

### **Escenario B: Cierre**

1.  El POS pierde señal o se cierra.
2.  SmartPedidos envía un **PUT** con `"availabilityState": "CLOSED"` en un lapso de **1 a 2 minutos máximo**.
3.  La tienda queda inactiva para prevenir órdenes fallidas.
