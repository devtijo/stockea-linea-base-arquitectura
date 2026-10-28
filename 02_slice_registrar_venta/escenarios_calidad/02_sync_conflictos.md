# ESC-CAL-DIS-0008 — Dos dispositivos venden del mismo lote

Cuando dos dispositivos que operaron offline vendan del mismo lote, el sistema debe aceptar ambas ventas y registrar una discrepancia de inventario visible si el stock no alcanza.

| Campo | Valor |
|---|---|
| Atributo de calidad | Disponibilidad |
| Prioridad | Alta |
| Dificultad / Riesgo | Muy alta |
| RF relacionado | RF-011.3 |

## Definición del escenario

| # | Parte | Descripción |
|---|---|---|
| 1 | **Origen** | Dos dispositivos del mismo negocio que operaron offline simultáneamente y vendieron del mismo lote. |
| 2 | **Estímulo** | El dispositivo A sincroniza una venta de 8 unidades del lote X (que tenía 10 disponibles). Minutos después, el dispositivo B sincroniza una venta de 5 unidades del mismo lote, que ya solo tiene 2 disponibles en el servidor. |
| 3 | **Artefacto** | El servicio de sincronización del backend, que procesa cada operación contra el estado actual del inventario. |
| 4 | **Entorno** | Fase de sincronización posterior a un período de operación offline en dos dispositivos del mismo negocio. |
| 5 | **Respuesta** | Ambas ventas se aceptan, porque son hechos reales. La de A queda `ACEPTADA`. La de B se cubre con FEFO: 2 unidades del lote X y el resto del siguiente lote; lo que ningún lote alcance se registra como `cantidad_sin_cobertura` y la operación queda en `DISCREPANCIA`, visible al supervisor, que la resuelve con un movimiento `CORRECCION`. |
| 6 | **Medida de respuesta** | 0 ventas perdidas, 100% de discrepancias detectadas y notificadas al supervisor, resueltas en ≤ 1 día, con respuesta por operación (nunca "todo o nada" en el lote sincronizado). |

## Nota de arquitectura

La decisión de fondo es que **el conflicto es de inventario, no de venta** (ADR-002): rechazar una venta ya ocurrida descuadra la caja del cajero. El stock, sin embargo, nunca queda negativo: `lote.cantidad_disponible` tiene `CHECK >= 0` y lo que falta se guarda en `item_venta.cantidad_sin_cobertura` más una fila de `discrepancia`. La idempotencia la garantizan la PK compuesta `(negocio_id, operacion_id)` y un `pg_advisory_xact_lock` por operación, de modo que un reenvío devuelve el mismo `ventaId` sin duplicar nada. El orden entre operaciones de un mismo dispositivo lo da `secuenciaDispositivo`, no el reloj local.
