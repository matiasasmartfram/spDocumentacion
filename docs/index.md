Claro que sí. Aquí tienes la documentación de la **Celda 1** formateada estrictamente en Markdown.

***

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



