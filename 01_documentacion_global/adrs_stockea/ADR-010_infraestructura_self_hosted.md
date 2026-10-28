# ADR-010 — Infraestructura de soporte self-hosted y open source

| Campo                   | Valor                                               |
| ----------------------- | --------------------------------------------------- |
| Estado                  | **Aprobado (reescrito)**                            |
| Escenarios relacionados | Restricciones de presupuesto y de stack tecnológico |
| Atributos de calidad    | Viabilidad económica del producto                   |

### Contexto

STOCKEA necesita componentes de soporte: identidad, almacenamiento de objetos, caché, gestor de secretos y observabilidad (APM). La restricción de presupuesto (USD 50/mes por 100 negocios) y la de "solo open source" obligan a verificar la licencia de cada componente y a preferir alternativas livianas en el MVP, porque varias herramientas candidatas cambiaron de licencia o de estado de mantenimiento.

### Alternativas evaluadas

|#|Alternativa|Ventajas|Desventajas|
|---|---|---|---|
|A|Self-hosted open source sobre el mismo VPS|Sin costo de licencia. Cumple presupuesto.|Responsabilidad operativa propia.|
|B|SaaS gestionados|Cero mantenimiento.|Costo mensual que supera el techo de presupuesto.|
|C|**Self-hosted open source con componentes verificados de licencia OSI**|Cumple estrictamente "solo open source". Sin costo de licencia.|Requiere verificar cada licencia al elegir.|

### Decisión

**Alternativa C.** Componentes finales del MVP y verificación de licencia:

|Componente|Decisión MVP|Licencia|Justificación|
|---|---|---|---|
|**Identidad**|Spring Security + JWT + tabla `usuario` propia, detrás de `AuthPort`|Apache 2.0|Suficiente para MVP. Keycloak se difiere a Fase 2 (SSO/MFA/federación).|
|**Object Storage**|**Eliminado del MVP.** Backups y exportaciones van a **S3 externo** (Backblaze B2 / Cloudflare R2).|—|MinIO en modo de mantenimiento desde finales de 2025. Auto-hospedar storage en el mismo host que respaldas es circular. (ver ADR-020)|
|**Secrets**|Docker secrets + SOPS/age (MVP) → **OpenBao** (Fase 2)|MPL 2.0 (OpenBao)|Vault es BSL desde 2023 (no OSI). OpenBao es el fork compatible con MPL 2.0.|
|**Cache**|**Caffeine** (MVP) → **Valkey** (Fase 2)|Apache 2.0 / BSD 3-Clause|Una sola instancia de Backend no necesita caché distribuida. Valkey es el fork BSD si se requiere. (ver ADR-019)|
|**Observabilidad**|Prometheus + Grafana self-hosted|Apache 2.0 / AGPLv3|Estándar de la industria, ambos OSI.|
|**Backup**|pgBackRest → bucket S3 externo|MIT|PITR y WAL archiving. Destino off-site en B2/R2. (ver ADR-014)|

### Consecuencias

- El MVP es más liviano: menos contenedores, menos RAM, sin problemas de licencia.
- Se eliminan 3 contenedores del MVP respecto al diseño original (MinIO, Redis, Vault) y se reemplazan por alternativas más simples o externas.
- Los contenedores diferidos se documentan con disparadores en el ADR-012.

### Referencias

- Business Source License 1.1 (HashiCorp Vault, 2023).
- MinIO Community License y estado de mantenimiento (2025).
- OpenBao (Linux Foundation).
- Valkey (Linux Foundation); Redis 8 volvió a AGPLv3 (OSI) en mayo 2025 — ver ADR-019.
