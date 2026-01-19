---
layout: default
title: Guía de Estilo para IA (Markdown)
---

# Guía de Redacción y Estilo para Asistentes IA

Este documento define cómo una IA debe generar archivos Markdown para que se visualicen correctamente con el diseño "Violeta/Gold" actual del sitio.

## 1. Estructura Básica
Todo archivo `.md` debe comenzar con el siguiente **Front Matter**:

```yaml
---
layout: default
title: Título de la Página
subtitle: Breve descripción (Opcional)
---
```

## 2. Jerarquía y Colores
El CSS está personalizado para usar una paleta **Violeta y Dorado**. Usa los encabezados correctamente para aprovechar esto:

*   **H1 (`#`)**: No usar en el cuerpo. El título se genera automáticamente desde el front matter.
*   **H2 (`##`)**: Úsalo para **Secciones Principales**.
    *   *Estilo:* Color Violeta Oscuro (`#9225eb`) con borde inferior.
*   **H3 (`###`)**: Úsalo para **Sub-secciones**.
    *   *Estilo:* Color Violeta Claro (`#9681f1`).

## 3. Énfasis y Texto
Para destacar conceptos clave, usa **Negrita** (`**texto**`).
*   **Efecto:** El texto se volverá **Dorado/Oro (`#fbbf24`)**.
*   *Uso recomendado:* Nombres de variables importantes, conceptos clave o alertas visuales.

## 4. Bloques de Código (JSON)
El sitio está optimizado para mostrar JSON con alto contraste.
Siempre especifica el lenguaje:

````markdown
```json
{
  "key": "value",
  "number": 123
}
```
````

**Código de Colores en JSON:**
*   **Strings ("valor"):** Se verán **Dorado/Oro (`#fbbf24`)**.
*   **Números y Claves:** Se verán **Violeta Claro (`#9681f1`)**.
*   **Booleanos (true/false) y Keywords:** Se verán **Lavanda/Rosa Suave (`#d4bfff`)**.
*   **Fondo:** Negro Profundo (`#0d1117`).

## 5. Notas Importantes
Para cajas de información, usa la clase HTML `note` (ya que los blockquotes estándar no tienen estilo personalizado):

```html
<div class="note">
  <p><strong>Nota:</strong> Este es un mensaje destacado con borde azul.</p>
</div>
```

## 6. Tablas
Las tablas tienen un estilo oscuro con encabezados azules (legacy).
*   Úsalas para definiciones de campos o matrices de datos.
*   Se renderizan automáticamente con Markdown estándar:

```markdown
| Campo | Tipo | Descripción |
|-------|------|-------------|
| id    | int  | ID único    |
```
