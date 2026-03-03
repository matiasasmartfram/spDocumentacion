---
layout: default
title: OpenClose - Registro de Aperturas y Cierres
subtitle: Documentación del registro opencloselogs generado por el Monitor de Conectividad y el Cron de Sincronización
---

## Introducción

Este documento tiene como objetivo explicar, en términos de negocio, el funcionamiento y contenido del registro de actividades denominado `opencloselogs` (o `openClosedLog`).

Este registro almacena los resultados de los procesos automáticos encargados de **abrir o cerrar tiendas** en las plataformas de delivery (PedidosYa, Rappi, Uber Eats) basándose en la conectividad del local o en horarios programados.

Su función principal es permitir monitorear cómo y por qué el sistema decidió intervenir en la disponibilidad de una sucursal en las aplicaciones.

---

## Análisis de Atributos del Documento

A continuación, se detalla el significado de cada campo (variable) dentro de un registro de esta colección, utilizando el ejemplo proporcionado y el análisis del sistema.

### 1. `_id`
*   **Que es:** El identificador único del registro en la base de datos.
*   **Valor:** Una cadena alfanumérica generada automáticamente (Ej: `696e3f35335f84eaebebe4dd`).
*   **Significado:** Referencia técnica única para este evento específico.

### 2. `result`
*   **Que es:** El resultado general del proceso masivo de todas las tiendas en el lote.
*   **Valores Posibles:**
    *   `true` (Verdadero): Todas las acciones intentadas en este lote se realizaron con éxito en la plataforma.
    *   `false` (Falso): Al menos una de las acciones falló (por ejemplo, error de conexión con PedidosYa al intentar cerrar una sucursal específica).
*   **Significado:** Indica si el proceso "macro" se ejecutó correctamente. Si es `false`, se debe revisar el detalle en `extraData` para ver cuál sucursal falló.

### 3. `Ident` (Identificador del Proceso)
*   **Que es:** Indica **quién** y **qué proceso** generó este registro.
*   **Valores Posibles y Explicación:**
    *   `"PedidosYa"`, `"Rappi"`, `"Uber"`:
        *   Estos valores indican que el registro fue generado por el **Monitor de Conectividad** (Heartbeat).
        *   El sistema detectó que un local dejó de reportar actividad (se quedó sin internet/sistema) o volvió a tenerla, y actuó en consecuencia.
    *   `"PedidosYaCRON [Pais]"`, `"RappiCRON [Pais]"` (Ej: `"PedidosYaCRON Argentina"`):
        *   Estos valores indican que el registro fue generado por la **Sincronización Programada**.
        *   Es una tarea que corre a horarios fijos para "reforzar" el estado de los locales, asegurando que todos los que deberían estar abiertos lo estén y viceversa.
*   **Significado:** Nos dice si el cierre/apertura fue por una caída de internet del local (Monitor) o por una rutina de mantenimiento de horarios (CRON).

### 4. `platformId`
*   **Que es:** El código numérico interno que representa a la plataforma de delivery.
*   **Valores Comunes:**
    *   `1`: PedidosYa
    *   `2`: Rappi
    *   `4`: Uber Eats
*   **Significado:** Identifica rápidamente hacia qué aplicación se dirigieron las acciones.

### 5. `type` (Tipo de Evento)
*   **Que es:** Clasifica la naturaleza de la operación.
*   **Valores Posibles:**
    *   `"Special"`:
        *   **Contexto:** Monitor de Conectividad ("Zombie Check").
        *   **Funcionamiento:** El sistema revisa cada minuto.
            *   Si el local **no envía señales ("news") en 2 minutos** y figuraba abierto -> El sistema lo **CIERRA** automáticamente (Evita pedidos cancelados).
            *   Si el local **estaba cerrado por el sistema y vuelve a enviar señal** -> El sistema lo **RE-ABRE** automáticamente.
    *   `"CRON"`:
        *   **Contexto:** Sincronización Horaria (Refuerzo de Estado).
        *   **Funcionamiento:** En horarios específicos del día (hora local del país de la sucursal), el sistema fuerza un envío masivo del estado que "debería" tener el local según la base de datos. Esto corrige posibles desincronizaciones donde la plataforma (ej: PedidosYa) crea que el local está cerrado cuando debería estar abierto, o viceversa.
        *   **Horarios y Plataformas Afectadas (Hora Local):**
            *   **PedidosYa:** Se ejecuta múltiples veces durante la mañana, tarde y noche para asegurar la apertura/cierre correcto:
                *   10:01 hs
                *   10:31 hs
                *   11:01 hs
                *   11:31 hs
                *   12:01 hs
                *   17:01 hs
                *   23:01 hs
            *   **Rappi:** Se ejecuta por la mañana y por la tarde:
                *   11:16 hs
                *   17:16 hs
            *   **Uber Eats:** No posee una rutina de sincronización horaria fija ("CRON") en este código, solo funciona por el monitoreo de conectividad ("Special").
*   **Significado:** Es la clave para entender el **"por qué"**. ¿Fue `"Special"`? Entonces el local tuvo problemas de conexión. ¿Fue `"CRON"`? Es una validación de rutina programada a una hora fija.

### 6. `extraData` (Detalle de Ejecución)
*   **Que es:** Una lista detallada con el resultado para cada sucursal involucrada en este proceso.
*   **Estructura Interna:** Cada elemento representa una sucursal:
    *   `branchId`: El ID interno de la sucursal en el sistema.
    *   `results`: Lista de acciones tomadas para esa sucursal (generalmente una por plataforma).
        *   `branchName` / `rappiNumber` / `branchReference`: El ID que usa la plataforma externa (ej: el código del local en PedidosYa).
        *   `type` (Acción Intentada):
            *   `"Open"`: El sistema intentó **ABRIR** el local.
            *   `"Closed"`: El sistema intentó **CERRAR** el local.
        *   `result`:
            *   `true`: La plataforma confirmó el cambio (Ej: "OK, local cerrado").
            *   `false`: Hubo un error al comunicar con la plataforma.

### 7. `createdAt` / `updatedAt`
*   **Que es:** Fecha y hora exacta del registro.
*   **Significado:** Fundamental para cruzar con reclamos de locales. ("*El local se cerró solo a las 14:27*": buscamos un registro con `createdAt` cercano a esa hora y `type: Special` para confirmar que fue por pérdida de conectividad).

---

## Resumen de Casos de Uso

1.  **Local pierde internet (Caso "Zombie"):**
    *   Se genera un registro con `type: "Special"`.
    *   En `extraData`, veremos que para esa sucursal el `type` es `"Closed"`.
    *   **Diagnóstico:** El sistema protegió al local cerrándolo porque dejó de comunicar.

2.  **Local recupera internet:**
    *   Se genera un registro con `type: "Special"`.
    *   En `extraData`, veremos que para esa sucursal el `type` es `"Open"`.
    *   **Diagnóstico:** El sistema detectó la vuelta de conexión y reabrió el local.

3.  **Sincronización Diaria:**
    *   Se genera un registro con `type: "CRON"`.
    *   **Diagnóstico:** Proceso de rutina para alinear estados.

Este documento sirve como guía para interpretar los movimientos automáticos del Concentrador y dar respuesta rápida a consultas sobre la disponibilidad de las tiendas.
