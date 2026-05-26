---
layout: default
title: "Formato de Promociones y Descuentos – PedidosYa (Grido)"
subtitle: "Instructivo de configuración de promociones y descuentos en PedidosYa"
---

## **1. Conceptos**

En PedidosYa hay dos mecanismos distintos que pueden parecer similares pero funcionan de forma completamente diferente:

| Mecanismo | ¿Qué hace? | ¿Afecta el nombre del producto? | ¿Agrupa ítems? |
| :--- | :--- | :--- | :--- |
| **Descuento** | Rebaja el precio de un producto | No | Puede ser sobre cualquier tipo de Item o Promo agrupada |
| **Promo** | Agrupa varios productos bajo una cabecera | Sí (debe empezar con `Promo`) | Sí |

Un descuento puede aplicarse sobre **cualquier producto**, incluyendo productos que a su vez forman parte de una promo. Son conceptos independientes.

## **2. Descuentos**

Un descuento es una reducción de precio que PedidosYa aplica sobre un producto existente o agrupación de ítems: simplemente el producto vale menos.

### **2.1. Características**

- El producto se carga de forma normal en el catálogo, **sin cambios en el nombre ni en el SKU**.
- El descuento se configura desde el panel de promociones de PedidosYa y llega en el pedido de forma separada (no se codifica dentro del SKU).
- El descuento llega por producto y también como total del pedido. Si el descuento proviene de un cupón, PedidosYa lo informa como voucher y se refleja en las observaciones del pedido (`Descuento por Voucher PedidosYa`).
- Se puede aplicar sobre cualquier tipo de producto: un producto individual o un producto agrupado (Promo).
- El nombre del producto **no debe empezar con `Promo`**.

### **2.2. Ejemplo — Descuento sobre un producto a granel**

Un `1/2 kg de Helado` (SKU `65`) con 20% Off configurado por PedidosYa:

| Elemento | Nombre en PedidosYa | SKU | Descuento |
| :--- | :--- | :--- | :--- |
| Producto con descuento | 1/2 kg de Helado | **65** | 20% Off (configurado en PedidosYa) |

El cliente sigue eligiendo sus sabores normalmente (como modificadores del producto). El descuento impacta en el precio final, no en la selección de sabores ni en la estructura del pedido.

## **3. Promos**

Una promo es una **agrupación de productos** bajo una cabecera. El cliente elige o recibe múltiples ítems que el sistema vincula entre sí como parte de una misma oferta.

### **3.1. Regla fundamental**

El nombre de **toda promo debe comenzar con la palabra `Promo`**. El sistema reconoce la promo cuando el nombre del producto comienza con `Promo`; la detección **no distingue mayúsculas de minúsculas** (el sistema normaliza el texto), pero se recomienda escribirla con `P` mayúscula. Esta es la única forma en que el sistema la reconoce como una agrupación de ítems.

| Correcto | Incorrecto |
| :--- | :--- |
| `Promo Tentacion 50 Off en la 2da Unidad` | `Tentacion 50 Off en la 2da Unidad` |
| `Promo 1 Kilo + 1/2 Kilo` | `1 Kilo + 1/2 Kilo` |

<div class="note">
  <p><strong>Diferencia clave con Rappi / MercadoPago:</strong> en PedidosYa <strong>no se usa el formato de SKU con guion</strong> (<code>64-A</code>, <code>64-A-99999</code>). La relación entre la cabecera, los contenedores y los sabores la provee la <strong>estructura nativa de modificadores de PedidosYa</strong> (grupos de modificadores → opciones). El sabor se identifica porque su SKU es <strong><code>99999</code></strong>.</p>
</div>

### **3.2. Promo Tipo 1 — Sin granel (productos enteros)**

Se usa para combinar productos individuales: postres, potes, palitos, productos envasados, etc.

#### **Cómo se cargan los SKUs**

La cabecera (el producto cuyo nombre empieza con `Promo`) lleva su SKU numérico de promoción. Los productos dentro de la promo se cargan como **opciones dentro de un grupo de modificadores**, y cada uno lleva su **SKU numérico propio del producto** (válido, distinto de `99999`), es decir exactamente igual a si se vendieran solos.

Cada opción con SKU válido se convierte en una **línea propia** dentro del pedido (un ítem de la promo).

**Ejemplo — Promo con dos potes:**

| Elemento | Nombre en PedidosYa | SKU a cargar |
| :--- | :--- | :--- |
| Cabecera (producto) | Promo Tentacion 50 Off en la 2da Unidad | **10553** |
| Grupo de modificadores | Elegí tus potes | — |
| Opción 1 (dentro del grupo) | Pote Tentacion Crema Cookie 1 Litro | **83** |
| Opción 2 (dentro del grupo) | Pote Tentación Frutilla 1 Litro | **19** |

<div class="note">
  <p>Los productos dentro de la promo usan exactamente el mismo SKU que tienen como productos individuales. No hay ninguna modificación al SKU. Como cada opción tiene un SKU válido (≠ <code>99999</code>), el sistema genera una línea independiente para cada una.</p>
</div>

### **3.3. Promo Tipo 2 — Con granel**

Se usa cuando la promo incluye productos de 1 Kilo, 1/2 Kilo, 1/4 Kilo y el cliente elige los sabores que componen ese peso al hacer el pedido.

