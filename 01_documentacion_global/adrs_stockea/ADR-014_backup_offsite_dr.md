# ADR-014 — Backup off-site y recuperación ante desastres

| Campo                   | Valor                                           |
| ----------------------- | ----------------------------------------------- |
| Estado                  | **Aprobado**                                    |
| Escenarios relacionados | Restricción legal, ESC-CAL-CON-0001, ESC-CAL-DR |
| Atributos de calidad    | Confiabilidad, Recuperabilidad                  |

### Contexto

En el diseño original no había destino off-site: todo vivía en un VPS. Sin backup off-site no se cumple la conservación legal de registros, y un fallo del VPS pierde datos y respaldos a la vez. Además, al eliminar MinIO (ADR-020), el destino de backups debe ser externo.

### Alternativas evaluadas

|#|Alternativa|Ventajas|Desventajas|
|---|---|---|---|
|A|Backup local en el mismo VPS|Simple, sin costo.|No cumple "off-site". Un fallo del VPS pierde todo.|
|B|**Backup a S3 externo (Backblaze B2 / Cloudflare R2)**|Cumple "off-site". Costo muy bajo (~USD 1–5/mes).|Depende de un proveedor externo.|
|C|Backup a otro VPS propio|Control total.|Costo de un segundo VPS. Complejidad operativa.|

### Decisión

**Alternativa B: Backup a S3 externo.**

- **Herramienta:** pgBackRest (WAL archiving + PITR) para PostgreSQL.
- **Destino:** bucket S3-compatible en Backblaze B2 **o** Cloudflare R2 (elegir uno).
- **Retención:** **10 años** (ver ADR-018; el documento de restricciones ya fue actualizado a "10 años").
- **Escenario nuevo:** ESC-CAL-DR (RPO ≤ 5 min, RTO ≤ 2 h).
- **Simulacro de restauración trimestral.**
- **Infraestructura como código** (Compose + scripts) para reconstruir el VPS.
- **Alerta de backup fallido en MVP:** UptimeRobot (externo, gratuito) sobre endpoint `/health/backup` que verifica la recencia del último backup exitoso. Alertmanager → Fase 2.

### Consecuencias

- Presupuesto: +USD 1–5/mes. Legal ↑ (cumple conservación a 10 años).
- Se elimina el contenedor MinIO del MVP.
- El Backend ya no escribe comprobantes en MinIO; se regeneran desde la BD.

### Referencias

- pgBackRest documentation. Backblaze B2 pricing. Cloudflare R2 pricing. Art. 60 Código de Comercio de Colombia.
