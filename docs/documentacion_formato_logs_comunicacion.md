---
layout: default
title: "Documentación de Logs de Comunicacion con Plataformas"
subtitle: "Estructura y acciones para la observabilidad de Logs de comunicacion con Plataformas"
---

Este documento detalla la nueva estructura de logs, diseñada para mejorar la observabilidad en las comunicaciones con Plataformas. Basandose en las llamadas a la API de cada plataforma.

## **1. Estructura Genérica del Log**

Los logs siguen un formato priorizando la información estructurada para permitir búsquedas en sistemas como MongoDB o Cloudwatch.

### **Atributos del Objeto de Log**

| Atributo | Tipo | Descripción |
| :--- | :--- | :--- |
| **message** | `String` | Un encabezado humano-legible que sigue el patrón: `[Prefix]: [Platform/Service] [Action] [OrderId]`. Los prefijos comunes son `Platform Communication Error` o `Sml Communication Error`. |
| **action** | `String` | El nombre técnico de la operación que se estaba intentando realizar (ej: `receiveOrder`, `confirmOrder`). |
| **platformId** | `Number` | El identificador único de la plataforma en el sistema (para PediGrido es `7`). |
| **order** | `Object` | Contiene información contextual de la orden afectada. |
| **order.id** | `Number` | El ID interno de la orden en nuestra base de datos. |
| **order.originalId** | `String` | El ID original de la orden proporcionado por la plataforma. |
| **order.statusId** | `Number` | El estado actual de la orden en el momento del fallo. |
| **order.branchId** | `Number` | El ID de la sucursal a la que pertenece la orden. |
| **order.country** | `String` | El país de la sucursal (ej: "Argentina", "Uruguay"). |
| **request** | `Object` | Detalles de la petición HTTP que falló. |
| **request.url** | `String` | La URL completa del endpoint al que se realizó la petición. |
| **request.body** | `Object` | El cuerpo (payload) enviado a la plataforma. |
| **request.header** | `Object` | Los encabezados (headers) de configuración utilizados. |

---

## **2. Descripción de Acciones (Actions)**

Las acciones representan los diferentes puntos de integración en el ciclo de vida de un pedido.

### **Acciones de Plataforma (PediGrido)**

| Acción | Descripción |
| :--- | :--- |
| **receiveOrder** | Se dispara cuando el sistema intenta notificar a la plataforma que el pedido ha sido ingresado correctamente en el sistema local. |
| **confirmOrder** | Se ejecuta cuando el pedido es aceptado por la sucursal (ya sea manual o automáticamente). |
| **readyOrder** | Notifica a la plataforma que el pedido está preparado y listo para ser entregado o retirado por el repartidor. |
| **dispatchOrder** | Indica que el pedido ha salido de la sucursal y está en camino (despachado). |
| **deliveryOrder** | Confirma que el pedido ha sido entregado exitosamente al cliente final. |
| **branchRejectOrder** | Se utiliza cuando la sucursal rechaza el pedido por algún motivo específico (falta de stock, zona no cubierta, etc.). |

### **Acciones de Fidelización (SML)**

Para pedidos de PediGrido que involucran beneficios del programa de lealtad (canje de puntos), el archivo `rapiboy.js` también gestiona comunicaciones con el servicio **SML**. Estos logs se distinguen por el prefijo `Sml Communication Error`.

| Acción | Descripción |
| :--- | :--- |
| **receiveOrder** | Notifica a SML el ingreso de un pedido con canje para reservar o procesar los puntos correspondientes. |
| **confirmOrder** | Informa a SML que el pedido ha sido confirmado, permitiendo la ejecución definitiva del canje. |
| **branchRejectOrder** | Notifica a SML que el pedido fue rechazado, lo que desencadena la devolución o anulación de los puntos utilizados. |

---

## **3. UberEats (`uberEats.js`)**

La integración con **UberEats** (Platform ID: `4`) utiliza el esquema estandarizado para sus operaciones principales de cambio de estado.

### **Acciones de Plataforma (UberEats)**

| Acción | Descripción |
| :--- | :--- |
| **receiveOrder** | Se dispara durante la aceptación automática (si la sucursal o cadena tiene `autoAccept` activo). Envía el tiempo estimado de preparación (`ready_for_pickup_time`) a Uber. |
| **confirmOrder** | Se ejecuta en la aceptación manual del pedido. Al igual que `receiveOrder`, notifica a Uber el tiempo de preparación configurado. |
| **readyOrder** | Informa a Uber que el pedido está listo para ser retirado. No envía cuerpo (`body: null`) en la petición. |
| **branchRejectOrder** | Se utiliza para cancelar el pedido en la plataforma de Uber. Envía un motivo de rechazo (`type`) y un mensaje informativo (`info`). |

---

## **4. PedidosYa (`pedidosYa.js`)**

La integración con **PedidosYa** (Platform ID: `1`) es una de las más completas y cuenta con logs detallados para casi todo el ciclo de vida del pedido, incluyendo la fase de autenticación.

### **Acciones de Plataforma (PedidosYa)**

