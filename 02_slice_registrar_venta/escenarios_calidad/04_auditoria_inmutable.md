# ESC-CAL-AUD-0005 — Auditoría completa y automática de la venta

Cuando el cajero confirme una venta, el sistema debe guardar automáticamente todos los metadatos de auditoría sin que el cajero ingrese nada adicional.

| Campo | Valor |
|---|---|
| Atributo de calidad | Capacidad para ser auditado |
| Prioridad | Crítica |
| Dificultad / Riesgo | Baja |
| RF relacionado | RF-004.1 |

## Definición del escenario

| # | Parte | Descripción |
|---|---|---|
| 1 | **Origen** | Cajero que confirma el pago de una venta en el punto de venta. |
| 2 | **Estímulo** | El cajero registra una venta de 3 productos con método de pago efectivo y confirma la transacción. |
| 3 | **Artefacto** | El servicio de dominio de ventas que persiste la transacción con todos los metadatos de auditoría: quién, cuándo, qué y cómo. |
| 4 | **Entorno** | Operación normal del punto de venta, en modo online u offline. |
| 5 | **Respuesta** | El sistema guarda el cajero responsable, la fecha y hora exacta, el turno asociado, el detalle de cada producto (lote, precio, descuento y subtotal), método de pago, total y cambio, sin intervención del cajero. |
| 6 | **Medida de respuesta** | 100% de ventas con auditoría completa, 0 ventas sin cajero o sin fecha y hora, sin agregar más de 100 milisegundos al proceso. |

## Nota de arquitectura

La auditoría no es un registro aparte: **es el modelo de datos**. `venta` guarda `cajero_id`, `turno_caja_id`, `fecha_cliente` y `registrada_en`; `asignacion_lote` deja el rastro por lote con su descuento; `movimiento_inventario` es append-only. El `cajero_id` y el `negocio_id` se toman del **JWT**, nunca del cuerpo de la petición, así que no dependen de que el cliente los envíe bien. La inmutabilidad se protege en dos capas (ADR-004): la API no expone endpoints destructivos sobre entidades de auditoría y el rol de base de datos tiene `REVOKE UPDATE, DELETE` sobre esas tablas. Toda corrección es un registro nuevo que referencia al original, nunca una edición.
