# ADR-002 — Estrategia de resolución de conflictos en sincronización

| Campo                   | Valor                              |
| ----------------------- | ---------------------------------- |
| Estado                  | **Aprobado (reescrito)**           |
| Escenarios relacionados | ESC-CAL-DIS-0008, ESC-CAL-REN-0011 |
| Atributos de calidad    | Confiabilidad, Disponibilidad      |

### Contexto

Cuando dos o más dispositivos operan offline y venden del mismo lote, al sincronizar puede haber sobreventa. **Una venta es un hecho del mundo real**: el cliente ya se llevó el producto y pagó. Rechazarla descuadra la caja del cajero y del negocio y rompe la confianza en el sistema. El conflicto, por tanto, no es de venta sino de inventario.

### Alternativas evaluadas

|#|Alternativa|Ventajas|Desventajas|
|---|---|---|---|
|A|First-write-wins: la primera venta se acepta, la segunda se marca CONFLICTO y se rechaza|Simple|**Rechaza una venta real** y descuadra la caja|
|B|**Venta siempre aceptada + reasignación FEFO en el servidor + discrepancia de inventario visible**|**No se pierde ninguna venta; el inventario se corrige con trazabilidad**|Requiere modelo de discrepancias y pantalla de resolución|
|C|Reserva de stock por dispositivo|Elimina conflictos de raíz|Subutiliza inventario y no escala con más de 2 dispositivos|

### Decisión

**Alternativa B: la venta siempre se acepta; el conflicto es de inventario.**

### Justificación

Rechazar una venta que ya ocurrió es incorrecto desde el negocio. El servidor, al sincronizar, **reasigna lotes con FEFO** (First Expired, First Out, el mismo algoritmo del ADR-003). Si un lote ya no alcanza, se toma el siguiente y la operación se marca **"reasignada"**. Si ningún lote alcanza, se registra una **discrepancia de inventario** (stock en revisión) que el supervisor resuelve con un ajuste `CORRECCION`. El stock negativo es un **estado transitorio visible**, no un rechazo de la venta.

### Tácticas arquitectónicas

- `operacion_id` (UUID v7) generado en el cliente, para idempotencia.
- **Número de secuencia por dispositivo** para ordenar operaciones sin depender del reloj local.
- **Reasignación FEFO** en el servidor al sincronizar.
- Estados por operación: `ACEPTADA`, `REASIGNADA`, `DISCREPANCIA`, `RECHAZADA_POR_PERMISOS`.
- **Respuesta por operación**, nunca "todo o nada" en un lote.
- **Cola de discrepancias** visible al supervisor, con flujo de resolución vía `AjusteInventario(CORRECCION)`.

### Spikes

|Spike|Objetivo|Criterio de éxito|Duración|
|---|---|---|---|
|SPK-002|Dos dispositivos vendiendo del mismo lote offline|Ambas ventas aceptadas; el servidor reasigna FEFO; si no alcanza, genera discrepancia visible; stock final nunca negativo|2 días|
|SPK-002b|10 dispositivos offline vendiendo del mismo lote con solapamiento total|Ninguna venta se pierde; las discrepancias se generan y las resuelve el supervisor en ≤ 1 día|1 día|

### Consecuencias

- El inventario puede quedar **transitoriamente inconsistente**; se compensa con la cola de discrepancias.
- Se reescribe **RF-011.3** y **ESC-CAL-DIS-0008** con la nueva medida: **"0 ventas perdidas; discrepancias resueltas en ≤ 1 día"** (ya no "0 stock negativo").
- Se requiere pantalla de resolución de discrepancias en el frontend del supervisor.

**Enmienda (2026-10-28) — Materialización de “el stock nunca es negativo”.** El texto original describe el stock negativo como “estado transitorio visible”. El diseño final lo evita en la base de datos: `lote.cantidad_disponible` tiene `CHECK (cantidad_disponible >= 0)`, y la parte de la venta que ningún lote cubre se guarda en `item_venta.cantidad_sin_cobertura` y en una fila de `discrepancia` (`cantidad_faltante`, estado `ABIERTA`). Así la venta se acepta siempre y el inventario nunca queda con saldo negativo. El faltante se corrige con un movimiento de tipo `CORRECCION`. Se mantiene la invariante `SUM(asignacion_lote.cantidad) + cantidad_sin_cobertura = cantidad` por línea de venta.

### Enmienda (2026-10-28) — Idempotencia, concurrencia y casos de borde

**Idempotencia.** `operacion_sync` tiene **PK compuesta `(negocio_id, operacion_id)`** (el `operacion_id` es un UUID v7 generado en el cliente). Antes de procesar, cada operación toma un lock de transacción:

```sql
SELECT pg_advisory_xact_lock(hashtextextended(negocio_id::text || operacion_id::text, 0));
```

El hash es **numérico de 64 bits** (el advisory lock opera sobre `bigint`, no sobre texto). El lock serializa dos reintentos concurrentes del mismo `operacion_id` y se libera solo al `COMMIT` o `ROLLBACK`; la PK compuesta es la red de seguridad final.

**Aislamiento.** Se mantiene **READ COMMITTED**. No se sube a SERIALIZABLE porque la corrección no depende de él: (1) el advisory lock serializa por operación, (2) `SELECT ... FOR UPDATE` sobre lotes en orden `(producto_id, fecha_vencimiento)` serializa el stock y evita interbloqueos (los ítems se ordenan por `productoId` antes del bucle FEFO), (3) `CHECK (cantidad_disponible >= 0)` es la última barrera.

**Casos de borde** (la venta ya ocurrió físicamente):

| Caso | Resultado | Estado `operacion_sync` | Discrepancia |
|---|---|---|---|
| Stock insuficiente | Venta aceptada, `cantidad_sin_cobertura > 0` | `DISCREPANCIA` | `STOCK_INSUFICIENTE` |
| Producto inactivo | Venta aceptada | `DISCREPANCIA` | `PRODUCTO_INACTIVO` |
| Precio cobrado ≠ vigente | Venta aceptada con el precio cobrado | `DISCREPANCIA` | `PRECIO_DIFERENTE` (ADR-021) |
| Turno cerrado a `fechaCliente` | **No se crea venta**, se registra el rechazo | `RECHAZADA_SIN_TURNO` | — |

El turno cerrado es la única excepción a "la venta siempre se acepta": sin turno no hay caja a la que imputar el dinero. El rechazo queda registrado, nunca se pierde en silencio. El turno se valida contra `fechaCliente`, no contra la hora de sincronización.

**Fecha de negocio.** El filtro de vencimiento usa `fechaVenta`, nunca `CURRENT_DATE` (ver ADR-023).

### Enmienda (2026-10-28) — Corrección de ajuste de inventario

Donde las tácticas dicen "flujo de resolución vía `AjusteInventario(CORRECCION)`", el modelo final usa un **`MovimientoInventario` de tipo `CORRECCION`** (`movimiento_inventario.tipo = CORRECCION`), append-only, enlazado a la discrepancia mediante `discrepancia.movimiento_correccion_id`.
