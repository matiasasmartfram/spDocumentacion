---
layout: default
title: OpenClose - Registro de Aperturas y Cierres
subtitle: Documentación del registro opencloselogs generado por el Monitor de Conectividad y el Cron de Sincronización (checkIns)
---

> **Actualización:** los horarios de sincronización ya no están fijos ("hardcodeados") en el código. Ahora se configuran desde el **Backoffice** y se guardan como datos en **MongoDB** (colección `platforms`, mediante el mecanismo de **checkIns**). El comportamiento de negocio es el mismo (reforzar el estado abierto/cerrado a horas determinadas), pero **los horarios y las plataformas afectadas ahora son dinámicos y editables sin tocar código**.

---

## 1. Introducción

El Concentrador es el responsable de **abrir o cerrar automáticamente las tiendas en las plataformas de delivery** (PedidosYa, Rappi, Uber Eats, PediGrido y MercadoPago) para que un local solo esté disponible en la app cuando realmente puede operar.

Toda esta actividad queda registrada en la colección **`openClosedLog`** (`opencloselogs` en la base). Este registro permite monitorear **cómo y por qué** el sistema intervino en la disponibilidad de una sucursal.

Existen **dos motores** que generan aperturas/cierres, y por lo tanto dos grandes orígenes de registros:

| Motor | ¿Cuándo actúa? | ¿Por qué? | `type` del log |
|---|---|---|---|
| **Monitor de Conectividad** ("Zombie Check" / Heartbeat) | Cada 1 minuto | El POS del local dejó de comunicarse (se cayó internet/sistema) o volvió a comunicarse | `"Special"` |
| **Sincronización por Horarios (CheckIns)** | A las horas configuradas en el Backoffice | Rutina de refuerzo: reafirma a la plataforma el estado que el local *debería* tener | `"CRON"` |

La diferencia clave para diagnosticar:
- ¿Fue `"Special"`? → el local tuvo un **problema de conectividad**.
- ¿Fue `"CRON"`? → fue una **validación de rutina programada** a una hora configurada.

---

## 2. Colecciones y documentos que intervienen

| Colección | Campos relevantes | Rol |
|---|---|---|
| `platforms` | `internalCode`, `checkIns`, `checkInActive`, `alreadySincro` | Guarda la **configuración de horarios (checkIns)** de cada plataforma. Es lo que reemplaza a los horarios hardcodeados. |
| `branches` | `lastGetNews`, `alreadyClosedRappi`, `alreadyClosedPeya`, `alreadyClosedUber`, `alreadyClosedPediGrido`, `alreadyClosedMercadoPago`, `platforms[]` | Estado real de cada sucursal: última señal del POS y flag de "ya cerrado/abierto" por plataforma. |
| `countries` | `country`, `tzo` (timezone offset) | Permite convertir la hora local del checkIn a UTC por país. |
| `openClosedLog` | `result`, `Ident`, `platformId`, `type`, `extraData`, `createdAt` | Registro de auditoría de cada corrida de apertura/cierre. |

### Códigos internos de plataforma (`internalCode` / `platformId`)

| Código | Plataforma |
|---|---|
| `1` | PedidosYa |
| `2` | Rappi |
| `4` | Uber Eats |
| `8` | MercadoPago |
| *(sin `internalCode` de checkIn)* | PediGrido |

---

## 3. Motor 1 — Monitor de Conectividad (`type: "Special"`)

Es el mecanismo de protección "en vivo". Corre **cada 1 minuto** (un cron por plataforma) y decide en base a la **última señal del POS** (`lastGetNews` en el documento `branch`).

### Funciones que lo implementan (en `controllers/branch.js`)

| Función | Plataforma | `Ident` del log |
|---|---|---|
| `checkOfflinePOSpeya()` | PedidosYa | `"PedidosYa"` |
| `checkOfflinePOSrappi()` | Rappi | `"Rappi"` |
| `checkOfflinePOSuber()` | Uber Eats | `"Uber"` |
| `checkOfflinePOSpedigrido()` | PediGrido | `"PediGrido"` |
| `checkOfflinePOSmercadopago()` | MercadoPago | `"MercadoPago"` |

### Cómo funciona

1. Toma la hora actual y calcula un umbral (`now - 1 minuto`).
   > Nota técnica: la variable se llama `date3minutes` pero el umbral vigente es de **1 minuto**.
2. Busca sucursales de esa plataforma que estén activas y particiona:
   - **A CERRAR**: `lastGetNews` es más viejo que el umbral **y** todavía figuraban abiertas (`alreadyClosed*` en `null`/`false`). → El POS no reporta actividad ⇒ el sistema **cierra** el local en la plataforma.
   - **A ABRIR**: `lastGetNews` es reciente **y** figuraban cerradas por el sistema (`alreadyClosed*` en `true`). → El POS volvió a la vida ⇒ el sistema **reabre** el local.
