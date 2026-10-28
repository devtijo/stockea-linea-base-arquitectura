# Restricciones técnicas

## Stack obligado

- **Solo open source con licencia OSI verificada.** Cada componente se valida antes de adoptarse: quedaron fuera MinIO (modo mantenimiento, AGPLv3) y HashiCorp Vault (BSL desde 2023, no OSI).
- **Backend:** Java 21 LTS + Spring Boot 3.5.x. **Frontend:** React + Vite + `vite-plugin-pwa` (Workbox) + Dexie.js. **Base de datos:** PostgreSQL.
- El **Worker** es el mismo JAR que el Backend ejecutado con perfil `worker`; no se mantienen dos ecosistemas.

## Arquitectura

- **PWA offline-first obligatoria**: la aplicación debe poder vender, consultar inventario y cerrar caja sin conexión, con Service Worker e IndexedDB persistente.
- **Monolito modular** con Spring Modulith, no microservicios: el equipo es de 3 personas y la infraestructura es un VPS.
- **Multi-tenant con discriminador `negocio_id` + Row-Level Security** en una sola instancia de PostgreSQL. Una base o esquema por negocio multiplicaría el costo operativo por encima del techo de presupuesto.
- **Sin API Gateway dedicado**: Nginx termina TLS y reenvía; la validación de JWT, la autorización y el rate limiting (Bucket4j) viven dentro del Backend.
- **Registros de auditoría inmutables** a nivel de API y de base de datos (`REVOKE UPDATE, DELETE`).

## Infraestructura

- **Un solo VPS** en Fase 1, con 6 contenedores: Frontend, Nginx, Backend, Worker, PostgreSQL y Monitoring. El backup es una tarea programada, no un contenedor.
- **Backup off-site obligatorio** en S3 externo (Backblaze B2 o Cloudflare R2): respaldar en el mismo host que se respalda no cumple la conservación legal.
- Componentes diferidos a Fase 2 con **disparador explícito**: Valkey, RabbitMQ, Keycloak, OpenBao, Alertmanager.
- Entrega con **GitHub Actions + GHCR**; no son contenedores de ejecución.

## Operación

- El sistema debe funcionar en equipos modestos, con varias pestañas abiertas y jornadas de 8 horas continuas, sin degradación.
- La sesión offline dura 8 horas y se cierra por 15 minutos de inactividad, evaluado localmente.

[Restricciones técnicas globales](../../01_documentacion_global/restricciones/restricciones_tecnicas.md)