A diferencia de Rappi / MercadoPago, **no se usa un formato especial de SKU con guion**. En PedidosYa cada contenedor (tamaño) es un **grupo de modificadores** y los sabores son sus **opciones**.

#### **Cómo se carga en el catálogo de PedidosYa**

Una promo a granel se compone de la **cabecera** (el producto `Promo`) más uno o varios **grupos de modificadores**, uno por cada contenedor/tamaño. Dentro de cada grupo se cargan los **sabores** disponibles.

La relación contenedor → sabores la da la **estructura nativa de PedidosYa**: cada sabor está literalmente anidado dentro de su grupo. No hace falta ninguna letra ni prefijo de SKU.

**Cabecera de la promo** — el producto `Promo`, con SKU numérico normal.

| Elemento | Nombre | SKU |
| :--- | :--- | :--- |
| Cabecera | Promo 1 Kilo + 1/2 Kilo | **2500** |

**Contenedor granel (grupo de modificadores)** — representa el tamaño (1 Kilo, 1/2 Kilo, 1/4 Kilo). Se crea **un grupo por cada tamaño** y el grupo lleva como SKU el **número del producto granel** correspondiente.

| Contenedor (grupo) | Nombre del grupo | SKU del grupo |
| :--- | :--- | :--- |
| Tamaño 1 | 1 Kilo | **64** |
| Tamaño 2 | 1/2 Kilo | **65** |

<div class="note">
  <p>A diferencia de Rappi/MercadoPago, el contenedor <strong>no lleva letra</strong> (<code>64-A</code>). Cada tamaño es un grupo de modificadores independiente, y eso ya los distingue.</p>
</div>

**Sabores (opciones del grupo)** — los sabores que el cliente elige para **cada** contenedor. Por cada grupo (tamaño) se cargan sus sabores como opciones. El cliente puede elegir entre 2 y 4 sabores por tamaño (min/max configurable por quien carga el catálogo).

**Todos los sabores se cargan con SKU `99999`.**

El SKU `99999` es la marca que le indica al sistema que la opción es un **sabor** (texto descriptivo) y no un producto con línea propia. El sistema agrupa automáticamente todos los sabores `99999` de un grupo bajo la línea de ese contenedor.

| Sabor (opción) | Contenedor (grupo) | SKU a cargar |
| :--- | :--- | :--- |
| Chocolate | 1 Kilo (64) | **99999** |
| Crema Cookie | 1 Kilo (64) | **99999** |
| Vainilla | 1 Kilo (64) | **99999** |
| Chocolate | 1/2 Kilo (65) | **99999** |
| Crema Cookie | 1/2 Kilo (65) | **99999** |
| Vainilla | 1/2 Kilo (65) | **99999** |

<div class="note">
  <p><strong>Regla importante:</strong> los sabores no llevan SKU propio; todos van con <code>99999</code>. Lo que los vincula a su contenedor es estar cargados <strong>dentro del grupo de ese tamaño</strong>. Un grupo sin sabores no tiene opciones para ofrecer.</p>
</div>

#### **Ejemplo completo**

**Configuración en el catálogo de PedidosYa:**

A diferencia de Rappi/MercadoPago, en PedidosYa los sabores se cargan **anidados dentro de su grupo** (estructura jerárquica nativa). La relación contenedor → sabor no la asignamos por SKU, la da el propio anidamiento del catálogo.

```
Promo 1 Kilo + 1/2 Kilo                       ← Producto cabecera (name empieza con "Promo")   SKU: 2500

[GRUPO MODIFICADOR]  1 Kilo                    ← SKU del grupo: 64       (sabores: min 2 / max 4)
   [OPCIÓN]  Chocolate                         ← SKU: 99999  (sabor → texto)
   [OPCIÓN]  Crema Cookie                      ← SKU: 99999  (sabor → texto)
   [OPCIÓN]  Vainilla                          ← SKU: 99999  (sabor → texto)
[GRUPO MODIFICADOR]  1/2 Kilo                  ← SKU del grupo: 65       (sabores: min 2 / max 4)
   [OPCIÓN]  Chocolate                         ← SKU: 99999  (sabor → texto)
   [OPCIÓN]  Crema Cookie                      ← SKU: 99999  (sabor → texto)
   [OPCIÓN]  Vainilla                          ← SKU: 99999  (sabor → texto)
```

**¿Cómo sabe el sistema qué sabor le corresponde a qué contenedor?**

Por el anidamiento nativo del catálogo: cada sabor (SKU `99999`) está cargado dentro del grupo de su tamaño.

- Los sabores cargados dentro del grupo `1 Kilo` (SKU 64) se agrupan en la línea del contenedor 1 Kilo.
- Los sabores cargados dentro del grupo `1/2 Kilo` (SKU 65) se agrupan en la línea del contenedor 1/2 Kilo.

**Pedido del cliente:** 1 Kilo con Chocolate, Crema Cookie y Vainilla / 1/2 Kilo con Vainilla.

El sistema reagrupa automáticamente los sabores por contenedor usando el anidamiento del grupo:

- 1 Kilo (línea con SKU 64) → sabores: Chocolate, Crema Cookie, Vainilla
- 1/2 Kilo (línea con SKU 65) → sabores: Vainilla
