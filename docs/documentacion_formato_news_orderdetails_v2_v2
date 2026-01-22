---
layout: default
title: "Especificación: Formato JSON Estructurado para Pedidos v5.0"
subtitle: "Documento: Nuevo Formato de News en order.details"
---

Versión: 5.0 (Final)
Fecha: 12 de Septiembre, 2025

## **1. Objetivo General**

Este documento define el formato JSON de salida para el objeto `details` en la coleccion `news` de pedidos procesados para nuevos tenants.

## **2. Estructura Raíz**

El objeto JSON de un pedido tiene una única clave de nivel superior: `details`. El valor de esta clave es siempre un **Array** que contendrá objetos `item`.

```json
{
  "order":{
      //... Demas Atributos...
      "details": [
       // ... Array de objetos Ítem ...
      ]
}
```

## **3. El Objeto `Item`**

El "Ítem" es el objeto principal que describe cada línea del pedido. Su comportamiento y estructura están determinados por el atributo `itemType`. A continuación se detallan todos los atributos posibles que un Ítem puede tener.

| Atributo | Tipo | Descripción |
| :--- | :--- | :--- |
| `itemType` | String | Define el rol del `item`. Valores: `PRODUCT`, `COMBO`, `PROMOTION`, `COMPONENT`. |
| `itemId` | String | Identificador único del `item` en la plataforma de origen. |
| `sku` | String | El Identificador Único de `item` (SKU) interno. |
| `itemDescription` | String | El nombre descriptivo y legible del `item`. |
| `price` | Number | Precio unitario del `item`. |
| `discount` | Number | Monto de descuento aplicado a este `item`. Por defecto es 0. |
| `quantity` | Number | La cantidad de unidades de este `item`. |
| `customizations` | Object \| null | Encapsula modificaciones. Para `PRODUCT`, puede contener `removedComponents`, `extras` y `observation`. Para `COMBO` y `PROMOTION`, **solo puede contener `observation`**. Siempre es `null` para `COMPONENT`. |
| `includedItems` | Array \| null | Un array de `items` que están contenidos dentro de un `COMBO` o `PROMOTION`. Siempre es `null` para otros tipos. |

## **4. Análisis Detallado de los `itemType`**

### **itemType: 'PRODUCT'**

**Propósito:** Representa un artículo vendible, ya sea de forma independiente o como parte de un combo.

**Comportamiento:** Es el único tipo de ítem que puede tener el atributo `customizations` completamente poblado. Su atributo `includedItems` siempre es `null`.

#### **Ejemplo A: Producto simple**

```json
{
  "itemType": "PRODUCT",
  "itemId": "176321948",
  "sku": "176321948",
  "itemDescription": "Gaseosa Pepsi 354 ml",
  "price": 4000,
  "discount": 0,
  "quantity": 2,
  "customizations": null,
  "includedItems": null
}
```

#### **Ejemplo B: Producto con personalizaciones**

```json
{
  "itemType": "PRODUCT",
  "itemId": "143787167",
  "sku": "143787167",
  "itemDescription": "Hamburguesa Smack",
  "price": 18500,
  "discount": 0,
  "quantity": 1,
  "customizations": { /* ... ver sección 5 ... */ },
  "includedItems": null
}
```

### **itemType: 'COMBO' y 'PROMOTION'**

**Propósito:** Representan un **contenedor** o un agrupador comercial. Su función es empaquetar varios `PRODUCT`s bajo un precio único.

**Comportamiento:** Su atributo `includedItems` siempre es un array que contiene los `PRODUCT`s que lo componen. Su atributo `customizations` **puede existir, pero únicamente para contener un `observation`**. Si existe, no puede contener `removedComponents` ni `extras`. Si no hay observación, será `null`.

#### **Ejemplo: Un COMBO completo con observación**

