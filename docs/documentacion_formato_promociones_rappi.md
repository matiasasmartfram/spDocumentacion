---
layout: default
title: "Formato de Promociones y Descuentos – Rappi"
subtitle: "Instructivo de configuración de promociones y descuentos en Rappi"
---

## **1. Conceptos**

En Rappi hay dos mecanismos distintos que pueden parecer similares pero funcionan de forma completamente diferente:

| Mecanismo | ¿Qué hace? | ¿Afecta el nombre del producto? | ¿Agrupa ítems? |
| :--- | :--- | :--- | :--- |
| **Descuento** | Rebaja el precio de un producto | No | Puede ser sobre cualquier tipo de Item o Promo agrupada |
| **Promo** | Agrupa varios productos bajo una cabecera | Sí (debe empezar con `Promo`) | Sí |

Un descuento puede aplicarse sobre **cualquier producto**, incluyendo productos que a su vez forman parte de una promo. Son conceptos independientes.

## **2. Descuentos**

Un descuento es una reducción de precio que Rappi aplica sobre un producto existente o agrupación de ítems: simplemente el producto vale menos.

### **2.1. Características**

- El producto se carga de forma normal en el catálogo, **sin cambios en el nombre ni en el SKU**.
- El descuento se configura desde el panel interno de Rappi de Promociones.
- Se puede aplicar sobre cualquier tipo de producto: un producto individual o un producto agrupado (Promo).
- El nombre del producto **no debe empezar con `Promo`**.

### **2.2. Ejemplo — Descuento sobre un producto a granel**

Un `1/2 kg de Helado` (SKU `65`) con 20% Off configurado por Rappi:

| Elemento | Nombre en Rappi | SKU | Descuento |
| :--- | :--- | :--- | :--- |
| Producto con descuento | 1/2 kg de Helado | **65** | 20% Off (configurado en Rappi) |

El cliente sigue eligiendo sus sabores normalmente (como toppings del producto). El descuento impacta en el precio final, no en la selección de sabores ni en la estructura del pedido.

## **3. Promos**

Una promo es una **agrupación de productos** bajo una cabecera. El cliente elige o recibe múltiples ítems que el sistema vincula entre sí como parte de una misma oferta.

### **3.1. Regla fundamental**

El nombre de **toda promo debe comenzar con la palabra `Promo`** (P mayúscula). Esta es la única forma en que el sistema la reconoce como una agrupación de ítems.

| Correcto | Incorrecto |
| :--- | :--- |
| `Promo Tentacion 50 Off en la 2da Unidad` | `Tentacion 50 Off en la 2da Unidad` |
| `Promo 1 Kilo + 1/2 Kilo` | `1 Kilo + 1/2 Kilo` |

### **3.2. Promo Tipo 1 — Sin granel (productos enteros)**

Se usa para combinar productos individuales: postres, potes, palitos, productos envasados, etc.

#### **Cómo se cargan los SKUs**

La cabecera lleva su SKU numérico de promoción. Los productos dentro de la promo también llevan su SKU numérico propio del producto, es decir exactamente igual a si se vendieran solos.

**Ejemplo — Promo con dos potes:**

| Elemento | Nombre en Rappi | SKU a cargar |
| :--- | :--- | :--- |
| Cabecera | Promo Tentacion 50 Off en la 2da Unidad | **10553** |
| Producto 1 (dentro) | Pote Tentacion Crema Cookie 1 Litro | **83** |
| Producto 2 (dentro) | Pote Tentación Frutilla 1 Litro | **19** |

<div class="note">
  <p>Los productos dentro de la promo usan exactamente el mismo SKU que tienen como productos individuales. No hay ninguna modificación al SKU.</p>
</div>

### **3.3. Promo Tipo 2 — Con granel**

Se usa cuando la promo incluye productos de 1 Kilo, 1/2 Kilo, 1/4 Kilo y el cliente elige los sabores que componen ese peso al hacer el pedido.

Este tipo requiere un formato especial de SKU.

#### **Cómo se carga en el catálogo de Rappi**

Una promo a granel se compone de la **cabecera** más **modificadores**. Todos los modificadores se crean **al mismo nivel** en Rappi — no hay jerarquía visual ni anidamiento: tanto los tamaños como los sabores son modificadores **hermanos**. Lo único que les da el "nivel" (qué es padre, qué es hijo y a qué padre pertenece cada hijo) es el **formato del SKU**, no su posición en el catálogo.

<div class="note">
  <p>En este instructivo (<strong><code>Promo 1 Kilo + 1/2 Kilo</code></strong>) se cargan 4 modificadores, todos hermanos:</p>
  <ul>
    <li><strong>2 de tamaño</strong> → <code>1 Kilo</code> y <code>1/2 Kilo</code>, selección obligatoria, <strong>min 1 / max 1</strong> cada uno.</li>
    <li><strong>2 de sabores</strong> → uno que agrupa los sabores del <code>1 Kilo</code> y otro que agrupa los del <code>1/2 Kilo</code>, cada uno con su propio <strong>min / max</strong> (ej. min 2 / max 4).</li>
  </ul>
</div>

Existen dos tipos de modificadores:

**Cabecera de la promo** — SKU numérico normal.

| Elemento | Nombre | SKU |
| :--- | :--- | :--- |
| Cabecera | Promo 1 Kilo + 1/2 Kilo | **2500** |

