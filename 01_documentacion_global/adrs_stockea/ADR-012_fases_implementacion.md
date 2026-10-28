# ADR-012 — Fases de implementación y disparadores de Fase 2

| Campo                   | Valor                                 |
| ----------------------- | ------------------------------------- |
| Estado                  | **Aprobado**                          |
| Escenarios relacionados | Restricciones de presupuesto y tiempo |
| Atributos de calidad    | Viabilidad económica, Mantenibilidad  |

### Contexto

Trece contenedores para un MVP con USD 50/mes por 100 negocios es desproporcionado, y algunas herramientas cambiaron de licencia o de estado de mantenimiento. El equipo tiene 3 semanas por funcionalidad. La arquitectura completa (Nivel 2) se mantiene como **objetivo**, pero la implementación se ordena por fases con disparadores explícitos.

### Decisión

**Fase 1 — MVP (6 contenedores permanentes + Backup como tarea programada + infraestructura de entrega):**

|#|Contenedor|Tecnología|
|---|---|---|
|1|Frontend|React + Vite + PWA + Dexie.js|
|2|Reverse Proxy|Nginx|
|3|Backend API|Spring Boot (perfil `api`)|
|4|Worker/Jobs|Spring Boot (perfil `worker`)|
|5|PostgreSQL|Multi-tenant con RLS|
|6|Monitoring|Prometheus + Grafana self-hosted|
|—|Backup (tarea programada, no contenedor)|pgBackRest + cron|
|—|Infraestructura de entrega (no es runtime)|GitHub Actions + GHCR|

**Sistemas externos (6):**

|#|Sistema externo|Tecnología|
|---|---|---|
|1|CDN/WAF|Cloudflare|
|2|Pasarela de Pagos|Wompi|
|3|Almacenamiento off-site|S3 (Backblaze B2 / Cloudflare R2)|
|4|Notificaciones — correo|SMTP genérico (Mailtrap en pruebas; SES / Brevo en producción)|
|5|Notificaciones — Web Push|Servicio push del navegador (FCM / APNs / Mozilla)|
|6|Monitoreo externo|UptimeRobot (uptime y alerta de backup fallido)|

**Fase 2 — Escalamiento (disparadores):**

|Componente|Disparador|
|---|---|
|Valkey (caché distribuida)|Más de 1 instancia de Backend o latencia medida en catálogo.|
|RabbitMQ|Más de 1 instancia de Backend o integraciones asíncronas externas.|
|Keycloak|SSO, MFA o federación requeridos.|
|OpenBao|Más de un host o rotación exigida de credenciales.|
|Garage / SeaweedFS|Necesidad real de archivos que no sean backups.|
|Alertmanager|Más de 1 servicio que requiera alertas proactivas. En MVP se usa UptimeRobot (externo).|

**Fase 3 — Producción:**

|Componente|Disparador|
|---|---|
|Jaeger/Tempo|Tracing distribuido requerido.|
|Loki|Volumen de logs que supere `docker logs`.|
|Guardianes CI/CD (Sonar, Trivy, ArchUnit)|Antes del primer despliegue a producción.|

### Regla de representación en el diagrama

- **Sólido** = contenedor de Fase 1.
- **Punteado con etiqueta "Fase 2"** = contenedor diferido. En el N2 se dibujan solo los que tienen dependencia directa con Backend/Worker: **RabbitMQ, Keycloak, OpenBao**. **Valkey, Garage/SeaweedFS y Alertmanager** se documentan únicamente en la tabla de Fase 2, con su disparador.
- CI/CD y GHCR son infraestructura de entrega, no contenedores de runtime. UptimeRobot es un sistema externo.
- **Caffeine no se dibuja como contenedor**: vive dentro del Backend.

### Consecuencias

- El diagrama de Nivel 2 muestra la arquitectura objetivo con la regla sólido/punteado.
- El diagrama de Despliegue muestra la arquitectura por fases.
- El equipo sabe qué implementar primero y cuándo activar cada contenedor diferido.
