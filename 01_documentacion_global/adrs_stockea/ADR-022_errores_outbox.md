# ADR-022 — Manejo de errores de la outbox (5xx vs 4xx)

| Campo                   | Valor                              |
| ----------------------- | ---------------------------------- |
| Estado                  | **Aprobado**                       |
| Escenarios relacionados | ESC-CAL-DIS-0001, ESC-CAL-DIS-0008 |
| Atributos de calidad    | Confiabilidad, Disponibilidad      |
| Relacionados            | ADR-001, ADR-002, ADR-008          |

## Contexto

La outbox reintenta hasta que el servidor confirma. Reintentar a ciegas un error permanente —un payload malformado, un token revocado— bloquea la cola indefinidamente y, como las operaciones se procesan en orden de `secuenciaDispositivo`, una sola operación envenenada detendría todas las ventas posteriores de ese dispositivo. No reintentar un error transitorio, en cambio, pierde ventas. Hacía falta una política explícita.

## Decisión

| Respuesta | Significado | Acción de la outbox |
|---|---|---|
| Red caída, timeout, **5xx** (incluye `VentaException` mapeada a 500) | Fallo transitorio; la transacción hizo `ROLLBACK` y no quedó rastro parcial | **Reintenta** con backoff exponencial y *jitter*, con el **mismo `operacion_id`**. Es seguro por la idempotencia del ADR-002 |
| **200** con resultado por operación | Procesada, incluso si el resultado es `DISCREPANCIA` o `RECHAZADA_SIN_TURNO` | Borra la operación de la cola y actualiza el estado local |
| **401** | Token vencido | Refresca la sesión y reintenta sin perder la cola (ADR-001) |
| **4xx** (400, 403, 422) | Error permanente de payload o de permisos | **No reintenta.** Marca la operación como `FALLIDA`, conserva el payload íntegro y la muestra al supervisor |

Reglas complementarias:

- **Las reglas de negocio no devuelven 4xx.** Stock insuficiente, producto inactivo, precio diferente y turno cerrado viajan dentro del `200`, en el resultado de cada operación. Un 4xx significa que la petición estaba mal formada o no autorizada, nunca que el negocio dijo que no.
- Una operación `FALLIDA` **no bloquea** las siguientes: la cola continúa con la siguiente `secuenciaDispositivo`.
- Ninguna operación se descarta en silencio. Si no se puede procesar, queda visible con su payload para diagnóstico.

## Justificación

La distinción entre fallo transitorio y permanente es la que decide si reintentar ayuda o solo repite el daño. Apoyarla en la familia del código HTTP la hace evidente para quien lea los logs y no requiere un protocolo propio. Sacar las reglas de negocio de los códigos de error es lo que hace que esta política sea estable: si "stock insuficiente" fuera un 422, la outbox descartaría ventas legítimas.

## Consecuencias

- (+) La cola nunca se atasca ni pierde operaciones en silencio.
- (+) El backoff con *jitter* evita que todos los dispositivos de un negocio reintenten a la vez al volver la conexión.
- (−) Hace falta una pantalla de "operaciones fallidas" para el supervisor.
- (−) Una operación `FALLIDA` puede quedar fuera de orden respecto de las que siguieron; el `operacion_sync` conserva la secuencia original para reconstruir el hilo.

## Spike relacionado

**SPK-006** — Idempotencia bajo concurrencia y reintentos. 1 día.
