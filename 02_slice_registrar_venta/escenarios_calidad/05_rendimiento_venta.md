# ESC-CAL-REN-0021 — Venta de 10 productos en menos de 3 segundos

Al registrar una venta con hasta 10 productos, el sistema debe completar la operación en un tiempo máximo de 3 segundos.

| Campo | Valor |
|---|---|
| Atributo de calidad | Rendimiento |
| Prioridad | Crítica |
| Dificultad / Riesgo | Media |
| RF relacionado | RF-004.1 |

## Definición del escenario

| # | Parte | Descripción |
|---|---|---|
| 1 | **Origen** | Cajero que confirma el pago de una venta con múltiples productos durante el horario comercial. |
| 2 | **Estímulo** | El cajero confirma el pago de una venta con 10 productos distintos, cantidades variadas y algunos con descuento activo. |
| 3 | **Artefacto** | El flujo completo de procesamiento de la venta: validación de stock por lote, cálculo de descuentos, actualización de lotes y stock, y generación del comprobante. |
| 4 | **Entorno** | Operación normal del punto de venta con el cliente esperando frente al cajero. El sistema puede estar en modo online u offline. |
| 5 | **Respuesta** | La venta se registra completamente, el inventario se actualiza, el comprobante se genera y la interfaz confirma el éxito al cajero. |
| 6 | **Medida de respuesta** | Tiempo total desde confirmar hasta ver la confirmación ≤ 3 segundos, igual en modo online que offline. |

## Nota de arquitectura

El tiempo percibido no depende de la red, y esa es la decisión clave: la venta se confirma contra **IndexedDB** y el comprobante se genera en el cliente, así que el cajero ve la confirmación sin esperar al servidor (ADR-001, ADR-011). La llamada al backend ocurre después, desde la `SyncOutbox`, y su latencia no está en el camino crítico. Del lado del servidor, el costo se acota ordenando los ítems por `productoId` y bloqueando los lotes en un solo `SELECT ... FOR UPDATE` ordenado, lo que evita esperas por interbloqueo; el catálogo y las configuraciones se sirven desde la caché Caffeine en proceso (ADR-019).
