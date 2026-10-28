# ADR-023 — Manejo de fechas y zona horaria

| Campo                   | Valor                                      |
| ----------------------- | ------------------------------------------ |
| Estado                  | **Aprobado**                               |
| Escenarios relacionados | ESC-CAL-CON-0014, ESC-CAL-DIS-0008         |
| Atributos de calidad    | Confiabilidad, Capacidad para ser auditado |
| Relacionados            | ADR-002, ADR-003, ADR-021                  |

## Contexto

En un sistema offline-first la fecha de una operación y la fecha en que el servidor la procesa son distintas, y pueden separarse por días. Tres reglas del negocio dependen de "qué día fue": qué lotes estaban vencidos al momento de la venta (ADR-003), qué precio estaba vigente (ADR-021) y qué turno de caja cubría la operación. Si esas reglas se evalúan con la hora del servidor, una venta hecha el martes con un lote que vence el miércoles se rechazaría al sincronizar el jueves. Además, un vencimiento es una **fecha calendario** del negocio, no un instante: "vence el 30 de octubre" no cambia de día porque el servidor esté en UTC.

El reloj del dispositivo tampoco es confiable: puede estar desajustado por accidente o a propósito.

## Alternativas evaluadas

| # | Alternativa | Ventajas | Desventajas |
|---|---|---|---|
| A | Usar `CURRENT_DATE` / `now()` del servidor | Trivial | Rompe el offline: la regla depende de cuándo se sincroniza, no de cuándo se vendió. Depende de la zona horaria de la sesión de PostgreSQL |
| B | Confiar en la fecha del cliente tal cual | Respeta el momento real de la venta | Un reloj adelantado permite vender lotes aún no vencidos como si lo estuvieran, o fijar precios futuros |
| C | **Fecha del cliente acotada por el servidor y convertida a la zona del negocio** | Respeta el momento real y limita la manipulación | Requiere inyectar un `Clock` y disciplina para no usar `now()` en el dominio |

## Decisión

**Alternativa C.**

1. Toda operación viaja con `fechaCliente` (`TIMESTAMPTZ`, instante con zona), generada en el dispositivo y persistida sin alterar en `venta.fecha_cliente` y `operacion_sync.creada_en_cliente`. Es el dato de auditoría: lo que el dispositivo afirma.
2. El servidor deriva la **fecha de negocio**:
   `fechaVenta = min(fechaCliente, clock.instant()).atZone(ZoneId.of("America/Bogota")).toLocalDate()`
   El `Clock` se inyecta como bean (nunca `LocalDate.now()` directo), lo que además hace testeables las reglas de vencimiento.
3. **Todas** las reglas de negocio dependientes del día usan `fechaVenta`: filtro `lote.fecha_vencimiento >= fechaVenta`, `precioVigente(productoId, fechaVenta)` y la validación de turno (`turnoAbiertoEn(cajeroId, fechaCliente)`, que compara instantes).
4. **Prohibido** `CURRENT_DATE`, `now()` y `CURRENT_TIMESTAMP` dentro de reglas de negocio, tanto en SQL como en Java. Quedan permitidos solo en columnas de auditoría (`registrada_en`, `recibida_en`, `creado_en`).
5. `lote.fecha_vencimiento` es **`DATE`**, no `TIMESTAMPTZ`: es una fecha calendario del negocio sin hora ni zona. Los instantes técnicos (`registrada_en`, `fecha_entrada`, `vigente_desde`) sí son `TIMESTAMPTZ` en UTC.
6. La zona del negocio se guarda como configuración (`America/Bogota` por defecto) y no se deduce de la zona del servidor ni de la del navegador.

## Justificación

La fecha de negocio tiene que ser una función determinista del hecho, no del momento en que se procesa; esa es la única forma de que reprocesar la cola dé el mismo resultado. Acotar con `min(fechaCliente, ahora)` cierra el abuso más fácil, el reloj adelantado, sin romper el caso legítimo de sincronizar días después. El reloj atrasado sigue siendo posible y no se previene: se **detecta**, porque elegir una fecha pasada cambia el precio vigente y eso genera una discrepancia `PRECIO_DIFERENTE` (ADR-021).

## Consecuencias

- (+) El resultado de sincronizar no depende de cuándo se sincroniza.
- (+) Las reglas de vencimiento son testeables con un `Clock` fijo.
- (+) La zona horaria del servidor y la de la sesión de PostgreSQL dejan de ser relevantes para el negocio.
- (−) Riesgo residual aceptado: un reloj atrasado deliberadamente puede apuntar a un precio histórico; queda registrado como discrepancia, no bloqueado.
- (−) Exige una regla de revisión en code review: cualquier `now()` en `domain` o `application` es un hallazgo.
- Un negocio en otra zona horaria requiere configurar la suya; el valor no se infiere.

## Spike relacionado

**SPK-007** — Venta offline con reloj desfasado: verificar que `fechaVenta` se acota, que el filtro de vencimiento la usa y que la diferencia de precio se detecta. 1 día.
