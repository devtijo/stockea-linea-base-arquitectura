# ADR-021 — Autoridad del precio (el servidor no recotiza)

| Campo                   | Valor                                      |
| ----------------------- | ------------------------------------------ |
| Estado                  | **Aprobado**                               |
| Escenarios relacionados | ESC-CAL-CON-0001, ESC-CAL-AUD-0016         |
| Atributos de calidad    | Confiabilidad, Capacidad para ser auditado |
| Relacionados            | ADR-002, ADR-004, ADR-023                  |

## Contexto

En operación offline el cajero cobra el precio que ve en su catálogo local, que puede estar desactualizado respecto del servidor. Ese cobro es un **hecho real**: el cliente ya pagó y se llevó el producto. Si el servidor recotizara con el precio actual al sincronizar, alteraría un hecho consumado y la venta registrada no coincidiría con el dinero en la caja. Al mismo tiempo, aceptar cualquier precio sin ninguna validación abre la puerta a errores de catálogo y a manipulación.

## Alternativas evaluadas

| # | Alternativa | Ventajas | Desventajas |
|---|---|---|---|
| A | Recotizar siempre con el precio actual | Datos "limpios" y consistentes con el catálogo | Contradice que la venta sea un hecho real; descuadra la caja; oculta el problema en vez de mostrarlo |
| B | Rechazar la venta si el precio difiere | Evita datos inconsistentes | Viola ADR-002: rechaza una venta que ya ocurrió |
| C | **Guardar el precio cobrado y validarlo contra el historial** | Respeta el hecho, detecta y audita la divergencia | Requiere una tabla de historial y una consulta por ítem |

## Decisión

**Alternativa C.**

1. El servidor **no recotiza**: persiste en `item_venta.precio_unitario` el **precio cobrado** que viene en la operación.
2. Valida contra `historial_precio` el precio **vigente a `fechaVenta`**: última fila con `vigente_desde <= fechaVenta` (ADR-023).
3. Si difieren, la venta **se acepta** y se crea una `discrepancia` con motivo `PRECIO_DIFERENTE`, guardando `precio_cobrado` y `precio_esperado`. Queda en la cola del supervisor.
4. `historial_precio` es **insert-only** y queda cubierta por la inmutabilidad del ADR-004. Cambiar un precio es insertar una fila nueva, nunca actualizar.
5. `producto.precio_venta` pasa a ser una **proyección** del último registro del historial, útil para listar el catálogo pero sin autoridad para valorar una venta pasada.
6. El mismo criterio aplica al producto inactivo: si el producto fue desactivado después de la venta, la venta se acepta y se levanta una discrepancia `PRODUCTO_INACTIVO`.

## Justificación

Las tres reglas del sistema tienen que apuntar en la misma dirección: la venta es un hecho real (ADR-002), los registros mercantiles son inmutables (ADR-004) y el inventario se corrige con registros nuevos. Recotizar rompería las tres a la vez. Validar sin corregir conserva la coherencia y, además, convierte un problema invisible —catálogos desactualizados en los dispositivos— en un dato medible.

## Consecuencias

- (+) La caja cuadra con lo que el cajero cobró.
- (+) Toda divergencia de precio queda auditada con su valor esperado y su valor cobrado.
- (+) El historial habilita reportes de margen correctos sobre ventas pasadas.
- (−) Una tabla más y una consulta por ítem en el camino de sincronización.
- (−) Riesgo residual: el reloj del dispositivo influye en qué precio histórico se compara. Se acota con `Clock` (ADR-023) y se detecta, no se previene.
- (−) El total de la venta lo propone el cliente; `Venta.registrar` debe verificar que el total recibido coincide con la suma de líneas menos descuentos y, si no, marcar también `PRECIO_DIFERENTE`.

## Spike relacionado

**SPK-007** — Venta offline con precio histórico y reloj desfasado. 1 día.
