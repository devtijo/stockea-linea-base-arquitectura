# ADR-013 — Endurecimiento de Cloudflare + origen

| Campo                   | Valor            |
| ----------------------- | ---------------- |
| Estado                  | **Aprobado**     |
| Escenarios relacionados | ESC-CAL-SEG-0010 |
| Atributos de calidad    | Seguridad        |

### Contexto

Si la IP del VPS es alcanzable directamente, el WAF de Cloudflare se evade. Además, el TLS termina en Cloudflare, así que el cifrado de extremo a extremo no es automático.

### Decisión

- **Cloudflare Full (strict)** con **Origin CA**.
- **Firewall del VPS** aceptando 80/443 **solo desde rangos de Cloudflare** o con **Authenticated Origin Pulls** (mTLS).
- **No exponer la IP del origen** en registros DNS ni en correos.
- **Nginx `limit_req`** como respaldo si Cloudflare cambia de plan.
- **Admin de Grafana no público** (VPN/SSH tunnel o Zero Trust gratuito). Si Keycloak se activa en Fase 2, su admin tampoco público.
- Documentar en ESC-CAL-SEG-0010 que el cifrado es Cliente → Cloudflare → Nginx, y Nginx → Backend es HTTP interno en la misma máquina.

### Consecuencias

- Seguridad ↑. Presupuesto neutro. Operación ligeramente más compleja.
