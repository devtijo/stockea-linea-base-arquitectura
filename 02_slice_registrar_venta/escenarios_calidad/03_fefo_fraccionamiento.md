# ESC-CAL-CON-0014 — Descuento FEFO con fraccionamiento entre lotes

Cuando se confirme una venta de 3 unidades de un producto con stock de 10, el sistema debe mostrar inmediatamente un stock de 7 sin recargar la página, descontando del lote que vence primero.

| Campo | Valor |
|---|---|
| Atributo de calidad | Confiabilidad |
| Prioridad | Crítica |
| Dificultad / Riesgo | Media |
| RF relacionado | RF-004.1 |

## Definición del escenario

| # | Parte | Descripción |
|---|---|---|
| 1 | **Origen** | Cajero que confirma el pago de una venta desde el punto de venta. |
| 2 | **Estímulo** | Se confirma una venta de 3 unidades de un producto con 10 disponibles, repartidas entre un lote con vencimiento próximo (7 unidades) y otro más lejano (3 unidades). |
| 3 | **Artefacto** | El servicio de dominio de ventas que descuenta stock por lote siguiendo la regla de vencimiento más próximo primero, y la actualización del stock calculado del producto. |
| 4 | **Entorno** | Operación normal del punto de venta en modo online. |
| 5 | **Respuesta** | El sistema descuenta del lote con vencimiento más próximo, deja intacto el otro y actualiza el stock total del producto de forma inmediata en la interfaz. |
| 6 | **Medida de respuesta** | 100% de ventas descontadas del lote correcto según FEFO; stock reflejado en la interfaz sin recargar. |

## Nota de arquitectura

El algoritmo vive en `AsignadorFefo`, un **servicio de dominio puro**: recibe la lista de lotes ya consultada y la cantidad, y devuelve el reparto sin tocar la base de datos (ADR-003, ADR-009). Quien consulta y bloquea es `InventarioApiImpl`, con `SELECT ... ORDER BY producto_id, fecha_vencimiento FOR UPDATE`, un orden de bloqueo global que evita interbloqueos entre sincronizaciones concurrentes. El resultado se registra como **un `ItemVenta` con varias `AsignacionLote`**, no como varias líneas: el cliente compró un producto a un precio, y de qué lote salió cada fracción es un detalle de inventario. El filtro de vencimiento usa `fechaVenta`, no `CURRENT_DATE` (ADR-023).