| Acción | Descripción |
| :--- | :--- |
| **getAccessToken** | Se dispara durante el proceso de login o renovación de token. Es una acción de infraestructura, por lo que el objeto `order` se reporta como `null`. |
| **receiveOrder** | Se ejecuta en la **aceptación automática**. Notifica a PedidosYa la recepción del pedido y envía el `acceptanceTime` calculado según las reglas de la sucursal. |
| **confirmOrder** | Se ejecuta en la **aceptación manual**. Al igual que `receiveOrder`, notifica la aceptación y el tiempo estimado de preparación. |
| **readyOrder** | Informa a la plataforma que el pedido ha sido preparado (`orderPreparedUrl`). Se envía con un cuerpo de petición vacío (`body: null`). |
| **dispatchOrder** | Se dispara cuando el pedido es marcado como "Despachado" (especialmente en pedidos de tipo `pickup` o sin asignación de rider). |
| **branchRejectOrder** | Notifica el rechazo del pedido desde la sucursal. Envía el `message` y el `reason` del rechazo (ej: `TOO_BUSY`). |

<div class="note">
  <p><strong>Nota Técnica:</strong> En la versión actual del código, el log de la acción <code>dispatchOrder</code> contiene un mensaje inconsistente (<code>Platform Communication Error: UberEats receiveOrder</code>), aunque los metadatos de la acción y el ID de plataforma (<code>1</code>) son correctos para PedidosYa.</p>
</div>

---

## **5. Rappi (`rappi.js`)**

La integración con **Rappi** (Platform ID: `2`) utiliza un sistema de **polling** para obtener pedidos y endpoints específicos para la gestión del flujo de trabajo.

### **Acciones de Plataforma (Rappi)**

| Acción | Descripción |
| :--- | :--- |
| **getOrders** | Se dispara periódicamente (polling) para consultar nuevos pedidos en la plataforma. Al ser una consulta general de sucursal, el objeto `order` se reporta como `null`. |
| **receiveOrder** | Se ejecuta en la **aceptación automática**. Realiza una petición `PUT` al endpoint `/take/{time}` enviando el tiempo de preparación calculado en minutos. |
| **confirmOrder** | Se ejecuta en la **aceptación manual**. Al igual que `receiveOrder`, notifica la toma del pedido con el tiempo de preparación correspondiente. |
| **branchRejectOrder** | Envía el rechazo del pedido a Rappi. A diferencia de otras acciones, esta envía un cuerpo con el motivo detallado (`reason: rejectDesc`). |

<div class="note">
  <p><strong>Nota de Implementación:</strong> Rappi utiliza una autenticación basada en Auth0, y las URLs de los endpoints cambian dinámicamente según el país configurado para la sucursal.</p>
</div>

---

## **6. ThirdParty / I+D (`thirdParty.js`)**

La integración con **ThirdParty** (Platform ID: `12`), usualmente referenciada como **I+D** en los logs, es una integración genérica utilizada para plataformas de terceros. Al igual que PediGrido, esta integración también gestiona de forma nativa la comunicación con el servicio de fidelización **SML**.

### **Acciones de Plataforma (I+D)**

| Acción | Descripción |
| :--- | :--- |
| **receiveOrder** | Se dispara en la **aceptación automática**. Realiza un `POST` al endpoint `/ConfirmarPedido` (aunque el nombre de la acción sea `receiveOrder`) enviando el tiempo de demora. |
| **viewOrder** | Notifica a la plataforma que el pedido ha sido visualizado (`/VistarPedido`). |
| **confirmOrder** | Se ejecuta en la **aceptación manual**. Al igual que `receiveOrder`, consume el endpoint `/ConfirmarPedido`. |
| **dispatchOrder** | Indica que el pedido ha sido enviado (`/EnviarPedido`). |
| **deliveryOrder** | Informa que el pedido ha sido entregado exitosamente (`/EntregarPedido`). |
| **branchRejectOrder** | Se utiliza para cancelar el pedido (`/CancelarPedido`). Envía el ID del motivo de rechazo y una nota opcional. |

### **Acciones de Fidelización (SML via I+D)**

Los logs de SML disparados desde esta clase utilizan el prefijo `Sml Communication Error`.

| Acción | Descripción |
| :--- | :--- |
| **SMLLogin** | Se dispara al inicializar la plataforma para obtener el token de acceso al servicio SML. |
| **receiveOrder** | Notifica a SML sobre un nuevo pedido con canje de puntos generado desde la integración I+D. |
| **confirmOrder** | Confirma el procesamiento de puntos en SML tras la aceptación del pedido. |
| **branchRejectOrder** | Solicita la anulación del canje de puntos en SML debido al rechazo del pedido. |

---

## **7. MercadoPago (`mercadoPago.js`)**

La integración con **MercadoPago** (Platform ID: `8`) implementa el esquema estandarizado para sus operaciones de aceptación y cancelación. Una característica particular de esta plataforma es que el proceso de comunicación requiere la obtención previa de un token de acceso desencriptado desde el servicio de Cloud del sistema.

### **Acciones de Plataforma (MercadoPago)**

| Acción | Descripción |
| :--- | :--- |
| **receiveOrder** | Se dispara en la **aceptación automática**. Antes de enviar el estado `accepted` a MercadoPago, el sistema realiza el flujo de obtención y desencriptación del token de usuario desde la API Cloud. |
| **confirmOrder** | Se ejecuta en la **aceptación manual**. Al igual que en `receiveOrder`, notifica el estado `accepted` a través del endpoint de proximity-integration. |
| **branchRejectOrder** | Se utiliza para cancelar el pedido (`/cancel`) enviando el estado `cancelled`. Este flujo también requiere la validación previa del Merchant ID y el token de Cloud. |

<div class="note">
  <p><strong>Nota de Implementación:</strong> A diferencia de otras plataformas, MercadoPago no requiere el envío de tiempos de preparación (<code>ready_for_pickup_time</code>) en sus peticiones de aceptación. El foco principal de los logs en esta integración es la trazabilidad del proceso de obtención de tokens.</p>
</div>
