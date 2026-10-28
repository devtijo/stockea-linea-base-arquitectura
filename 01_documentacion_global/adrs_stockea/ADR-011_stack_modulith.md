# ADR-011 — Stack de implementación de Frontend, Backend y Worker/Jobs

| Campo                   | Valor                                                                                  |
| ----------------------- | -------------------------------------------------------------------------------------- |
| Estado                  | **Aprobado (corregido)**                                                               |
| Escenarios relacionados | ESC-CAL-DIS-0001, ESC-CAL-DIS-0005, restricciones de equipo (3 personas) y presupuesto |
| Atributos de calidad    | Mantenibilidad, Disponibilidad, Viabilidad económica                                   |

### Decisión

- **Frontend:** React + Vite + `vite-plugin-pwa` (Workbox) + Dexie.js.
- **Backend y Worker/Jobs:** Java + Spring Boot. El Worker es el **mismo JAR** que el Backend, ejecutado con perfil `worker`, con tareas periódicas vía `@Scheduled`/Spring Batch.
- **Modularidad del Backend:** monolito modular con **Spring Modulith**. Los eventos de dominio de Modulith son **internos al proceso del Backend**; Backend y Worker no se comunican por eventos ni por HTTP, solo a través de PostgreSQL (tablas de resultados y outbox).

### Justificación

Ambas decisiones priorizan la consistencia con lo ya aprobado y la carga de mantenimiento de un equipo de 3 personas. React es la única opción que se alinea sin fricción con Workbox y Dexie.js (ADR-001). Spring Boot formaliza lo que ADR-004, ADR-005 y ADR-009 ya asumían (Spring Security, Spring Data). Usar el mismo stack en el Worker evita mantener dos ecosistemas de backend.

### Tácticas arquitectónicas (adiciones)

- **CI/CD:** el destino de las imágenes Docker es **GitHub Container Registry (GHCR)** (gratuito para repos públicos, plan generoso para privados), no un registry genérico.
- **Worker/Jobs:** mismo JAR con perfil `worker`; usa **ShedLock** para evitar ejecución duplicada si hay más de una instancia.
- **Comprobantes PDF:** se generan en el **cliente**. El Backend solo expone un endpoint para regenerarlos desde la BD si es necesario.
- Frontend y Backend se mantienen como proyectos independientes, comunicándose solo vía REST/JSON.

### Consecuencias

- El contenedor CI/CD especifica "GitHub Actions + GHCR".
- El Worker comparte código con el Backend pero se despliega como proceso independiente (aislamiento de fallos, ESC-CAL-DIS-0029).
- Se actualizan los diagramas C4 (Nivel 2 y 3) para reflejar estas marcas (incluyendo corregir "EF Core" → Spring Data).

**Enmienda (2026-10-28) — Estructura modular del Backend.** El Backend se materializó como **monolito modular con Spring Modulith** bajo `com.stockea`. Los módulos `ventas`, `inventario`, `sync`, `caja` y `descuentos` se organizan en `domain`, `application`, `infrastructure` y `api`. Un módulo solo puede depender de la `api` de otro (por ejemplo `InventarioApi` o `CajaApi`), declarada con `@NamedInterface("api")`, y `ApplicationModules.verify()` en las pruebas falla si hay una dependencia indebida o un ciclo. `sync` no depende de `ventas`: expone una interfaz `ProcesadorOperacion` que `ventas` implementa. El módulo `shared` (`TenantContext`, `DomainException`) es abierto y `config` solo contiene beans.