3. Envía el comando OPEN/CLOSE a la API de la plataforma y **actualiza el flag `alreadyClosed<Plataforma>`** en el documento `branch` (`true` = cerrado por el sistema, `false` = abierto).
4. Escribe un registro en `openClosedLog` con `type: "Special"`.

> **El flag `alreadyClosed*` es el "estado deseado" del local.** Lo setea este monitor y es la fuente de verdad que después usa el Motor 2 para reforzar.

---

## 4. Motor 2 — Sincronización por Horarios / CheckIns (`type: "CRON"`)

Es el refuerzo programado. A las horas configuradas, **re-empuja a la plataforma el estado que el local debería tener** según el flag `alreadyClosed*`. Corrige desincronizaciones (ej.: PedidosYa cree que el local está cerrado cuando debería estar abierto).

### 4.1. Configuración dinámica (ya no hardcodeada)

- **Antes:** los horarios estaban fijos en el código (PedidosYa 10:01, 10:31, 11:01…; Rappi 11:16, 17:16; Uber sin CRON).
- **Ahora:** cada plataforma tiene un arreglo **`checkIns`** en su documento de la colección `platforms`, editable desde el Backoffice. Cada ítem es:

  ```json
  { "hour": 10, "minute": 1, "active": true }
  ```

  (hora **local**). El campo **`checkInActive`** (boolean) es el interruptor maestro de la plataforma. Como consecuencia, **cualquier plataforma soportada puede tener o no CRON según su configuración** — incluyendo Uber Eats y MercadoPago.

### 4.2. Configuración desde el Backoffice (API)

En `controllers/thirdParty.js` / `routes/thirdParty.js`:

| Endpoint | Función | Qué hace |
|---|---|---|
| `GET /thirdParty/getPlatformsCheckIns/:id` | `getPlatformsCheckIns` | Devuelve `name`, `checkIns` y `checkInActive` de la plataforma (`:id` = `internalCode`). |
| `POST /thirdParty/savePlatformsCheckIns` | `savePlatformsCheckIns` | Recibe `{ platformId, checkInActive, checkIns }`, los guarda en `platforms` **y marca `alreadySincro: false`** para que el motor regenere los horarios. |

### 4.3. Cómo se activan los horarios (crons dinámicos)

En `controllers/branch.js`:

1. **`sincroCheckIns(platform)`** — el corazón del motor. Para una plataforma:
   - Destruye los crons anteriores de esa plataforma (registro en memoria `cronsCheckIns[internalCode]`).
   - Trae **todos los países** (`countries`) y filtra los checkIns con `active === true`.
   - Por cada **país × checkIn**, convierte la hora local a **UTC** según el `tzo` del país (`toUTC(...)`) y programa un cron diario.
   - Al dispararse, según el `internalCode` de la plataforma llama a la función de estado correspondiente **pasándole el país**:
     - `1` → `enviarEstadoPedidosYa(country)`
     - `2` → `enviarEstadoRappi(country)`
     - `4` → `sendStatusUberEats(country)`
     - `8` → `sendStateMercadoPago(country)`
   > Así, un mismo horario local dispara a la hora correcta en cada país aunque internamente el cron corra en UTC.

2. **Arranque del servicio:** al iniciar, busca todas las plataformas con `checkInActive: true` y ejecuta `sincroCheckIns` para cada una (crea los crons iniciales).

3. **`checkforcedCheckins()`** — watchdog que corre **cada 15 segundos**. Busca plataformas con `checkInActive: true` **y** `alreadySincro: false` (es decir, con cambios guardados desde el Backoffice), las marca como sincronizadas y llama a `sincroCheckIns` para **regenerar los horarios en caliente**, sin reiniciar el servicio.

**Flujo completo de un cambio de horario:**
`Backoffice guarda` → `savePlatformsCheckIns` deja `alreadySincro:false` en Mongo → `checkforcedCheckins` (≤15s) lo detecta → `sincroCheckIns` reconstruye los crons con los nuevos horarios.

### 4.4. Qué hacen las funciones de estado (`enviarEstado*` / `sendStatus*` / `sendState*`)

Todas siguen el mismo patrón: buscan las sucursales del país + plataforma y las particionan **según el flag `alreadyClosed*`** (no según conectividad):
- Las marcadas como abiertas (`alreadyClosed* = false`) → reenvían **OPEN** a la plataforma.
- Las marcadas como cerradas (`alreadyClosed* = true`) → reenvían **CLOSE**.
- **No cambian el flag**; solo reafirman el estado ya decidido por el Motor 1.
- Registran el resultado en `openClosedLog` con `type: "CRON"` e `Ident` con el país (ej.: `"PedidosYaCRON Argentina"`).

---

## 5. Anatomía del registro `openClosedLog`

Estructura del documento (modelo `models/openClosedLog.js`):

