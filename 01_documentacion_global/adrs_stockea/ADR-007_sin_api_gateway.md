# ADR-007 — No incluir un API Gateway dedicado en el MVP

| Campo                   | Valor                                                |
| ----------------------- | ---------------------------------------------------- |
| Estado                  | **Aprobado**                                         |
| Escenarios relacionados | ESC-CAL-ESC-0008, ESC-CAL-SEG-0006, ESC-CAL-DIS-0005 |
| Atributos de calidad    | Rendimiento, Seguridad, Escalabilidad                |

### Contexto

STOCKEA es multi-tenant y debe limitar el consumo por negocio según su plan, además de validar JWT y rol en cada request. El patrón habitual es un API Gateway dedicado (Kong, Apigee, AWS API Gateway).

### Decisión

**Reverse Proxy (Nginx) + middleware de autenticación y rate limiting dentro del Backend.**

### Justificación

Un Gateway que solo reenvía el JWT no aporta seguridad real, porque la validación efectiva ocurre igual en el backend (ESC-CAL-SEG-0006 exige validación local offline-aware, que un Gateway no resuelve). Si el Gateway validara el JWT y el rate limit, necesitaría conocer el plan de suscripción, dato que vive en la BD del backend, duplicando lógica. Con tráfico en ráfagas por sincronización, es más seguro diferenciar políticas por endpoint dentro del backend que aplicar un límite genérico externo.

### Tácticas arquitectónicas

- El Reverse Proxy solo termina TLS y reenvía; no inspecciona el token.
- El componente Auth & Access Control valida JWT, aplica autorización por rol y rate limiting (Bucket4j in-memory) con políticas diferenciadas (más permisiva para sincronización, más restrictiva para dashboards).

### Consecuencias

- Bucket4j in-memory funciona con una sola instancia de backend. Si se escala horizontalmente, el contador debe migrar a un almacén compartido (Valkey). Deuda técnica aceptada conscientemente.
- El endurecimiento del borde (Cloudflare + Nginx) se detalla en el **ADR-013**.
