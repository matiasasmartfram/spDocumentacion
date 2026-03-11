---
layout: default
title: Integración Uber Eats Marketplace API
subtitle: Guía para la autenticación y gestión de pedidos en entorno Sandbox
---

## Introducción

Este documento detalla el proceso para autenticar una aplicación de terceros (POS/Middleware) con la plataforma **Uber Eats** en entorno de pruebas (**Sandbox**), permitiendo la recepción y gestión de pedidos simulados.

## Credenciales del Entorno

Para este proyecto se utilizan dos tipos de cuentas:

*   **Cuenta de Desarrollador (Dev Account)**: Utilizada para gestionar la aplicación en el **Uber Developer Dashboard**.
    *   **App Name**: `SmartPedidosTest`
    *   **Client ID**: `fmxmwoqTZqLwtsSytOmO5_5t_-vcmuQo`
*   **Tienda de Prueba (Test Store)**: Representa el restaurante físico que recibirá los pedidos.
    *   **Store UUID**: `007bde17-4be8-45fb-b5c3-7ca6b67b441c`

## Configuración Previa (Dashboard)

Antes de solicitar un token, la aplicación en el **Dashboard de Uber** debe cumplir tres requisitos:

1.  **Redirect URI**: Debe haber una URL configurada (ej. `http://localhost:8080`) para recibir el código de autorización.
2.  **Privacy Policy URL**: Uber requiere una URL válida (aunque sea de prueba) para habilitar los permisos (**Scopes**).
3.  **Scopes**: Para aplicaciones nuevas de tipo **Marketplace**, el permiso inicial suele ser `eats.pos_provisioning`.

## Proceso de Autenticación (OAuth 2.0)

Uber utiliza el flujo de **Authorization Code Grant** para asegurar que el dueño de la tienda autoriza explícitamente a la aplicación.

### Paso 1: Obtención del Código de Autorización

Se debe generar una URL de autorización que el dueño de la tienda abrirá en su navegador.

**URL de Sandbox**:
```text
https://sandbox-login.uber.com/oauth/v2/authorize?client_id=fmxmwoqTZqLwtsSytOmO5_5t_-vcmuQo&redirect_uri=http://localhost:8080&response_type=code&scope=eats.pos_provisioning
```

1.  El usuario ingresa las credenciales de la **Tienda de Prueba**.
2.  Al aceptar los términos, Uber redirige a la **Redirect URI** configurada.
3.  Se debe extraer el código de la URL: `http://localhost:8080/?code=XXXXXX`.

<div class="note">
  <p><strong>Nota:</strong> El código tiene una validez de 10 minutos y es de un solo uso.</p>
</div>

### Paso 2: Intercambio de Código por Access Token

Con el código obtenido, se realiza una petición **POST** para obtener el token definitivo. Es vital usar el subdominio **sandbox-login** para aplicaciones de prueba.

**Comando cURL**:
```bash
curl -X POST "https://sandbox-login.uber.com/oauth/v2/token" \
     -H "Content-Type: application/x-www-form-urlencoded" \
     -d "client_secret=TU_CLIENT_SECRET" \
     -d "client_id=fmxmwoqTZqLwtsSytOmO5_5t_-vcmuQo" \
     -d "grant_type=authorization_code" \
     -d "redirect_uri=http://localhost:8080" \
     -d "code=CODIGO_OBTENIDO"
```

El JSON de respuesta contendrá el **access_token**, el cual tiene una duración de 30 días.

## El Ciclo de Vida del Pedido (Flujo de Pruebas)

Una vez obtenido le token, el proceso para probar la integración es el siguiente:

### 1. Inyección del Pedido (Simulación)

Dado que no se pueden realizar pedidos reales en **Sandbox** mediante API de cliente, se utiliza el **Order Injector**:

*   **Herramienta**: **Uber Eats Order Injector**
*   Se ingresa el **Store UUID** y se seleccionan los productos. Al hacer clic en "Create Order", Uber dispara el flujo hacia tu sistema.

### 2. Notificación (Webhook)

Uber envía un evento `orders.notification` a tu servidor. Tu sistema debe:

1.  Recibir el **order_id**.
2.  Responder con un **200 OK** inmediatamente.

### 3. Obtención de Detalles

Tu sistema consulta la API de Uber para conocer el contenido del pedido:

*   **Endpoint**: `GET /v1/eats/orders/{order_id}`
*   **Auth**: `Authorization: Bearer <TOKEN>`

### 4. Aceptación del Pedido (Confirmación POS)

Para finalizar, el POS debe confirmar que ha procesado el pedido correctamente:

*   **Endpoint**: `POST /v1/eats/orders/{order_id}/accept_pos_order`

<div class="note">
  <p><strong>Importante:</strong> Si no se acepta en un periodo de pocos minutos, Uber cancelará el pedido automáticamente por falta de respuesta del restaurante.</p>
</div>

## Consideraciones Técnicas de Sandbox

*   **Subdominios**: Utilizar siempre `sandbox-login.uber.com` para autenticación y la base `https://api.uber.com` para los endpoints de recursos.
*   **Errores Comunes**:
    *   `invalid_grant`: El código ha expirado o ya fue utilizado.
    *   `mismatched environment`: Se está intentando usar una App de Test en el servidor de producción (`auth.uber.com`).
    *   `invalid_scope`: El permiso solicitado no está habilitado en el Dashboard o el nombre es incorrecto.