```json
{
  // --- El Contenedor ---
  "itemType": "COMBO",
  "itemId": "143787155",
  "sku": "143787155",
  "price": 18000,
  "discount": 0,
  "itemDescription": "Combo 1 - Gulosa",
  "quantity": 1,
  "customizations": {
    "observation": "Preparar este combo para llevar."
  },
  
  // --- Los Productos Incluidos ---
  "includedItems": [
    {
      "itemType": "PRODUCT",
      "itemId": "143787155",
      "sku": "143787155",
      "itemDescription": "Gulosa",
      "price": 0, 
      "discount": 0,
      "quantity": 1,
      "customizations": { /* ... puede tener sus propias personalizaciones ... */ }
    },
    {
      "itemType": "PRODUCT",
      "itemId": "185798491",
      "sku": "185798491",
      "itemDescription": "Gaseosa 7Up Lima 354 ml",
      "price": 0,
      "discount": 0,
      "quantity": 1,
      "customizations": null
    }
  ]
}
```

## **5. El Objeto `customizations` y el `itemType: 'COMPONENT'`**

Este objeto es fundamental para describir cómo se modifica un `PRODUCT`.

*   **Propósito del `customizations`:** Centralizar todas las modificaciones de un producto: ingredientes quitados, extras añadidos y notas de texto libre.
*   **Propósito del `COMPONENT`:** Describir un modificador con las propiedades de un artículo (sku, price, etc.), actuando como un **punto final** en la estructura. Un `COMPONENT` no puede tener `customizations` ni `includedItems`.

#### **Estructura del objeto `customizations` (para `itemType: 'PRODUCT'`):**

| Atributo | Tipo | Descripción |
| :--- | :--- | :--- |
| `removedComponents` | Array | Array de objetos con `itemType: 'COMPONENT'` que fueron quitados. |
| `extras` | Array | Array de objetos con `itemType: 'COMPONENT'` que fueron añadidos (pueden tener costo). |
| `observation` | String | Un campo de texto libre con notas para la cocina (ej. "Bien cocida"). |

#### **Ejemplo completo de un objeto `customizations`:**

```json
{
  "observation": "CON POCA SAL",
  "removedComponents": [
    {
      "itemType": "COMPONENT",
      "itemId": "145742786",
      "sku": "145742786",
      "itemDescription": "Sin queso cheddar",
      "price": 0,
      "discount": 0,
      "quantity": 1
    }
  ],
  "extras": [
    {
      "itemType": "COMPONENT",
      "itemId": "185804975",
      "sku": "185804975",
      "itemDescription": "Acompañamiento con cheddar",
      "price": 2500,
      "discount": 0,
      "quantity": 1
    }
  ]
}
```

## **6. Pedido con Json de Ejemplo**

### **Ejemplo Visual del Pedido**

La siguiente imagen muestra el ticket original del cual se deriva el ejemplo JSON. Sirve como referencia visual para entender el desglose de los productos y combos.

