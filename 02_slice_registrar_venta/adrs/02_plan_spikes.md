# Plan de spikes

Los spikes están planificados pero **no ejecutados**: corresponden a la fase de implementación, no a esta línea base de diseño. Si un spike falla, se reevalúa la alternativa del ADR que lo originó antes de construir la funcionalidad.

| Spike    | ADR              | Objetivo                                                                        | Criterio de éxito                                                                                                           | Prioridad |
| -------- | ---------------- | ------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------- | --------- |
| SPK-001  | ADR-001          | Venta completa offline con Dexie + Workbox y sincronización al reconectar       | Flujo completo sin red (abrir caja → vender → cerrar caja) y sincronización correcta al volver                              | Crítica   |
| SPK-001b | ADR-001          | Login offline con PIN local, sesión 8 h, inactividad 15 min                     | El cajero enciende el equipo sin internet, entra con PIN y sigue vendiendo; al reconectar la cola se sincroniza sin pérdida | Crítica   |
| SPK-002  | ADR-002          | Dos dispositivos vendiendo del mismo lote offline                               | Ambas ventas aceptadas, FEFO reasigna, discrepancia visible, stock final nunca negativo                                     | Crítica   |
| SPK-002b | ADR-002          | 10 dispositivos offline con solapamiento total                                  | Ninguna venta se pierde; discrepancias resueltas en ≤ 1 día                                                                 | Alta      |
| SPK-003  | ADR-003          | Fraccionamiento de una línea entre varios lotes con descuentos                  | Totales que cuadran con descuento por lote                                                                                  | Alta      |
| SPK-006  | ADR-002, ADR-022 | Idempotencia bajo concurrencia: N reenvíos simultáneos del mismo `operacion_id` | Una sola venta y una sola respuesta, sin interbloqueos                                                                      | Crítica   |
| SPK-007  | ADR-021, ADR-023 | Venta offline con precio histórico y reloj desfasado                            | Se cobra el precio del cliente, se detecta `PRECIO_DIFERENTE` y el filtro de vencimiento usa `fechaVenta`                   | Alta      |

