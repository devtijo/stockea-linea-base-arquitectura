# ADR-019 — Estrategia de caché: Caffeine (MVP) → Valkey (Fase 2)

| Campo                   | Valor                              |
| ----------------------- | ---------------------------------- |
| Estado                  | **Aprobado**                       |
| Escenarios relacionados | ESC-CAL-REN-0021, ESC-CAL-REN-0029 |
| Atributos de calidad    | Rendimiento                        |

### Contexto

El Backend necesita cachear lecturas de catálogo, precios y configuraciones (ESC-CAL-REN-0021/0029). Con una sola instancia de Backend en el MVP, no se necesita una caché distribuida.

### Alternativas evaluadas

|#|Alternativa|Ventajas|Desventajas|
|---|---|---|---|
|A|**Caffeine (caché en proceso)**|Apache 2.0. Sin contenedor. Spring Boot lo auto-configura. Latencia de nanosegundos.|No compartida entre instancias; se invalida al reiniciar la JVM.|
|B|**Valkey (fork BSD de Redis)**|BSD 3-Clause. API compatible con Redis. Caché distribuida.|Requiere un contenedor adicional (~100 MB RAM).|
|C|Redis|Maduro. Redis 8 (mayo 2025) volvió a AGPLv3, que sí es OSI.|Contenedor adicional innecesario para una sola instancia.|

### Decisión

**MVP: Caffeine. Fase 2: Valkey** cuando exista más de una instancia de Backend.

- La caché vive **detrás del puerto de repositorio** (coherente con "cache va en repositorio" de las notas de clase); el dominio no sabe que existe.
- **Claves con prefijo de tenant obligatorio** (`negocio:{negocio_id}:catalogo:...`) para evitar fugas entre tenants.
- **TTL corto** (5 min), invalidado por evento de dominio al cambiar.

### Disparadores para migrar a Valkey

- Más de 1 instancia de Backend; latencia medida en consultas de catálogo; necesidad de invalidación compartida.

### Consecuencias

- Se elimina el contenedor Redis del Nivel 2. El MVP no necesita caché distribuida. La migración a Valkey es transparente gracias al puerto.

### Referencias

- Caffeine documentation; Valkey (Linux Foundation); cambio de licencia de Redis (2024) y retorno a AGPLv3 en Redis 8 (mayo 2025).

### Nota (2026-10-28) — Caché y precio vigente

`precioVigente(productoId, fechaVenta)` depende de la fecha de la operación, no del instante actual, por lo que **no puede cachearse** con la clave de catálogo `negocio:{negocio_id}:catalogo:...` ni con TTL de 5 minutos. Si se cachea, la clave debe ser `negocio:{negocio_id}:precio:{productoId}:{fechaVenta}`; las entradas con fecha pasada son inmutables (TTL largo) y el precio del día en curso no se cachea porque puede cambiar (ADR-021).
