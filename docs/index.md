### 1. Inicialización de Contexto e Ingesta de Fuentes de Datos

**Objetivo del Bloque:**
Establecer los parámetros globales de la integración y realizar la ingesta inicial de los archivos de origen (ERP), normalizando su estructura técnica para permitir un procesamiento unificado de productos y promociones.

**Lógica de Negocio y Transformación:**

1.  **Definición de Contexto de Integración:**
    *   Se configuran las variables maestras que inyectarán información de contexto en el encabezado del archivo final. Esto incluye:
        *   **Lista de Precios:** Se define qué tarifa del ERP se utilizará para valorizar los productos (asegurando coherencia con los precios de la app de delivery).
        *   **Plataforma Destino:** Se etiqueta el origen de la integración (ej. PedidosYa) para trazabilidad.
        *   **Sucursales Objetivo:** Se delimita el alcance geográfico de la integración especificando los identificadores de las tiendas involucradas.

2.  **Ingesta y Normalización (Wrapping):**
    *   El sistema carga dos fuentes de datos independientes:
        *   **Catálogo de Items:** Productos unitarios (Stock, Insumos, Productos de Venta).
        *   **Matriz de Promociones:** Estructuras complejas (Combos, Menús).
    *   Para estandarizar el flujo, se aplica un patrón de **"Envoltura" (Wrapping)**. En lugar de procesar los datos crudos directamente, cada registro se encapsula identificando explícitamente su naturaleza.

**Impacto en la Estructura del JSON:**

Los datos evolucionan de una estructura plana a una estructura tipada y encapsulada.

*   **Entrada (Raw ERP):**
    Objeto simple directo del archivo.
*   **Salida (Memoria Normalizada):**
    Se genera un objeto contenedor que separa los metadatos del contenido real.

    ```json
    {
      "type": "item",       // O "promo", define la naturaleza del objeto
      "data": {             // Contiene el objeto original del ERP intacto
         "id": "...",
         "description": "...",
         ...
      }
    }
    ```

**Resultado:**
Se dispone de dos colecciones unificadas en memoria, donde cada elemento es consciente de su propio tipo, permitiendo aplicar reglas de validación específicas en los pasos siguientes sin mezclar peras con manzanas.


### 1.5. Enriquecimiento del Catálogo: Gestión Master de Imágenes

**Objetivo del Bloque:**
Integrar un repositorio externo de activos digitales (URLs de imágenes) en la estructura del catálogo. El objetivo es asegurar que tanto los productos principales como sus componentes internos (modificadores o ingredientes visuales) cuenten con su referencia gráfica correspondiente antes de continuar con el procesamiento de reglas de negocio.

**Lógica de Negocio y Transformación:**

1.  **Carga del Diccionario de Activos:**
    *   Se ingesta un archivo maestro que funciona como un "mapa de referencia" (Índice Clave-Valor), relacionando identificadores únicos de productos con sus URLs de imagen alojadas en la nube.
    *   Se implementa un mecanismo de **resiliencia**: si el archivo de imágenes no existe o está corrupto, el proceso no se detiene; simplemente asume que ningún producto tiene imagen y continúa, reportando una alerta.

2.  **Inyección Jerárquica (Full Depth):**
    *   El proceso recorre el catálogo normalizado y aplica una estrategia de enriquecimiento en profundidad:
    *   **Nivel Producto (Padre):** Se busca el identificador del producto vendible en el mapa de imágenes. Si existe coincidencia, se inyecta el atributo de imagen en el nivel raíz del objeto y en su sub-estructura de definición para garantizar redundancia y compatibilidad.
    *   **Nivel Composición (Hijos/Ingredientes):** Se detecta si el producto es compuesto (ej. un combo o un plato con guarniciones). De ser así, se explora su estructura interna de ingredientes ("composition"). Cada sub-item es validado contra el mapa de imágenes y, si tiene foto propia (ej. la foto de una gaseosa dentro de un combo), se le asigna su URL específica.

**Impacto en la Estructura del JSON:**

Se agregan atributos visuales en múltiples niveles de la jerarquía del objeto.

*   **Antes:** El objeto solo contenía datos descriptivos y precios.
*   **Después:** Aparece el atributo `imageUrl` tanto en el producto principal como en sus componentes anidados.

```json
{
  "type": "item",
  "data": {
     "id": "1001",
     "name": "Hamburguesa Completa",
     "imageUrl": "https://cdn.../burger.jpg",  // <-- Nuevo Atributo (Nivel Padre)
     "item": {
        "imageUrl": "https://cdn.../burger.jpg", // <-- Inyección redundante por consistencia
        "composition": {
           "items": [
              {
                 "item": {
                    "id": "500",
                    "name": "Papas Fritas",
                    "imageUrl": "https://cdn.../fries.jpg" // <-- Nuevo Atributo (Nivel Hijo)
                 }
              }
           ]
        }
     }
  }
}
```