| Campo | Qué es | Valores / significado |
|---|---|---|
| `_id` | Identificador único del registro | Generado automáticamente. Referencia técnica del evento. |
| `result` | Resultado general del lote | `true` = todas las acciones del lote fueron OK. `false` = al menos una falló → revisar `extraData`. |
| `Ident` | Quién/qué proceso generó el registro | Ver tabla de la sección 6. Distingue Monitor (`"Rappi"`) de CRON (`"RappiCRON Argentina"`). |
| `platformId` | Código numérico de la plataforma | `1` PeYa, `2` Rappi, `4` Uber, `8` MercadoPago. |
| `type` | Naturaleza de la operación | `"Special"` (conectividad), `"CRON"` (horario), `"ClosedAfterClosed"` (ver 6.3). |
| `extraData` | Detalle por sucursal | Lista de sucursales con el resultado de cada acción (ver abajo). |
| `createdAt` / `updatedAt` | Fecha y hora del registro | Clave para cruzar con reclamos ("el local se cerró solo a las 14:27"). |

### Estructura de `extraData`

Cada elemento representa una sucursal:
- `branchId`: ID interno de la sucursal en el sistema.
- `results`: acciones tomadas para esa sucursal. Cada resultado incluye:
  - `branchName` / `rappiNumber` / `branchReference`: el ID que usa la plataforma externa.
  - `type`: acción intentada — `"Open"` (abrir) o `"Closed"` (cerrar).
  - `result`: `true` (la plataforma confirmó el cambio) / `false` (error de comunicación).
  - `resultApi` / `resultStatus`: respuesta y código HTTP devuelto por la plataforma.

---

## 6. Tabla de referencia de `Ident` y `type`

### 6.1. Monitor de Conectividad → `type: "Special"`

| `Ident` | Plataforma | Generado por |
|---|---|---|
| `"PedidosYa"` | PedidosYa | `checkOfflinePOSpeya` |
| `"Rappi"` | Rappi | `checkOfflinePOSrappi` |
| `"Uber"` | Uber Eats | `checkOfflinePOSuber` |
| `"PediGrido"` | PediGrido | `checkOfflinePOSpedigrido` |
| `"MercadoPago"` | MercadoPago | `checkOfflinePOSmercadopago` |

### 6.2. Sincronización por CheckIns → `type: "CRON"`

| `Ident` (con país) | Plataforma | Generado por |
|---|---|---|
| `"PedidosYaCRON <País>"` | PedidosYa | `enviarEstadoPedidosYa` |
| `"RappiCRON <País>"` | Rappi | `enviarEstadoRappi` |
| `"UberEatsCron <País>"` | Uber Eats | `sendStatusUberEats` |
| `"MercadoPagoCron"` | MercadoPago | `sendStateMercadoPago` |
| `"PediGrido"` | PediGrido | (rutina horaria propia de Grido) |

### 6.3. Otros orígenes

- **`type: "ClosedAfterClosed"`** (`Ident` = nombre de la plataforma): se genera cuando llega/se rechaza un pedido a un local que estaba cerrado y el sistema **vuelve a mandar el cierre** para reforzar. Es parte del flujo de rechazos por local cerrado (`sendClosedToPlatformsAfterRejectClosed` / `orderRejClosed`), no del open/close estándar.

---

## 7. Casos de uso / diagnóstico rápido

| Situación | Cómo se ve en `openClosedLog` | Diagnóstico |
|---|---|---|
| **Local pierde internet** (caso "Zombie") | `type: "Special"`; en `extraData` esa sucursal con `type: "Closed"` | El sistema protegió al local cerrándolo porque dejó de comunicar. |
| **Local recupera internet** | `type: "Special"`; en `extraData` esa sucursal con `type: "Open"` | El sistema detectó la vuelta de conexión y reabrió el local. |
| **Sincronización diaria** | `type: "CRON"` a una hora configurada | Rutina de refuerzo para alinear estados con la plataforma. |
| **Reclamo "se cerró solo a las HH:MM"** | Buscar `createdAt` cercano y ver `type` | `Special` = fue conectividad; `CRON` = fue horario programado. |

---

## 8. Notas y consideraciones

- Los horarios concretos (10:01, 11:16, etc.) ya **no son autoritativos**: son datos en `platforms.checkIns` y pueden variar por plataforma según lo que se cargue en el Backoffice.
- El umbral del Monitor de Conectividad vigente es de **1 minuto** desde `lastGetNews` (aunque la variable interna se llame `date3minutes`).
- El campo `alreadySincro` es puramente interno: **no lo edita el usuario**; lo pone en `false` el guardado del Backoffice y en `true` el watchdog cuando ya regeneró los crons.
- La conversión de hora local a UTC depende del `tzo` del documento `country`; una carga incorrecta de `tzo` desplaza el horario real de ejecución.
