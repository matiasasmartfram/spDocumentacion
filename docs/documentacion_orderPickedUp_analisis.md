---
layout: default
title: Análisis de Variable orderPickedUp
subtitle: Guia técnica sobre el propósito, ubicación y lógica de bloqueo de la variable orderPickedUp en el ecosistema SmartPedidos.
---

## Introducción

Este documento detalla el funcionamiento de la variable **orderPickedUp**, una implementación diseñada para sincronizar el estado físico de la orden con las capacidades de gestión del punto de venta (POS).

## 1. Propósito de la Variable

La variable **orderPickedUp** tiene como objetivo principal **evitar que una sucursal pueda rechazar o cancelar una orden que ya ha sido retirada físicamente** por un repartidor externo (logística de plataforma).

Esta validación previene conflictos operativos críticos:
*   **Inconsistencia de Estado**: Evita que el sistema del POS intente anular un pedido que ya está en tránsito.
*   **Conflictos con el Repartidor**: Previene situaciones donde el repartidor ya retiró los productos pero el sistema marca la orden como cancelada.
*   **Integridad de Datos**: Asegura que el flujo de la orden en SmartPedidos refleje la realidad física reportada por la plataforma original.

## 2. Ubicación en la Base de Datos

La variable se almacena en la colección **news** de MongoDB. Es importante destacar que se ubica directamente en la **raíz del documento**, garantizando un acceso rápido y desacoplado del objeto interno de la orden.

*   **Colección**: `news`
*   **Nivel**: Raíz (root level)
*   **Alcance**: Global (disponible para todas las plataformas).

### Ejemplo de Estructura JSON

A continuación se muestra el esquema de una "novedad" donde se resalta la ubicación de la variable:

```json
{
  "_id": { "$oid": "698cec3ec3f1d235da83c572" },
  "order": {
    "id": "1770843198714",
    "platformId": 1,
    "statusId": 5,
    "totalAmount": 4500
    // ... otros campos de la orden
  },
  "branchId": 50,
  "extraData": {
    "platform": "PedidosYa",
    "country": "Uruguay"
  },
  
  "orderPickedUp": false,  // <--- UBICACIÓN: Raíz del documento news
  
  "traces": [
    { "entity": "PLATFORM", "update": { "typeId": 1 } },
    { "entity": "BRANCH", "update": { "typeId": 10 } }
  ],
  "typeId": 14,
  "updatedAt": { "$date": "2026-02-12T19:45:26.172Z" }
}
```

## 3. Funcionamiento y Lógica de Bloqueo

Actualmente, el flujo de actualización está optimizado para **PedidosYa**, aunque la arquitectura permite la expansión a otras plataformas.

### Flujo de Actualización (PedidosYa)
1.  **Detección**: El sistema recibe un webhook de PedidosYa (`updateOrder`) con el estado unificado `"order_picked_up"`.
2.  **Persistencia**: El controlador correspondiente (`peya.js`) identifica este evento y realiza un `findOneAndUpdate`, estableciendo **orderPickedUp: true**.

### Validación de Bloqueo (SmartCloud/POS)
Cuando la sucursal intenta realizar una acción de **Rechazo** (vía `BranchRejectStrategy`), el sistema ejecuta la siguiente lógica:

1.  **Consulta**: Verifica el valor de **orderPickedUp** en la noticia asociada.
2.  **Condicional**: Si **orderPickedUp === true**:
    *   Se genera un log de advertencia técnica.
    *   Se **aborta el proceso de rechazo** inmediatamente.
    *   No se envía ninguna señal de cancelación a la plataforma de origen.

## 4. Escalabilidad e Infraestructura Genérica

La implementación se ha realizado siguiendo principios de diseño escalable:

*   **Modelo Unificado**: El campo **orderPickedUp** se inicializa para todas las noticias, independientemente de la plataforma de origen.
*   **Protección Transversal**: La lógica en `BranchRejectStrategy` es agnóstica a la plataforma; si el flag está encendido, el rechazo se bloquea siempre.
*   **Futuras Integraciones**: Para activar este comportamiento en **UberEats**, **Rappi** o **PediGrido**, solo se requiere implementar el mapeo del evento de retiro en sus respectivos controladores para que actualicen el campo en la raíz de la noticia.

---