**Resultado:**
El catálogo ahora posee riqueza visual (Visual Richness). Los productos y sus modificadores ya no son solo texto; tienen referencias a imágenes listas para ser renderizadas por la App de Delivery, mejorando la experiencia de usuario final.

### 1.6. Gestión de Imágenes para Promociones y Estrategia de Fallback

**Objetivo del Bloque:**
Asignar recursos gráficos específicos a las estructuras promocionales (Combos y Menús). A diferencia de los productos simples, las promociones requieren una lógica de búsqueda especializada basada en códigos de enlace externos y cuentan con un mecanismo de seguridad para garantizar que nunca aparezcan sin imagen.

**Lógica de Negocio y Transformación:**

1.  **Filtrado de Activos de Marketing:**
    *   Del repositorio maestro de imágenes, el sistema aísla un subconjunto específico etiquetado para uso promocional (filtro por "Tenant"). Esto evita mezclar fotos de stock de ingredientes con las artes de marketing diseñadas para los combos.

2.  **Resolución por Código de Enlace (SKU Type Target):**
    *   Para conectar una promoción con su imagen, no se utiliza el ID interno del ERP. En su lugar, el sistema explora la lista de códigos asociados a la promoción y busca un **"Código de Vinculación"** específico (identificado técnicamente como *SKU Type 6*).
    *   Este código actúa como la llave para recuperar la imagen personalizada del mapa de activos.

3.  **Estrategia de Imagen por Defecto (Fallback):**
    *   Se implementa una regla de negocio crítica para la consistencia visual: **"Prohibido mostrar items vacíos"**.
    *   Si una promoción no tiene el código de vinculación, o si la imagen asociada no existe, el sistema asigna automáticamente una **Imagen de Marca Genérica** (Logo Corporativo). Esto asegura que el catálogo final luzca profesional y completo, incluso si faltan fotos específicas.

**Impacto en la Estructura del JSON:**

Los objetos de tipo "promo" (en la colección `relations`) reciben un nuevo atributo visual.

*   **Antes:** El objeto contenía definiciones de reglas y códigos, pero carecía de presentación visual.
*   **Después:**
    ```json
    {
      "type": "promo",
      "data": {
         "id": "9001",
         "name": "Combo Familiar",
         "relationCodes": [
            { "skuTypeId": 6, "code": "PROMO_VERANO" } // Código llave detectado
         ],
         "imageUrl": "https://.../promo_especific.jpg" // Imagen resuelta exitosamente
      }
    }
    ```
    *Caso Fallback (Sin imagen específica):*
    ```json
    {
         "name": "Combo Nuevo",
         "imageUrl": "https://.../brand_logo_default.jpg" // Imagen por defecto aplicada
    }
    ```

**Resultado:**
El 100% de las promociones cuentan ahora con una URL de imagen válida (ya sea la específica de la campaña o el logo de la marca), listas para su publicación.


### 2. Normalización de Modificadores: Agrupación de "Quitar Ingredientes"

**Objetivo del Bloque:**
Estandarizar la presentación de los ingredientes opcionales (aquellos que el cliente puede eliminar de su pedido). El objetivo es consolidar todos los modificadores negativos bajo un único grupo funcional en la interfaz de usuario, independientemente de su categoría original en el ERP.

**Lógica de Negocio y Transformación:**

1.  **Detección de Modificadores Opcionales:**
    *   El sistema inspecciona la estructura de composición de cada producto compuesto (ej. una hamburguesa).
    *   Se identifican aquellos ingredientes que poseen el indicador de "Opcionalidad" activo (`optional: true`). Esto señala que el ingrediente viene por defecto, pero puede ser retirado.

2.  **Re-categorización Forzada (Unificación):**
    *   Originalmente, estos ingredientes podrían pertenecer a grupos dispersos (ej. "Lácteos", "Verduras", "Salsas").
    *   Para mejorar la experiencia de usuario en la App de Delivery, se aplica una regla de **Sobreescritura de Grupo**.
    *   Cualquier ítem detectado como opcional es movido lógicamente a un **Grupo Estándar de "Quitar Ingredientes"**. Esto asegura que, en el menú final, el usuario vea una única lista limpia de lo que puede eliminar, en lugar de buscar en múltiples categorías.

**Impacto en la Estructura del JSON:**

Se modifica el objeto `group` dentro de los detalles de la composición del ítem.

