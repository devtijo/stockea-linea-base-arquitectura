# ADR-020 — Almacenamiento de objetos: S3 externo

| Campo                   | Valor                                |
| ----------------------- | ------------------------------------ |
| Estado                  | **Aprobado**                         |
| Escenarios relacionados | Restricción legal, ESC-CAL-REN-0021  |
| Atributos de calidad    | Viabilidad económica, Mantenibilidad |

### Contexto

El diseño original usaba MinIO self-hosted para comprobantes y backups. MinIO entró en modo de mantenimiento a finales de 2025; su licencia es AGPLv3 (copyleft fuerte); auto-hospedar object storage en el mismo host que respaldas es circular; y el comprobante PDF se puede regenerar desde un registro inmutable en la BD, por lo que no requiere almacenamiento permanente propio.

### Alternativas evaluadas

|#|Alternativa|Ventajas|Desventajas|
|---|---|---|---|
|A|MinIO self-hosted|Control total.|Modo mantenimiento. AGPLv3. Circular con backups.|
|B|**S3 externo (Backblaze B2 / Cloudflare R2)**|Barato (~USD 1–5/mes). Off-site real. Sin mantenimiento. S3-compatible.|Depende de un proveedor externo.|
|C|Garage / SeaweedFS self-hosted|Self-hosted.|Complejidad operativa. No resuelve el problema de off-site.|

### Decisión

**Alternativa B: S3 externo.**

- **Destino:** Backblaze B2 o Cloudflare R2 (elegir uno).
- **Uso:** backups (pgBackRest) y exportaciones (CSV, PDF bajo demanda).
- **Comprobantes PDF:** se generan en el cliente y se regeneran desde la BD cuando se necesitan; no se almacenan permanentemente.
- **Puerto `ObjectStoragePort`** en la capa de negocio para poder cambiar de proveedor.

### Consecuencias

- Se elimina el contenedor MinIO del Nivel 2.
- Se añade un sistema externo: "Almacenamiento Off-site (S3)".
- Se añaden dos flechas: **pgBackRest → S3** ("Backups (S3 API)") y **Backend → S3** ("Exportaciones (S3 API)").

### Referencias

- MinIO maintenance mode announcement (2025); Backblaze B2 pricing; Cloudflare R2 pricing.