[Ver Ejemplo Visual en Imgur](//imgur.com/a/WCzleQ2)

### **Descripción del Pedido**

El pedido original se traduce a la estructura JSON de abajo siguiendo estas reglas:

*   **Un "Combo 1 - Crispy":** Se mapea como un ítem de tipo `COMBO`. Contiene un producto principal ("Crispy") con un ingrediente removido ("Sin alioli") y un extra ("Extra palta"), además de sus productos incluidos.
*   **Un "Combo 1 - Gulosa":** Otro ítem de tipo `COMBO` que agrupa su hamburguesa, bebida, acompañamiento y un postre adicional como `includedItems` y una nota específica en `customizations.observation`.
*   **Una "Hamburguesa Gulosa" con acompañamiento:** Este caso es especial. Al no ser un "Combo", se desglosa en **dos ítems de primer nivel**:
    1.  La "Hamburguesa Gulosa" como un `PRODUCT` con su precio completo y una nota específica en `customizations.observation`.
    2.  Las "Papas fritas" como un segundo `PRODUCT` con precio 0, ya que su costo está incluido en el de la hamburguesa.
*   **Dos "Gaseosa Pepsi 354 ml":** Se mapea como un único `PRODUCT` simple con `quantity: 2`.

### **Formato JSON Resultante**

```json
{
  "details": [
    {
      "itemType": "COMBO",
      "itemId": "COMBO_CRISPY_1",
      "sku": "COMBO_CRISPY_1",
      "itemDescription": "Combo 1 - Crispy",
      "price": 19900,
      "discount": 0,
      "quantity": 1,
      "customizations": null,
      "includedItems": [
        {
          "itemType": "PRODUCT",
          "itemId": "BURGER_CRISPY",
          "sku": "BURGER_CRISPY",
          "itemDescription": "Crispy",
          "price": 0,
          "discount": 0,
          "quantity": 1,
          "customizations": {
            "observation": "",
            "removedComponents": [
              {
                "itemType": "COMPONENT",
                "itemId": "MOD_SIN_ALIOLI",
                "sku": "MOD_SIN_ALIOLI",
                "itemDescription": "Sin alioli",
                "price": 0,
                "discount": 0,
                "quantity": 1
              }
            ],
            "extras": [
              {
                "itemType": "COMPONENT",
                "itemId": "EXTRA_PALTA",
                "sku": "EXTRA_PALTA",
                "itemDescription": "Extra palta",
                "price": 2000,
                "discount": 0,
                "quantity": 1
              }
            ]
          }
        },
        {
          "itemType": "PRODUCT",
          "itemId": "ACOMP_PAPAS",
          "sku": "ACOMP_PAPAS",
          "itemDescription": "Papas fritas",
          "price": 0,
          "discount": 0,
          "quantity": 1,
          "customizations": null,
          "includedItems": null
        },
        {
          "itemType": "PRODUCT",
          "itemId": "BEB_PEPSI_BLACK",
          "sku": "BEB_PEPSI_BLACK",
          "itemDescription": "Gaseosa Pepsi Black 354 ml",
          "price": 0,
          "discount": 0,
          "quantity": 1,
          "customizations": null,
          "includedItems": null
        }
      ]
    },
    {
      "itemType": "COMBO",
      "itemId": "COMBO_GULOSA_1",
      "sku": "COMBO_GULOSA_1",
      "itemDescription": "Combo 1 - Gulosa",
      "price": 19900,
      "discount": 0,
      "quantity": 1,
      "customizations": {
        "observation": "Sin sal porfavor"
      },
      "includedItems": [
        {
          "itemType": "PRODUCT",
          "itemId": "BURGER_GULOSA",
          "sku": "BURGER_GULOSA",
          "itemDescription": "Gulosa",
          "price": 0,
          "discount": 0,
          "quantity": 1,
          "customizations": null
        },
        {
          "itemType": "PRODUCT",
          "itemId": "BEB_POMELO",
          "sku": "BEB_POMELO",
          "itemDescription": "Gaseosa Paso De Los Toros Pomelo",
          "price": 0,
          "discount": 0,
          "quantity": 1,
          "customizations": null,
          "includedItems": null
        },
        {
          "itemType": "PRODUCT",
          "itemId": "ACOMP_PAPAS",
          "sku": "ACOMP_PAPAS",
          "itemDescription": "Papas fritas",
          "price": 0,
          "discount": 0,
          "quantity": 1,
          "customizations": null,
          "includedItems": null
        },
        {
          "itemType": "PRODUCT",
          "itemId": "POSTRE_FRAMBUESA",
          "sku": "POSTRE_FRAMBUESA",
          "itemDescription": "Frambuesa con chocolate semiamargo",
          "price": 8000,
          "discount": 0,
          "quantity": 1,
          "customizations": null,
          "includedItems": null
        }
      ]
    },
    {
      "itemType": "PRODUCT",
      "itemId": "BURGER_GULOSA_SOLA",
      "sku": "BURGER_GULOSA_SOLA",
      "itemDescription": "Hamburguesa Gulosa",
      "price": 18900,
      "discount": 0,
      "quantity": 1,
      "customizations": {
        "observation": "Sin sal porfavor",
        "removedComponents": [],
        "extras": []
      },
      "includedItems": null
    },
    {
      "itemType": "PRODUCT",
      "itemId": "ACOMP_PAPAS",
      "sku": "ACOMP_PAPAS",
      "itemDescription": "Papas fritas",
      "price": 0,
      "discount": 0,
      "quantity": 1,
      "customizations": null,
      "includedItems": null
    },
    {
      "itemType": "PRODUCT",
      "itemId": "BEB_PEPSI_354",
      "sku": "BEB_PEPSI_354",
      "itemDescription": "Gaseosa Pepsi 354 ml",
      "price": 8000,
      "discount": 0,
      "quantity": 2,
      "customizations": null,
      "includedItems": null
    }
  ]
}
```
