---
layout: default
title: "Especificación Técnica – Formato de Promociones (ThirdParty)"
subtitle: Especificación Técnica – Formato de Promociones
---

## **1. Introducción**

Este documento define la estructura JSON y las reglas de negocio que deben seguir los terceros (ThirdParties) para enviar promociones a smartPedidos. El objetivo es garantizar que los datos lleguen en un formato uniforme y sean interpretados correctamente por los servicios de stock, pricing y facturación.

## **2. Alcance**

*   Describe únicamente el payload del nodo `details` correspondiente a ítems promocionales.
*   Se centra en la deducción de stock basada en los `optionGroups`.

## **3. Audiencia**

*   Desarrolladores backend/front‑end de terceros integradores.
*   Analistas funcionales responsables de catálogos y promociones.

## **4. Glosario**

| Término | Definición |
| :--- | :--- |
| `SKU` | Identificador único de un producto en el sistema de inventario. |
| `Promotion` | Flag booleano que indica que una línea del pedido corresponde a una promoción. |
| `OptionGroup` | Grupo de opciones que detalla los productos reales incluidos en la promo y sobre cuyos SKU se descontará el stock. |

## **5. Estructura General**

```json
"details": [
  {
    "id": 0,                  // Identificador interno de la línea
    "unitPrice": 5000,        // Precio final por unidad promocional
    "quantity": 1,            // Cantidad de promociones (VALOR SIEMPRE EN 1)
    "name": "2 Cuartos por $5000", // Descripción comercial
    "sku": "12760",           // SKU lógico de la promo (no descuenta stock)
    "notes": null,            // Observaciones generales
    "promotion": true,        // Flag obligatorio para indicar promoción
    "optionGroups": [         // Productos reales que conforman UNA promo
      {
        "id": 91,             // ID del grupo de opciones
        "name": "Sabores Helado 1/4 Kg", // Nombre descriptivo
        "sku": "66",          // SKU real que descuenta stock
        "unitPrice": 0,       // Siempre 0 dentro de la promo
        "notes": "Chocolate, Limón" // Sabores para los 2 primeros cuartos
      },
      {
        "id": 91,
        "name": "Sabores Helado 1/4 Kg",
        "sku": "66",
        "unitPrice": 0,
        "notes": "Menta Granizada, Sambayón" // Sabores para los 2 cuartos restantes
      }
    ]
  }
]
```

## **6. Descripción de Campos `details`**

| Campo | Tipo | Obligatorio | Descripción |
| :--- | :--- | :--- | :--- |
| `id` | number | Sí | Identificador secuencial dentro de `details`. No afecta lógica de negocio. |
| `unitPrice` | number | Sí | Precio final por unidad promocional. Moneda definida en la tienda. |
| `quantity` | number | Sí | Cantidad de promociones EL VALOR SIEMPRE ES 1. |
| `name` | string | Sí | Descripción comercial visible al cliente. |
| `sku` | string | Sí | SKU lógico de la promoción (no descuenta stock). |
| `notes` | string \| null | No | Observaciones generales (no confundir con sabores). |
| `promotion` | boolean | Sí | Debe ser `true` para que smartPedidos trate la línea como promoción. |
| `optionGroups` | array | Sí | Lista de productos reales que componen **una sola unidad** de la promo y descuentan stock. |

### **6.1. Estructura de `OptionGroup`**

| Campo | Tipo | Obligatorio | Descripción |
| :--- | :--- | :--- | :--- |
| `id` | number | Sí | ID del grupo de opciones (p. ej. "Sabores Helado 1/4 Kg"). |
| `name` | string | Sí | Nombre descriptivo del grupo. |
| `sku` | string | Sí | SKU real del producto que descuenta stock. |
| `unitPrice` | number | Sí | Debe ser `0` cuando el precio total se informa en la promo. |
| `notes` | string | Recomendado | Detalle específico (sabor, variante, etc.).`. |

## **7. Reglas de Negocio**

1.  **Flag obligatorio:** `promotion` debe enviarse con valor `true`.
2.  **SKU cabecera vs. SKU reales:** El SKU de cabecera (`details.sku`) no descuenta stock; solo los `optionGroups[*].sku`.
3.  **`quantity`SIEMPRE va a ser 1:** La promocion se va a enviar cuantas veces sea necesario, con `quantity = 1`. Es decir si el cliente selecciona 2 promociones, se enviaran dos productos promocion.
4.  **Coherencia cantidad:** El número de líneas en `optionGroups` representa los artículos distintos que integran **una unidad** de la promoción.
5.  **Precio en optionGroups:** Siempre `0`.
6.  **Notas por unidad:** Puede agregarse más de un sabor/variante en `notes` separando por coma cuando se soliciten múltiples unidades de promo.

## **8. Ejemplos**

### **8.1. Promo de 2 Cuartos por $5000 – 1 unidad**

```json
{
  "quantity": 1, // Se solicita 1 promo
  "name": "2 Cuartos por $5000",
  "promotion": true,
  "optionGroups": [
                  { "id": 91, 
                    "name": "Sabores Helado 1/4 Kg", 
                    "sku": "66", 
                    "unitPrice": 0, 
                    "notes": "Chocolate" },
                  { "id": 91, 
                    "name": "Sabores Helado 1/4 Kg", 
                    "sku": "66", 
                    "unitPrice": 0, 
                    "notes": "Frutilla" }
                  ]
 }
```

### **8.2. Promo de 2 Cuartos por $5000 – 2 unidades**

Se solicitan 2 promociones (4 cuartos en total). La estructura de `optionGroups` sigue representando **una** unidad de la promo, pero se envian 2 productos promocion.

```json
{
  "quantity": 1, // Se solicita 1 promo
  "name": "2 Cuartos por $5000",
  "promotion": true,
  "optionGroups": [
                  { "id": 91, 
                    "name": "Sabores Helado 1/4 Kg", 
                    "sku": "66", 
                    "unitPrice": 0, 
                    "notes": "Chocolate" },
                  { "id": 91, 
                    "name": "Sabores Helado 1/4 Kg", 
                    "sku": "66", 
                    "unitPrice": 0, 
                    "notes": "Frutilla" }
                  ]
 }
```

```json
{
  "quantity": 1, // Se solicita 1 promo
  "name": "2 Cuartos por $5000",
  "promotion": true,
  "optionGroups": [
                  { "id": 91, 
                    "name": "Sabores Helado 1/4 Kg", 
                    "sku": "66", 
                    "unitPrice": 0, 
                    "notes": "Limon" },
                  { "id": 91, 
                    "name": "Sabores Helado 1/4 Kg", 
                    "sku": "66", 
                    "unitPrice": 0, 
                    "notes": "Durazno" }
                  ]
 }
```

**Interpretación interna:** Se descuentan 4 cuartos de helado (2 ítems en `optionGroups` en una promocion y 2 ítems en `optionGroups` de la otra promocion) y se enviaron 2 objetos promocion de promocion.

## **9. Validaciones y Errores Comunes**

| Código | Descripción | Ejemplo |
| :--- | :--- | :--- |
| `PROMO-001` | Falta `promotion = true` en línea promocional. | – |
| `PROMO-002` | `optionGroups` vacío o no enviado. | – |
| `PROMO-003` | `optionGroups.sku` inexistente o inactivo. | SKU 999 no existe. |
| `PROMO-004` | `unitPrice` distinto de 0 en un `optionGroup`. | – |
