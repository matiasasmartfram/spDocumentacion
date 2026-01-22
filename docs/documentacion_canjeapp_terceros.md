---
layout: default
title: Especificación Técnica – Formato de Canje de Puntos
subtitle: Para PediGrido e I+Diot - SmartPedidos - CanjeApp
---

## **1. Objetivo**

Integrar el sistema de canje de puntos de ClubGrido con la plataforma SmartPedidos. El objetivo es garantizar que los pedidos con canje, originados en PediGrido e I+Diot, contengan los atributos necesarios para ser procesados correctamente y que esta información se transmita de forma transparente hacia SmartLoyalty.

---

## **2. Alcances y Responsabilidades**

*   **SmartPedidos** funcionará como un **puente de transmisión** de información, traspasando los datos del pedido a los sistemas correspondientes.
*   **SmartLoyalty** se encargará de toda la **lógica de negocio** del canje (validación de saldo, aplicación de beneficios, etc.).
*   Es responsabilidad de **PediGrido e I+Diot** construir y enviar la estructura del pedido (`order`) conforme a esta especificación.

---

## **3. Especificación del Payload `order`**

El payload se compone de la información del cliente (`user`) y el detalle de los ítems (`details`).

### **3.1. Atributos de Cliente (`order.user`)**

| Campo | Tipo | Obligatorio | Descripción |
| :--- | :--- | :--- | :--- |
| `dni` | String | Si `contieneCanje` es `true` | Documento/RUC del cliente. |
| `tipoIdentificacion` | String | Si `contieneCanje` es `true` | Tipo de documento (ej. "DNI", "RUC"). |
| `numeroTarjetaLoyalty` | String | Si `contieneCanje` es `true` | Número de tarjeta del programa de lealtad. |
| `contieneCanje` | Boolean | Sí | Debe ser `true` si el pedido incluye un canje. |
| `puntosCanjeados` | Number | Si `contieneCanje` es `true` | Cantidad total de puntos utilizados. |

### **3.2. Estructura de Ítems (`order.details`)**

Cuando un ítem es un canje, debe seguir una estructura anidada que separa el canje lógico de los productos físicos que descuentan stock.

```json
"details": [
  {
    "id": 0,
    "unitPrice": 5000,
    "quantity": 1,
    "name": "Canje 1 Kilo - 5000 puntos + 50%",
    "sku": "12931",            //SKU del Canje 
    "notes": null,
    "canje": 1,               // Flag Numerico para indicar que ES un canje
    "promotion": false,       // Flag booleano para indicar que NO es una promoción
    "optionGroups": [
      {
        "id": 91,
        "name": "Kilo de Helado",
        "sku": "64",              
        "unitPrice": 0,
        "notes": "Chocolate, Limón, Frutilla, Vainilla"
      }
    ]
  }
]
```

#### Descripción de Campos `details` (Nivel Superior del Ítem)

| Campo | Tipo | Obligatorio | Descripción |
| :--- | :--- | :--- | :--- |
| `id` | Number | Sí | Identificador secuencial interno de la línea. |
| `unitPrice` | Number | Sí | Precio final por unidad del Canje. |
| `quantity` | Number | Sí | Cantidad de ítems. Para canjes, **el valor siempre debe ser 1**. |
| `name` | String | Sí | Descripción comercial. Ej: "Canje 1 Kilo - 5000 puntos + 50%". |
| `sku` | String | Sí | SKU **DEL CANJE** (no descuenta stock). |
| `notes` | String | No | Observaciones generales. |
| `canje` | Numeric | Sí | Flag Numerico **Si es `1` indica que es un canje**; `0` que no lo es. |
| `promotion` | Boolean | Sí | Flag booleano. **Debe ser `false` si el item contiene `canje`.** |
| `optionGroups` | Array | Sí | Lista de productos reales que componen el ítem y descuentan stock. Requerido para canjes. |

#### Estructura de `OptionGroup` (Productos Físicos)

| Campo | Tipo | Obligatorio | Descripción |
| :--- | :--- | :--- | :--- |
| `id` | Number | Sí | ID del grupo de opciones. |
| `name` | String | Sí | Nombre descriptivo del producto físico. Ej: "Kilo de Helado". |
| `sku` | String | Sí | SKU **real** del producto que descuenta stock. Ej: "64". |
| `unitPrice` | Number | Sí | Debe ser `0` para ítems dentro de un canje. |
| `notes` | String | Sí | Detalle específico del producto (sabores, variantes, etc.). |

