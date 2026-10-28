# ADR-001 — Estrategia de operación offline

| Campo                   | Valor                                                                  |
| ----------------------- | ---------------------------------------------------------------------- |
| Estado                  | **Aprobado**                                                           |
| Escenarios relacionados | ESC-CAL-DIS-0001, ESC-CAL-DIS-0005, ESC-CAL-CON-0001, ESC-CAL-SEG-0004 |
| Atributos de calidad    | Disponibilidad, Confiabilidad, Seguridad                               |

### Contexto

STOCKEA debe operar sin internet en tiendas de barrio con conectividad intermitente. El cajero no puede dejar de vender porque se cayó el WiFi. La operación offline exige además que el cajero pueda **autenticarse sin conexión** y que la sesión sobreviva a cortes prolongados. Un proveedor de identidad externo requiere conexión; por eso se necesita un mecanismo de verificación local de credenciales en el dispositivo. En el MVP la autenticación la provee el propio Backend detrás de `AuthPort` (ver ADR-010); un IDP dedicado (Keycloak) se difiere a Fase 2 (ver ADR-012).

### Alternativas evaluadas

|#|Alternativa|Ventajas|Desventajas|
|---|---|---|---|
|A|**Offline-first con Service Worker + IndexedDB (Dexie.js) + Workbox**|El frontend siempre opera contra datos locales; experiencia idéntica online y offline. Workbox gestiona caché y BackgroundSync; Dexie simplifica IndexedDB.|Complejidad alta en la primera implementación. Requiere gestionar conflictos de sincronización.|
|B|Online-first con caché de respaldo (localStorage)|Implementación simple.|localStorage tiene límite de ~5 MB, es síncrono, bloquea el hilo principal y no soporta consultas indexadas. Experiencia degradada offline.|
|C|Aplicación de escritorio con SQLite embebido|Base de datos local robusta con SQL completo.|Requiere instalador y distribución manual. Contradice la decisión de PWA y duplica el esfuerzo.|

### Decisión

**Alternativa A: Offline-first con Service Worker + IndexedDB (Dexie.js) + Workbox**, con autenticación offline provista por el Backend.

**Autenticación offline:**

- El primer login del dispositivo se hace **online contra el Backend** (endpoint `/auth/login` detrás de `AuthPort`).
- El Backend entrega al dispositivo un **material de verificación local**: hash con sal + iteraciones (Argon2 o PBKDF2) del PIN del cajero, junto con `negocio_id`, `usuario_id` y `rol`.
- La sesión offline se mantiene con un **token de dispositivo de larga duración** (8 h) firmado por el Backend y verificado localmente.
- La inactividad de 15 min (ESC-CAL-SEG-0004) se evalúa **localmente**, sin depender del servidor.
- Al reconectar, el dispositivo refresca el token. Si el refresh expiró, se re-autentica **sin perder la cola de operaciones** (cada operación lleva firma del dispositivo + `usuario_id`; el servidor las valida contra el estado actual del usuario).

### Justificación

La alternativa A es la única que cumple simultáneamente con la operación offline transparente (ESC-CAL-DIS-0001) y con la restricción de ser PWA instalable sin tienda de apps. B falla por las limitaciones de localStorage; C contradice la restricción de canal web exclusivo y multiplica el esfuerzo de un equipo de 3 personas.

### Tácticas arquitectónicas

- **Cache-First** (Workbox) para assets estáticos.
- **StaleWhileRevalidate** (Workbox) para datos maestros (productos, lotes, precios).
- **BackgroundSync** (Workbox) para operaciones de escritura; fallback a reintento con backoff desde la app donde el navegador no lo soporte.
- **Esquema local con Dexie.js** reflejando las entidades del dominio.
- `navigator.storage.persist()` para evitar desalojo del almacenamiento por el navegador.
- `durability: 'strict'` en transacciones Dexie críticas (ESC-CAL-CON-0001).
- Migraciones de esquema versionadas en Dexie.
- **CSP estricta** para mitigar XSS (tokens en memoria cuando sea posible).
- Presupuesto de memoria y límite de datos locales con limpieza periódica (ESC-CAL-DIS-0033).

### Spikes

|Spike|Objetivo|Criterio de éxito|Duración|
|---|---|---|---|
|SPK-001|Venta completa offline con Dexie + Workbox y sincronización al reconectar|Flujo completo offline (abrir caja → agregar producto → confirmar venta → cerrar caja) y sincronización correcta al reconectar|3 días|
|SPK-001b|Login offline con PIN local, sesión de 8 h, inactividad 15 min, refresh sin perder cola|El cajero apaga el equipo, lo enciende sin internet, entra con PIN y sigue vendiendo; al reconectar, la cola se sincroniza|2 días|

### Consecuencias

- Se acepta la complejidad de gestionar dos fuentes de datos (IndexedDB local y PostgreSQL remoto).
- El hash local en el navegador es un riesgo si el equipo se roba; se mitiga con caducidad corta del token y revocación remota al sincronizar.
- El Backend (AuthPort) es necesario para el primer login online; el offline solo cubre sesiones ya establecidas.
- Si en Fase 2 se activa Keycloak, el endpoint `/auth/login` se redirige al IDP sin cambiar el contrato del frontend.

### Enmienda (2026-10-28) — Un solo endpoint para online y offline

**Contexto.** El diseño original describía el Sync Controller solo para lotes offline y un `POST /api/ventas` para el flujo conectado. Mantener dos caminos duplicaba la lógica de idempotencia y de FEFO.

**Decisión.** El frontend es *offline-first siempre*: toda venta se escribe primero en Dexie (venta + outbox + stock local en **una sola transacción**) y la **`SyncOutbox` es el único componente que habla con el backend**. Online y offline usan el mismo endpoint `POST /api/sync/operaciones` (lote de 1..N operaciones). **`POST /api/ventas` no existe.** Online la outbox drena de inmediato; offline espera el evento `online`.

**Consecuencias.** (+) Un único camino de servidor: idempotencia, FEFO, validación de precio y discrepancias se escriben y se prueban una vez. (+) El cajero nunca espera al servidor para cobrar. (−) Un cliente online también recibe el resultado de forma asíncrona (comprobante "sincronizando" → "confirmado"). (−) Endpoint por lotes más genérico que un `POST /ventas` REST clásico.

### Enmienda (2026-10-28) — Alcance de BackgroundSync

`BackgroundSync` (Workbox) se usa como disparador **oportunista** del drenaje de la outbox. La fuente de verdad del reenvío es la propia outbox en IndexedDB, el evento `online` y el reintento con backoff definido en el ADR-022; BackgroundSync no está disponible en todos los navegadores y nunca es la única garantía.
