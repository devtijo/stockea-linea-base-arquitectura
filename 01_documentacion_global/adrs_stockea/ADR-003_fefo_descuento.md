# ADR-003 — Descuento de stock por lote (FEFO por vencimiento)

| Campo                   | Valor                              |
| ----------------------- | ---------------------------------- |
| Estado                  | **Aprobado**                       |
| Escenarios relacionados | ESC-CAL-CON-0014, ESC-CAL-CON-0025 |
| Atributos de calidad    | Confiabilidad                      |

### Contexto

Un mismo producto puede tener múltiples lotes activos con fechas de vencimiento distintas. Al vender, el sistema debe decidir de cuál lote descontar. La decisión impacta las alertas de vencimiento, los descuentos y la trazabilidad.

### Decisión

**FEFO por fecha de vencimiento (primero el que vence antes).**

### Justificación

STOCKEA está diseñado para perecederos. Vender primero lo que vence antes es la práctica estándar del sector y reduce la merma. La complejidad del fraccionamiento entre lotes es manejable y se justifica por esa reducción de merma. FIFO por fecha de ingreso no optimiza para perecederos; la selección manual agrega fricción en cada venta.

### Tácticas arquitectónicas

- Consulta ordenada por `fechaVencimiento ASC` sobre lotes `DISPONIBLE`/`PROXIMO_A_VENCER`.
- **Fraccionamiento automático**: si la cantidad excede el primer lote, se generan múltiples `ItemVenta`, cada uno referenciando su lote.
- Validación de stock total antes de confirmar.
- El descuento por lote se aplica solo a las unidades del lote que originó el descuento.

### Consecuencias

- `ItemVenta` siempre referencia exactamente un lote.
- Una venta puede generar varios `ItemVenta` si hay fraccionamiento.
- El comprobante muestra un solo producto al cliente; el detalle por lote es para auditoría.
- **La reasignación FEFO del ADR-002 reutiliza exactamente este algoritmo.**

**Enmienda (2026-10-28) — Modelo de la venta fraccionada.** El texto original indica que el fraccionamiento FEFO genera “múltiples `ItemVenta`, cada uno referenciando su lote” y que “`ItemVenta` siempre referencia exactamente un lote”. El modelo de dominio final lo reemplaza: cada producto vendido es **un solo `ItemVenta`** (cantidad, precio y descuento) con **una o más `AsignacionLote` hijas** (`lote_id`, `cantidad`) que registran de qué lote salió cada fracción. Se cambia porque con varias líneas el precio y el descuento se duplican o hay que prorratearlos, y el comprobante mostraría el mismo producto repetido. El algoritmo FEFO no cambia, solo dónde se registra su resultado. Las tácticas y consecuencias que hablan de varios `ItemVenta` por fraccionamiento quedan sustituidas por esta enmienda.

### Enmienda (2026-10-28) — Descuento a nivel de `AsignacionLote`

El descuento se calcula y guarda **por asignación** (`asignacion_lote.descuento_id`, `monto_descuento`), porque un descuento por cercanía de vencimiento pertenece al lote, no al producto:

```
item_venta.descuento_total_linea = SUM(asignacion_lote.monto_descuento) + item_venta.descuento_sin_cobertura
```

`descuento_sin_cobertura` (`NUMERIC(14,2)`, `DEFAULT 0`) es el descuento de las unidades que **no encontraron lote** (`cantidad_sin_cobertura`). Sin este campo el total de línea no cuadraría cuando hay discrepancia de stock.

**Estructura.** `InventarioApiImpl` consulta y bloquea lotes; `AsignadorFefo` es **dominio puro** (`asignar(lotes, cantidad)`, sin acceso a base de datos); `DescuentosApi` calcula los montos por lote.

### Enmienda (2026-10-28) — Tácticas desactualizadas

- La consulta es ordenada por `(producto_id, fecha_vencimiento)` con `FOR UPDATE` sobre lotes en estado `ACTIVO` con `cantidad_disponible > 0` y `fecha_vencimiento >= fechaVenta`. `PROXIMO_A_VENCER` no es un estado persistido: es una vista derivada de la fecha de vencimiento (ADR-006).
- La viñeta "Validación de stock total antes de confirmar" queda sustituida por **cálculo de cobertura**: lo que ningún lote cubre se registra como `cantidad_sin_cobertura` y genera discrepancia (ADR-002), nunca un rechazo.
- Las viñetas sobre "múltiples `ItemVenta`" siguen sustituidas por la enmienda anterior; se conservan como registro histórico.