**Modificadores PADRE** — representan los tamaños (1 Kilo, 1/2 Kilo, 1/4 Kilo). Son obligatorios: el cliente debe elegir exactamente uno (min 1, max 1).

**Formato: `{SKU numérico}-{Letra}`**

La letra (A, B, C…) es obligatoria. Diferencia los tamaños cuando hay más de uno idéntico en la misma promo.

| Modificador Padre | Nombre | SKU a cargar |
| :--- | :--- | :--- |
| Tamaño 1 | 1 Kilo | **64-A** |
| Tamaño 2 | 1/2 Kilo | **65-A** |

<div class="note">
  <p>Si la promo tuviera dos porciones de 1 Kilo, el primero lleva <code>64-A</code> y el segundo <code>64-B</code>. Si los tamaños son distintos (1 Kilo y 1/2 Kilo), ambos pueden usar <code>-A</code> porque sus números base ya los distinguen.</p>
</div>

**Modificadores HIJO** — los sabores que el cliente elige para **cada** tamaño. Por cada modificador padre se crea **un** modificador de sabores (un único modificador que agrupa todas las opciones de sabor de ese tamaño; no es un modificador por cada sabor). El cliente puede elegir entre 2 y 4 sabores por tamaño (min/max configurable por quien carga el catálogo).

**Formato: `{SKU del padre}-99999`**

Todos los sabores de un mismo padre llevan exactamente el mismo SKU; lo que los distingue es el nombre asignado en el catálogo. **El prefijo del SKU es la llave que vincula cada sabor con su padre.**

| Modificador Hijo | Padre | SKU a cargar | Vinculación |
| :--- | :--- | :--- | :--- |
| Chocolate | 1 Kilo (64-A) | **64-A-99999** | Prefijo `64-A` → padre `64-A` |
| Crema Cookie | 1 Kilo (64-A) | **64-A-99999** | Prefijo `64-A` → padre `64-A` |
| Vainilla | 1 Kilo (64-A) | **64-A-99999** | Prefijo `64-A` → padre `64-A` |
| Chocolate | 1/2 Kilo (65-A) | **65-A-99999** | Prefijo `65-A` → padre `65-A` |
| Crema Cookie | 1/2 Kilo (65-A) | **65-A-99999** | Prefijo `65-A` → padre `65-A` |
| Vainilla | 1/2 Kilo (65-A) | **65-A-99999** | Prefijo `65-A` → padre `65-A` |

<div class="note">
  <p><strong>Regla importante:</strong> por cada modificador padre que se cree, se debe crear su grupo de modificadores hijo con los sabores correspondientes. Un padre sin hijos no tiene sabores para ofrecer.</p>
</div>

#### **Ejemplo completo**

**Configuración en el catálogo de Rappi:**

En Rappi todos los modificadores se crean **al mismo nivel**, sin anidamiento. La relación padre-hijo no es una jerarquía visual del catálogo — la asignamos nosotros al darle a cada SKU su formato específico.

Son **4 modificadores hermanos** (todos al mismo nivel, sin anidar) que cuelgan de la cabecera:

```
Promo 1 Kilo + 1/2 Kilo                         ← Cabecera · SKU: 2500

[MODIFICADOR 1 · PADRE]  Tamaño 1 Kilo          ← SKU: 64-A   · min 1 / max 1 (obligatorio)
[MODIFICADOR 2 · PADRE]  Tamaño 1/2 Kilo        ← SKU: 65-A   · min 1 / max 1 (obligatorio)

[MODIFICADOR 3 · HIJO]   Sabores 1 Kilo         ← min 2 / max 4
      • Chocolate                               ← SKU: 64-A-99999
      • Crema Cookie                            ← SKU: 64-A-99999
      • Vainilla                                ← SKU: 64-A-99999

[MODIFICADOR 4 · HIJO]   Sabores 1/2 Kilo       ← min 2 / max 4
      • Chocolate                               ← SKU: 65-A-99999
      • Crema Cookie                            ← SKU: 65-A-99999
      • Vainilla                                ← SKU: 65-A-99999
```

<div class="note">
  <p>Los 4 modificadores se cargan al mismo nivel en Rappi. El <code>min/max</code> de los modificadores <strong>3</strong> y <strong>4</strong> aplica al modificador de sabores completo (cuántos sabores puede elegir el cliente), no a cada sabor por separado. Dentro de cada modificador de sabores, todas las opciones comparten el mismo SKU; su prefijo (<code>64-A</code> / <code>65-A</code>) es lo que las vincula a su tamaño.</p>
</div>

**¿Cómo sabe el sistema qué hijo le corresponde a qué padre?**

Por el prefijo del SKU — que nosotros asignamos al cargar el catálogo:

- `64-A-99999` → el prefijo `64-A` lo vincula al padre `64-A` (1 Kilo)
- `65-A-99999` → el prefijo `65-A` lo vincula al padre `65-A` (1/2 Kilo)

**Pedido del cliente:** 1 Kilo con Chocolate, Crema Cookie y Vainilla / 1/2 Kilo con Vainilla.

El sistema reagrupa automáticamente los sabores por contenedor usando el prefijo del SKU:

- 1 Kilo → sabores: Chocolate, Crema Cookie, Vainilla
- 1/2 Kilo → sabores: Vainilla
