# ADR-016 — Observabilidad completa

| Campo                   | Valor                                         |
| ----------------------- | --------------------------------------------- |
| Estado                  | **Aprobado**                                  |
| Escenarios relacionados | Notas de clase (OTel + MDC), ESC-CAL-DIS-0029 |
| Atributos de calidad    | Capacidad para ser observado                  |

### Decisión

- **Prometheus scrapea** Reverse Proxy, Backend (`/actuator/prometheus`), Worker y Postgres exporter, por **HTTP interno** (misma máquina). Grafana no se expone públicamente (ADR-013).
- **MVP: UptimeRobot** (externo, gratuito) para chequeo de uptime y alerta de backup fallido.
- **Fase 2: Alertmanager** con notificaciones a correo/Telegram.
- **OpenTelemetry Collector** (o Grafana Cloud OTLP) para trazas y logs.
- **Loki** para logs centralizados (Fase 3).
- **MDC** con `negocio_id`, `usuario_id`, `dispositivo_id`, `operacion_id`.
- **Métricas de negocio:** cola de sync pendiente, operaciones rechazadas, edad del último backup.

### Consecuencias

- Se habilitan los escenarios de seguridad y auditoría. Caídas detectadas en ≤ 5 min.
- +RAM si todo es self-hosted; Grafana Cloud ahorra RAM.
