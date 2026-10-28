# ADR-017 — Notificaciones: Web Push primero, correo después

| Campo                   | Valor                            |
| ----------------------- | -------------------------------- |
| Estado                  | **Aprobado**                     |
| Escenarios relacionados | ESC-CAL-USA-0017, RF-008.3 / 8.4 |
| Atributos de calidad    | Usabilidad                       |

### Decisión

- **Canal 1 — Web Push (VAPID)** como canal principal de alertas de stock y vencimiento. Requiere un **sistema externo: el servicio push del navegador (FCM / APNs / Mozilla)**, que se añade al diagrama.
- **Canal 2 — Correo SMTP** para comprobantes y avisos de suscripción, con proveedor barato e intercambiable (Amazon SES, Brevo, Resend).
- **Outbox** para envío fiable. **SPF/DKIM/DMARC** configurados.

### Flechas obligatorias en el diagrama

- **Frontend → Servicio push del navegador**, "suscripción push (HTTPS)".
- **Servicio push del navegador → Frontend**, "entrega de notificación".
- **Worker → Servicio push del navegador**, "notificación Web Push (HTTPS, VAPID)". El Worker genera las alertas (ADR-006).
- **Backend → Pasarela SMTP**, "comprobantes por correo (SMTP)".
- **Worker → Pasarela SMTP**, "avisos de suscripción vencida (SMTP)". Las alertas de stock y vencimiento **no** van por correo.

### Consecuencias

- Más adopción (ESC-CAL-USA-0017). Costo cero (Web Push). El correo sigue siendo necesario para comprobantes.
