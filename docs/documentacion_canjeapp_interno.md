---
layout: default
title: "Especificación Técnica: Formato de Salida para Pedidos con Canje"
subtitle: "Documento para Consumidores de la API de SmartPedidos"
---

## **1. Objetivo de este Documento**

Este documento ejemplifica y describe el formato de datos final que los sistemas consumidores recibirán de SmartPedidos. Se detalla la estructura para **CanjeApp**.

## **2. Ejemplo Completo de Pedido con Canje**

A continuación, un pedido completo con un canje. **El formato dentro de `order.details[]` es Similar a una PROMOCION pero con el agregado del atributo `canje: 1`**

### **2.1. Pedido Recibido por SmartPedidos (Entrada de Referencia)**

```json
{
  "order": {
    "user": {
      "dni": "39690194",
      "tipoIdentificacion": "DNI",
      "numeroTarjetaLoyalty": "1234567890",
      "contieneCanje": true,
      "puntosCanjeados": 3000
    },
    "details": [
      {
        "name": "50% Descuento en Kilo por 3000 Puntos",
        "sku": "1641",
        "canje": 1,                          // Cabecera del item con canje
        "promotion": false,
        "optionGroups": [
          { 
          "name": "Kilo de Helado", 
          "sku": "64",                       //Sku del Item del Canje
          "notes": "Chocolate, Vainilla" }
        ]
      }
    ]
  }
}
```

### **2.2. Pedido Entregado por SmartPedidos (Salida Transformada News)**

```json
{
  "order": {
    "customer": {
      "dni": "39690194",
      "tipoIdentificacion": "DNI",                   //Nuevo Atributo
      "numeroTarjetaLoyalty": "1234567890",          //Nuevo Atributo      
      "contieneCanje": true,                         //Nuevo Atributo
      "puntosCanjeados": 3000                        //Nuevo Atributo
    },
    "details": [
      {
        "description": "50% Descuento en Kilo por 3000 Puntos",
        "sku": "1641",
        "price": 5000,
        "promo": 2,      // '2' identifica al item como Contenedor (Padre) de un grupo.
        "groupId": 1,    // '1' es el ID de este grupo de transacción.
        "canje": 1       // '1' en el Padre significa: "Todo este grupo (groupId=1) es un CANJE". - Nuevo Atributo
      },
      {
        "description": "Kilo de Helado",
        "sku": "64",
        "price": 0,
        "optionalText": "Chocolate, Vainilla",
        "promo": 1,      // '1' identifica al item como Componente (Hijo).
        "groupId": 1,    // Pertenece al mismo grupo que su Padre.
        "canje": 1       // El hijo lleva el flag de canje; se infiere de su Padre. - Nuevo Atributo
      }
    ]
  }
}
```