*   **Antes:** El ingrediente mantenía su clasificación original de inventario.
    ```json
    {
      "item": {
         "name": "Queso Cheddar",
         "group": { "id": 50, "name": "Lácteos e Insumos" } // Categoría interna del ERP
      },
      "optional": true
    }
    ```

*   **Después:** El ingrediente es reasignado al grupo funcional de interfaz.
    ```json
    {
      "item": {
         "name": "Queso Cheddar",
         "group": {
             "id": 10,
             "name": "Quitar Ingredientes", // Categoría unificada para la App
             "description": "Quitar Ingredientes",
             "isKds": false
         }
      },
      "optional": true
    }
    ```

**Resultado:**
Todos los ingredientes removibles quedan organizados bajo una misma etiqueta, simplificando la navegación del menú y evitando la fragmentación de opciones de personalización negativa.


### 3. Gestión de Visibilidad y Exclusión (Blacklist Logic)

**Objetivo del Bloque:**
Implementar un mecanismo de gobernanza de datos que permita excluir productos o categorías enteras del canal de venta digital. El objetivo es asegurar que ítems de uso interno (como insumos, inventario o pruebas) o productos no aptos para delivery sean ocultados programáticamente del menú final.

**Lógica de Negocio y Transformación:**

1.  **Configuración de Listas de Exclusión (Blacklists):**
    *   Se definen listas configurables de identificadores prohibidos. El sistema normaliza estos IDs (manejando tanto formatos numéricos como de texto) para asegurar una comparación robusta y evitar errores por incompatibilidad de tipos de datos.

2.  **Evaluación de Visibilidad Jerárquica:**
    *   El proceso recorre el catálogo y aplica un filtrado en cascada:
        *   **Regla de Grupo (Categoría):** Si un producto pertenece a un grupo marcado como restringido (ej. "Insumos de Cocina" o "Bebidas Alcohólicas sin Stock"), se oculta automáticamente todo su contenido.
        *   **Regla de SKU (Ítem Individual):** Si el producto supera el filtro de grupo, se verifica su ID específico contra la lista negra de ítems individuales. Esto permite ocultar productos puntuales (ej. un plato de temporada retirado) sin afectar al resto de su categoría.

3.  **Desactivación Lógica (Soft Disable):**
    *   Al detectar una coincidencia positiva en las listas de exclusión, el sistema no elimina el registro del JSON (para mantener la integridad referencial si otros procesos lo consultan). En su lugar, altera sus banderas de estado (`flags`) para "apagarlo".

**Impacto en la Estructura del JSON:**

Se modifican los atributos de control de venta y habilitación. Se fuerzan a `false` para garantizar que la plataforma de destino ignore estos ítems al renderizar el menú.

*   **Antes (Estado Default):** El ítem se asume activo y vendible.
    ```json
    {
      "enabled": true,
      "item": {
         "id": "126",
         "name": "Vino Malbec Reserva",
         "group": { "id": "50", "name": "Bodega" },
         "forSale": true
      }
    }
    ```

*   **Después (Tras aplicar Blacklist):** El ítem queda deshabilitado y retirado de la venta.
    ```json
    {
      "enabled": false,        // <-- Se apaga a nivel raíz (Sistema)
      "item": {
         "id": "126",
         "name": "Vino Malbec Reserva",
         "group": { "id": "50", "name": "Bodega" },
         "forSale": false      // <-- Se marca explícitamente como NO disponible comercialmente
      }
    }
    ```

**Resultado:**
Un catálogo depurado y seguro ("Sanitized Catalog"), donde solo los productos autorizados comercialmente están marcados como `forSale: true`. Los elementos internos o restringidos permanecen en la estructura técnica pero son invisibles para el cliente final.



### 4. Estrategia de Precios en Promociones: Sobrecargos Jerárquicos

**Objetivo del Bloque:**
Definir la lógica de valorización para los componentes dentro de un Combo o Menú. No todos los ítems intercambiables tienen el mismo costo (ej. cambiar agua por vino, o papas por aros de cebolla). Se implementa un sistema de reglas de precios para asignar el "Costo Extra" (`promotionValue`) que se cobrará al cliente al seleccionar ciertas opciones.

**Lógica de Negocio y Transformación:**

1.  **Mapeo de Referencia Cruzada:**
    *   Para poder aplicar reglas basadas en categorías, el sistema primero construye un mapa relacional cruzando el *Catálogo de Productos* con la *Matriz de Promociones*. Esto permite que el motor de precios sepa a qué "Familia" o "Grupo" pertenece cada ítem opcional dentro de un combo.

