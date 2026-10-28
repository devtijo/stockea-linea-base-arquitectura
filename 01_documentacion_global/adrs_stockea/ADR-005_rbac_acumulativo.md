# ADR-005 — Control de acceso basado en roles acumulativos (RBAC)

| Campo                   | Valor                              |
| ----------------------- | ---------------------------------- |
| Estado                  | **Aprobado**                       |
| Escenarios relacionados | ESC-CAL-SEG-0006, ESC-CAL-SEG-0004 |
| Atributos de calidad    | Seguridad                          |

### Contexto

STOCKEA tiene 3 roles (Cajero, Supervisor, Administrador). Se decide si los roles son independientes o acumulativos.

### Decisión

**Roles acumulativos** (Cajero ⊂ Supervisor ⊂ Administrador).

### Justificación

En una tienda de 1-5 empleados, la simplicidad del modelo acumulativo supera a la granularidad. El dueño entiende intuitivamente que "el administrador puede hacer todo". Los modelos granulares agregan complejidad de configuración sin valor para el mercado objetivo.

### Tácticas arquitectónicas

- Enum jerárquico: `CAJERO(1)`, `SUPERVISOR(2)`, `ADMINISTRADOR(3)`.
- Anotaciones de autorización en servicios de dominio; validación en el backend, no solo en el frontend.
- **Validación offline** contra el token cacheado localmente.
- **Refuerzo:** el Sync Controller **revalida permisos por operación al sincronizar**, contra el estado actual del usuario.
- Filtrado de UI por rol como conveniencia, nunca como única protección.

### Consecuencias

- Un usuario tiene un solo rol a la vez.
- Un rol que rompa la jerarquía (p. ej. "Auditor" de solo lectura) exigiría extender el modelo.
