# ADR-008 — Sincronización offline vía REST + outbox (sin Message Broker dedicado)

| Campo                   | Valor                                                |
| ----------------------- | ---------------------------------------------------- |
| Estado                  | **Aprobado**                                         |
| Escenarios relacionados | ESC-CAL-DIS-0001, ESC-CAL-DIS-0005, ESC-CAL-DIS-0008 |
| Atributos de calidad    | Disponibilidad, Rendimiento, Confiabilidad           |

### Contexto

El frontend debe encolar operaciones localmente sin conexión y sincronizarlas al reconectar (RF-011.1, RF-011.2). Se evalúa REST con outbox local vs. un Message Broker (RabbitMQ) que desacople productor y consumidor.

### Decisión

**REST + outbox pattern, sin Message Broker dedicado en el MVP.**

### Justificación

El volumen esperado (decenas de operaciones por negocio por sincronización) no justifica la infraestructura ni la complejidad de un broker. La resolución de conflictos (ADR-002) se hace en el backend al recibir la sincronización vía REST, sin intermediario asíncrono.

### Tácticas arquitectónicas

- El Sync Controller recibe lotes de operaciones con `operacion_id` y marca de secuencia.
- Procesamiento **idempotente**: reenviar la misma operación no la duplica.
- Validación por operación contra el stock real (ver ADR-002).

### Consecuencias

- Si el backend está caído durante una reconexión, el cliente reintenta con backoff exponencial; no hay cola intermedia que amortigüe.
- **RabbitMQ se difiere a Fase 2** con disparador explícito (ver ADR-012): más de 1 instancia de Backend o integraciones asíncronas externas.

### Nota de alineación (2026-10-28)

Tras la enmienda de ADR-001, `POST /api/sync/operaciones` es el **único** camino de escritura de operaciones: el cliente online envía un lote de una sola operación por el mismo endpoint. El patrón outbox + REST descrito aquí no cambia; cambia su alcance, que pasa a ser total y no solo el de reconexión. La política de reintentos y de errores 5xx/4xx se define en el ADR-022.
