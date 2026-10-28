# ADR-009 — Acceso a datos centralizado (Unit of Work)

| Campo                   | Valor                                                                  |
| ----------------------- | ---------------------------------------------------------------------- |
| Estado                  | **Aprobado**                                                           |
| Escenarios relacionados | ESC-CAL-CON-0001, ESC-CAL-CON-0003, ESC-CAL-CON-0014, ESC-CAL-ESC-0008 |
| Atributos de calidad    | Confiabilidad, Escalabilidad                                           |

### Contexto

Varias operaciones (registrar venta, ajustar inventario) modifican más de una entidad en una transacción atómica, y toda consulta debe filtrar por `negocio_id`. Se decide entre un repositorio compartido (Unit of Work) o repositorios por dominio.

### Decisión

**Data Repository como Unit of Work único, documentado explícitamente como fachada consciente.**

### Justificación

**Spring Data** provee manejo transaccional nativo sobre un contexto de persistencia compartido (`EntityManager`). Envolver varias entidades en una transacción es más simple y menos propenso a error con un Unit of Work único que coordinando repositorios independientes. El riesgo de "God Object" se mitiga documentando el límite: el Data Repository **es una fachada que coordina repositorios por agregado** y solo hace persistencia y filtrado multi-tenant, nunca lógica de negocio (que vive en los domain services).

> **Nota de corrección:** el stack es **Java + Spring Boot**; el componente de acceso a datos es **Spring Data / JPA (Hibernate)**. Cualquier referencia residual a "EF Core" en los diagramas es un error heredado de una plantilla .NET y debe corregirse a **Spring Data** (ver checklist del diagrama al final).

### Tácticas arquitectónicas

- Filtro automático por `negocio_id` inyectado en cada consulta (reforzado con RLS en PostgreSQL).
- Método transaccional explícito para operaciones cross-entidad (p. ej. `registrarVentaConAjusteDeLote`).
- El repositorio no contiene reglas de negocio.

### Consecuencias

- Si el Data Repository crece, se divide en repositorios por agregado (Sales, Inventory), manteniendo el Unit of Work como fachada coordinadora.

**Enmienda (2026-10-28) — Unit of Work en el diseño final.** La decisión de un “Data Repository como fachada única” se materializó de otra forma. Hay un repositorio Spring Data por agregado dentro de cada módulo, y la transacción atómica la abre el servicio de aplicación (`@Transactional` en `VentasService` o `SyncService`). Esa misma transacción incluye las llamadas a `InventarioApi` y a `CajaApi`, y confirma venta, asignaciones, descuento de stock y `operacion_sync` en un solo `COMMIT`. Se mantiene el propósito del ADR (atomicidad, filtro por `negocio_id` y RLS, sin lógica de negocio en repositorios) y se aplica la consecuencia que ya preveía: dividir los repositorios por agregado.
