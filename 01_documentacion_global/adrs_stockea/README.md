# STOCKEA — Decisiones Arquitectónicas (ADR)

Índice completo de las decisiones de arquitectura (ADR) de STOCKEA. Cada fila enlaza al documento con el contexto, las alternativas evaluadas, la decisión y su justificación. Para los ADR que aplican al slice "Registrar venta" ver [el índice del slice](../../02_slice_registrar_venta/adrs/01_adrs.md).

| ADR                                                                                  | Título                                                      | Estado   | Última enmienda |
| ------------------------------------------------------------------------------------ | ----------------------------------------------------------- | -------- | --------------- |
| [ADR-001](ADR-001_operacion_offline.md) | Estrategia de operación offline                             | Aprobado | 2026-10-28      |
| [ADR-002](ADR-002_conflictos_sync.md)   | Resolución de conflictos en sincronización                  | Aprobado | 2026-10-28      |
| [ADR-003](ADR-003_fefo_descuento.md)    | Descuento de stock por lote (FEFO)                          | Aprobado | 2026-10-28      |
| [ADR-004](ADR-004_inmutabilidad.md)     | Inmutabilidad de registros + separación de datos personales | Aprobado | 2026-10-28      |
| [ADR-005](ADR-005_rbac_acumulativo.md)                                               | RBAC acumulativo                                            | Aprobado | —               |
| [ADR-006](ADR-006_motor_recomendaciones.md)                                          | Motor de recomendaciones basado en reglas                   | Aprobado | —               |
| [ADR-007](ADR-007_sin_api_gateway.md)                                                | No incluir API Gateway dedicado                             | Aprobado | —               |
| [ADR-008](ADR-008_sync_rest_outbox.md)                                               | Sincronización vía REST + outbox (sin broker)               | Aprobado | 2026-10-28      |
| [ADR-009](ADR-009_unit_of_work.md)      | Acceso a datos centralizado (Unit of Work)                  | Aprobado | 2026-10-28      |
| [ADR-010](ADR-010_infraestructura_self_hosted.md)                                    | Infraestructura de soporte self-hosted open source          | Aprobado | —               |
| [ADR-011](ADR-011_stack_modulith.md)    | Stack de implementación (Front/Back/Worker)                 | Aprobado | 2026-10-28      |
| [ADR-012](ADR-012_fases_implementacion.md)                                           | Fases de implementación y disparadores                      | Aprobado | —               |
| [ADR-013](ADR-013_endurecimiento_cloudflare.md)                                      | Endurecimiento Cloudflare + origen                          | Aprobado | —               |
| [ADR-014](ADR-014_backup_offsite_dr.md)                                              | Backup off-site y recuperación ante desastres               | Aprobado | —               |
| [ADR-015](ADR-015_suscripcion_pasarela.md)                                           | Flujo de suscripción y pasarela de pagos                    | Aprobado | 2026-10-28      |
| [ADR-016](ADR-016_observabilidad.md)                                                 | Observabilidad completa                                     | Aprobado | —               |
| [ADR-017](ADR-017_notificaciones.md)                                                 | Notificaciones: Web Push primero, correo después            | Aprobado | —               |
| [ADR-018](ADR-018_retencion_datos.md)                                                | Retención y datos personales                                | Aprobado | —               |
| [ADR-019](ADR-019_estrategia_cache.md)                                               | Estrategia de caché: Caffeine → Valkey                      | Aprobado | 2026-10-28      |
| [ADR-020](ADR-020_almacenamiento_s3.md)                                              | Almacenamiento de objetos: S3 externo                       | Aprobado | —               |
| [ADR-021](ADR-021_autoridad_precio.md)  | Autoridad del precio (el servidor no recotiza)              | Aprobado | —               |
| [ADR-022](ADR-022_errores_outbox.md)    | Manejo de errores de la outbox (5xx vs 4xx)                 | Aprobado | —               |
| [ADR-023](ADR-023_manejo_fechas.md)     | Manejo de fechas y zona horaria                             | Aprobado | —               |

Ver también el [resumen de spikes](Resumen_spikes.md).
