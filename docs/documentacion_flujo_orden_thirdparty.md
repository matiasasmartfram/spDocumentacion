---
layout: default
title: "Documentación: El Viaje de una Orden de Third Party"
subtitle: "Mapa Conceptual: El Viaje de una Orden de Third Party"
---

Este documento describe el ciclo de vida completo de un pedido proveniente de una plataforma externa (ej. Pedigrido), desde su creación hasta su finalización, explicando cómo interactúan la Plataforma de Servicios y el Punto de Venta (POS).

## **Etapa 1: El Origen - Creación del Pedido**

1.  **Punto de Partida:** Un sistema externo realiza una llamada `POST` a un endpoint de la Plataforma de Servicios.
2.  **Acción Inmediata:** La aplicación recibe la información del pedido.
3.  **Primera Transformación:**
    *   El controlador de la API procesa los datos recibidos.
    *   Convierte la orden en un documento estandarizado para MongoDB, denominado **`news`**.
    *   Se le asigna el estado inicial: `typeNews` = **`new_ord`** (typeId: 1).
4.  **Comunicación con el POS (Ida):**
    *   La aplicación ejecuta la función `pushNewToQueue()`.
    *   La `news` recién creada se envía a la cola SQS del Punto de Venta (POS).

**Resultado Etapa 1:** La orden existe en la base de datos con un `trace` inicial y ha sido enviada al local para su gestión.

## **Etapa 2: El Bucle de Comunicación (Plataforma ↔ POS)**

Este es el corazón del proceso. Es un ciclo constante de mensajes entre la aplicación central y el POS.

### **A. El POS Actúa y Responde**

1.  **Acción en el Local:** El personal del POS visualiza la orden y realiza una acción (confirmar, rechazar, etc.).
2.  **Respuesta del POS:** El POS modifica el `typeId` de la `news` para reflejar la acción y la envía de vuelta a una cola SQS central.

### **B. La Plataforma Procesa la Novedad**

1.  **Punto de Entrada del Bucle:** La función `pollFromQueue()` recoge el mensaje del POS.
2.  **El Gran Distribuidor (`SetNews`):** La `news` se pasa al dispatcher `set-news.js`, que utiliza un `switch` para leer el `typeId` y elegir la Estrategia correcta.

### **C. La Estrategia Entra en Juego**

Cada `typeNews` activa una estrategia diferente que define el siguiente paso.

| `typeNews` (ID) enviado por POS | Estrategia Ejecutada | Acción y Resultado del Ciclo |
| :--- | :--- | :--- |
| `receive_ord` (10) | `ReceiveStrategy` | Actualiza estado a "Recibido". El ciclo continúa. |
| `view_ord` (11) | `ViewStrategy` | Actualiza estado a "Visto". El ciclo continúa. |
| `confirm_ord` (5) | `ConfirmStrategy` | Actualiza estado a "Confirmado". Puede notificar a la plataforma. El ciclo continúa. |
| `ready_ord` (15) | `ReadyStrategy` | Actualiza estado a "Listo para Despacho". El ciclo continúa. |
| `disp_ord` (3) | `DispatchStrategy` | Actualiza estado a "Despachado". FIN DEL CICLO. |
| `deliv_ord` (14) | `DeliveryStrategy` | Actualiza estado a "Entregado". FIN DEL CICLO. |
| `rej_ord` (2) | `BranchRejectStrategy` | Actualiza estado a "Rechazado por Local". Notifica a la plataforma. FIN DEL CICLO. |
| `platform_rej_ord` (4) | `PlatformRejectStrategy` | Actualiza estado a "Rechazado por Plataforma". FIN DEL CICLO. |

### **D. La Rueda Sigue Girando**

1.  **Actualización en BD:** La estrategia actualiza la `news` en MongoDB, añadiendo un nuevo `trace` que documenta el cambio.
2.  **Comunicación con el POS (Vuelta):** Si el ciclo no ha terminado, la aplicación vuelve a usar `pushNewToQueue()` para enviar la `news` actualizada de vuelta al POS.

## **Etapa 3: El Fin del Camino - Estados Terminales**

El ciclo se detiene cuando la `news` alcanza un estado que no requiere más interacción del POS.

*   **Rechazo (`rej_ord` o `platform_rej_ord`):** La orden se cancela. La estrategia notifica a la plataforma externa si es necesario y actualiza el estado final. No se envía nada más al POS.
*   **Finalización Exitosa (`disp_ord` o `deliv_ord`):** La orden se considera completada. La estrategia actualiza el estado final y el proceso termina.

## **Diagrama de Flujo Visual**

**1. API POST: Third Party**
(ej. Pedigrido)
↓
**2. Controlador API**
Transforma a `news` (typeId: 1)
↓
**3. Base de Datos (MongoDB)**
Guarda la `news` inicial
↓
**4. SQS (Cola del POS)**
`pushNewToQueue()` envía la orden al local
↓
**5. Punto de Venta (POS)**
Personal actúa y cambia el `typeId`
↓
**6. SQS (Cola Central)**
POS envía la `news` actualizada
↓
**7. Plataforma de Servicios**
`pollFromQueue()` → `SetNews` → **Estrategia**

*   **SI ES ESTADO TERMINAL**
    ↓
    **FIN DEL CICLO**

*   **SI NO ES ESTADO TERMINAL**
    ↓
    **8. Base de Datos (MongoDB)**
    Actualiza `news` con nuevo `trace`
    ↑
    **Vuelve al paso 4 (Bucle)**
