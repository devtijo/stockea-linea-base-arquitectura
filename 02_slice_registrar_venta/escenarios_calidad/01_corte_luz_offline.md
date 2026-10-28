# ESC-CAL-CON-0001 — Corte de luz durante una venta

Cuando ocurra un corte de energía durante el registro de una venta, el sistema debe garantizar que al reiniciar no queden datos a medias ni inventario desactualizado.

| Campo | Valor |
|---|---|
| Atributo de calidad | Confiabilidad |
| Prioridad | Crítica |
| Dificultad / Riesgo | Alta |
| RF relacionado | RF-004.1, RF-011.1 |

## Definición del escenario

| # | Parte | Descripción |
|---|---|---|
| 1 | **Origen** | Evento externo: corte de energía eléctrica en el local durante el horario operativo. |
| 2 | **Estímulo** | Se interrumpe la alimentación del equipo mientras el cajero registra una venta de 5 productos, 3 de ellos ya agregados al carrito. |
| 3 | **Artefacto** | El módulo de ventas del frontend (almacenamiento local del dispositivo) y la capa de persistencia del backend que gestionan la transacción de venta y la actualización del inventario. |
| 4 | **Entorno** | Operación normal del punto de venta en horario comercial. La venta puede estar en modo online u offline al momento del corte. |
| 5 | **Respuesta** | Al reiniciar, la venta inconclusa no queda registrada y el inventario vuelve exactamente al estado previo al intento: sin descuentos parciales ni ventas a medias. |
| 6 | **Medida de respuesta** | 0% de impacto en inventario, 0 registros de venta corruptos, sistema operativo en menos de 30 segundos tras el reinicio. |

## Nota de arquitectura

El escenario se cumple con **atomicidad en los dos lados**. En el cliente, la venta, su entrada en la outbox y el descuento de stock local se escriben en **una sola transacción de Dexie** con `durability: 'strict'`: o se guardan las tres cosas o ninguna (ADR-001). En el servidor, la venta, las asignaciones de lote, el movimiento de inventario y la fila de `operacion_sync` se confirman en **un solo `COMMIT`** abierto por el servicio de aplicación (ADR-009); si algo falla, el `ROLLBACK` revierte también el descuento de stock. Un carrito a medio armar nunca llega a ninguna de las dos transacciones: solo se persiste al confirmar.