2.  **Modelo de Precios Jerárquicos (Override Logic):**
    *   Se aplica una lógica de evaluación en cascada para determinar el sobrecargo, priorizando la especificidad sobre la generalidad:
        *   **Nivel 1: Regla Específica por Artículo (Prioridad Alta):** El sistema verifica si el ID del producto tiene un precio forzado manualmente. Esto se usa para excepciones de alto valor (ej. una botella de vino premium dentro de un menú estándar). Si existe esta regla, se aplica y se detiene la búsqueda.
        *   **Nivel 2: Regla General por Grupo (Prioridad Media):** Si no hay regla específica, se verifica el grupo del producto. Si el grupo tiene una política de precios definida (ej. "Todas las Bebidas Alcohólicas suman $100"), se aplica ese valor.

3.  **Asignación de Valor Promocional:**
    *   El valor resultante se inyecta en la estructura de la promoción, asegurando que la plataforma de delivery cobre el diferencial correcto al momento de la selección.

**Impacto en la Estructura del JSON:**

Se enriquece el array `details` dentro de los grupos de opciones de la promoción, agregando o actualizando el atributo `promotionValue`.

*   **Antes (Definición Base):** El ítem era una opción elegible sin impacto económico definido.
    ```json
    {
       "articleId": 126, // Vino Reserva
       "min": 0,
       "max": 1
       // No existe 'promotionValue' o es 0 por defecto
    }
    ```

*   **Después (Tras cálculo de Sobrecargos):**
    *   *Caso Excepción Específica:*
    ```json
    {
       "articleId": 126,
       "promotionValue": 150.00  // Precio específico para este SKU
    }
    ```
    *   *Caso Regla de Grupo:*
    ```json
    {
       "articleId": 99, // Cerveza (Parte del grupo Alcohol)
       "promotionValue": 100.00  // Precio heredado de la regla de grupo
    }
    ```

**Resultado:**
Las promociones ahora poseen lógica financiera inteligente. El precio final del combo es dinámico y reacciona a la selección del usuario, cobrando los suplementos correspondientes según reglas de negocio configurables y escalables.


### 5. Clasificación Semántica y Taxonomía de Productos Compuestos

**Objetivo del Bloque:**
Estructurar el menú de cara al usuario final mediante la categorización automática de los productos compuestos. El objetivo es derivar la "Sección" o "Categoría" correcta en la App de Delivery (ej. "Combos", "Promociones") basándose en el nombre comercial del producto, asegurando una navegación intuitiva.

**Lógica de Negocio y Transformación:**

1.  **Análisis Semántico de Nombres (Case Insensitive):**
    *   El proceso inspecciona el atributo `name` de cada promoción.
    *   Se aplica una **Normalización de Texto**: el sistema ignora las variaciones de escritura (mayúsculas, minúsculas o mezclas) para asegurar que "COMBO", "Combo" y "combo" se traten como la misma entidad lógica.

2.  **Reglas de Asignación por Palabras Clave:**
    *   Se implementa un motor de reglas condicionales basado en prefijos para reclasificar el tipo de objeto:
        *   **Combos:** Si el nombre comienza con la palabra "Combo", el ítem se etiqueta bajo la categoría maestra **"Combos"**.
        *   **Hamburguesas:** Si comienza con "Hamburguesa", se mueve a la categoría **"Hamburguesas"**.
        *   **Ofertas:** Si el nombre inicia con "Promo" o "Promoción", se asigna a la categoría **"Promociones"**.
    *   **Persistencia:** Si el nombre no coincide con ninguna de estas reglas, el ítem mantiene su clasificación original, evitando pérdidas de datos.

**Impacto en la Estructura del JSON:**

Se actualiza el atributo `type` del objeto contenedor (el "wrapper" generado en el Paso 1). Este campo deja de ser un identificador técnico genérico ("promo") y pasa a ser un identificador de negocio utilizable como título de sección en la UI.

*   **Antes (Clasificación Genérica):**
    ```json
    {
      "type": "promo",          // Tipo técnico interno
      "data": {
         "name": "Combo Doble Cuarto",
         ...
      }
    }
    ```

*   **Después (Clasificación Semántica):**
    ```json
    {
      "type": "Combos",         // Nueva Categoría de Negocio
      "data": {
         "name": "Combo Doble Cuarto",
         ...
      }
    }
    ```

**Resultado:**
El listado de promociones deja de ser una "bolsa de gatos" desordenada. Ahora posee una taxonomía clara donde los ítems están agrupados lógicamente por su naturaleza comercial, listos para ser presentados en las pestañas o secciones correspondientes de la plataforma de pedidos.