---

## **4. Reglas de Negocio Clave**

1.  **Identificación de Canje:** Un pedido contiene un canje si `order.user.contieneCanje` es `true` Y existe al menos un ítem en `details` con `canje: 1`.
2.  **Exclusión Mutua Canje/Promoción:** Si un ítem es un canje (`canje: 1`), su campo `promotion` **obligatoriamente debe ser `false`**.
3.  **SKU Lógico vs. SKU Real:** El SKU en la cabecera (`details.sku`) es un identificador lógico del canje y **no descuenta stock**. El stock se descuenta únicamente de los SKU especificados dentro de `optionGroups`.
4.  **Cantidad Fija en 1 para Canjes:** El campo `quantity` en un ítem de canje siempre debe ser `1`. Si un cliente adquiere dos canjes, se deben enviar dos objetos separados en el array `details`.
5.  **Precio Cero en `optionGroups`:** El campo `unitPrice` dentro de cada objeto de `optionGroups` de un canje debe ser `0`.
6.  **Notas Específicas:** Los detalles de los productos, como los sabores, deben ir en el campo `notes` del `optionGroup` correspondiente.

---

## **5. Ejemplo Completo de Pedido con Canje**

A continuación, un ejemplo de un pedido completo que incluye un canje de un kilo de helado y la compra de otro producto regular.

```json
{
  "order": {
    "user": {
      "dni": "39690194",
      "tipoIdentificacion": "DNI",
      "numeroTarjetaLoyalty": "1234567890",
      "contieneCanje": true,
      "puntosCanjeados": 5000
    },
    "details": [
                           //PEDIDO CON CANJE
      {
        "id": 0,
        "unitPrice": 5000,
        "quantity": 1,
        "name": "Canje 1 Kilo - 5000 puntos + 50%",
        "sku": "12931",
        "notes": "",
        "canje": 1,
        "promotion": false,
        "optionGroups": [
          {
            "id": 91,
            "name": "Kilo de Helado",
            "sku": "64",
            "unitPrice": 0,
            "notes": "Chocolate, Dulce de Leche, Frutilla, Vainilla"
          }
        ]
      }
                          //PEDIDO SIN PROMOCION NI CANJE
      {
        "id": 0,
        "unitPrice": 10500,
        "quantity": 1,
        "name": "Helado 1 Kg",
        "sku": "64",
        "notes": "Capuccino Granizado Granizado Marroc Grido Super Gridito ",
        "promotion": false,
        "canje": 0,
        "optionGroups": null
       }
                           //PEDIDO CON PROMOCION
      {
       "id": 4521388,
       "unitPrice": 17000,
       "quantity": 1,
       "name": "2 Kilos por $17000 ",
       "sku": "12761",
       "notes": null,
       "promotion": true,
       "canje": 0,
       "optionGroups": [
        {
          "id": 93,
          "name": "Sabores Helado 1 Kg",
          "sku": "64",
          "unitPrice": 0,
          "notes": "Sambayón Marroc Grido Choco Blanco Oreo Flan Dulce
        },
        {
          "id": 93,
          "name": "Sabores Helado 1 Kg",
          "sku": "64",
          "unitPrice": 0,
          "notes": ""
        }
      ]
    }
    ]
  }
}
```

---

## **6. Motivo de Rechazo Específico para CanjeApp**

En el caso de que un canje sea rechazado por falta de puntos, SmartPedidos informará el rechazo utilizando un nuevo motivo específico. Es necesario que PediGrido e I+Diot estén preparados para recibir y mapear este nuevo motivo de rechazo en sus plataformas.

A continuación, se detalla el Id del nuevo objeto que se enviará en la notificación de rechazo:

```json
{
  "id": 400,
  "name": "Falta de puntos CanjeApp",
  "descriptionES": "Falta de puntos CanjeApp",
}
```

<div style="text-align: center; margin-top: 3em; padding-top: 1em; border-top: 1px solid #eee; font-family: 'Source Code Pro', monospace; font-weight: 600; color: #7f8c8d;">
    { smart Pedidos }
</div>
