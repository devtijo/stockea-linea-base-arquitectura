# Resumen de spikes requeridos

| Spike     | ADR     | Descripción                                                            | Duración    | Prioridad       |
| --------- | ------- | ---------------------------------------------------------------------- | ----------- | --------------- |
| SPK-001   | ADR-001 | Venta completa offline con Dexie + Workbox y sincronización            | 3 días      | Crítica         |
| SPK-001b  | ADR-001 | Login offline con PIN local, sesión 8 h, refresh sin perder cola       | 2 días      | Crítica         |
| SPK-002   | ADR-002 | Conflicto de sincronización entre 2 dispositivos (FEFO + discrepancia) | 2 días      | Crítica         |
| SPK-002b  | ADR-002 | 10 dispositivos offline con solapamiento total                         | 1 día       | Alta            |
| SPK-003   | ADR-003 | Fraccionamiento de venta entre lotes con descuentos (FEFO)             | 2 días      | Alta            |
| SPK-004   | ADR-006 | Motor de recomendaciones con datos simulados de 30 días                | 2 días      | Alta            |
| SPK-005   | ADR-007 | Carga de rate limiting con Bucket4j in-memory                          | 1 día       | Opcional / Baja |
| SPK-006   | ADR-002, ADR-022 | Idempotencia bajo concurrencia: N reenvíos simultáneos del mismo `operacion_id` (advisory lock + PK compuesta) | 1 día | Crítica |
| SPK-007   | ADR-021, ADR-023 | Venta offline con precio histórico y reloj desfasado (`PRECIO_DIFERENTE`, `fechaVenta`) | 1 día | Alta |
| **Total** |         |                                                                        | **15 días** |                 |

Los spikes deben ejecutarse antes de iniciar el desarrollo de las funcionalidades asociadas. Si un spike falla, se reevalúa la alternativa del ADR correspondiente antes de continuar.

Los spikes están **planificados, no ejecutados**: corresponden a la fase de implementación.
