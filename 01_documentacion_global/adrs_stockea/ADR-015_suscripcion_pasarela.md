# ADR-015 — Flujo de suscripción y pasarela de pagos

| Campo                   | Valor                                                  |
| ----------------------- | ------------------------------------------------------ |
| Estado                  | **Aprobado**                                           |
| Escenarios relacionados | RF-016.x (ex RF-012.1 bis), restricciones estratégicas |
| Atributos de calidad    | Disponibilidad, Rendimiento                            |

### Contexto

"Validar el plan en cada acceso" contra la pasarela rompe el offline y añade latencia en el camino crítico. Además había dos pasarelas (Wompi y PayU) cuando una basta para el MVP, y el flujo de pago no estaba modelado (faltaban webhook y checkout).

### Alternativas evaluadas — Pasarela

|#|Alternativa|Ventajas|Desventajas|
|---|---|---|---|
|A|**Wompi**|Soporta Nequi y Botón Bancolombia. API moderna. Presencia local fuerte.|—|
|B|PayU|Amplia presencia en LatAm.|Integración más pesada para el caso local.|
|C|Ambas|Flexibilidad.|Duplica integración para el MVP.|

> **Comisiones:** usar "tarifa vigente según wompi.co (consultada al cierre del MVP)". No se fijan cifras en el ADR porque cambian (Wompi ajustó tarifas en 2026).

### Decisión

**Wompi** como pasarela principal. **PayU** queda como adaptador alterno detrás del `PaymentPort` (demuestra la arquitectura hexagonal; no se implementa en el MVP).

**Flujo de suscripción:**

- Tabla `suscripcion(plan, estado, valido_hasta, gracia_hasta)` en la BD del negocio.
- Actualizada por: **webhook firmado** de Wompi → Backend (verificación de firma + deduplicación por id de evento) y **conciliación periódica del Worker** (RF-016.x).
- **El acceso solo consulta el estado local.**
- **Checkout hospedado** por Wompi (PCI mínimo): Frontend → Wompi (redirección).
- **Periodo de gracia offline:** 7 días (decisión de negocio).

### Flechas obligatorias en el diagrama

- **Wompi → Backend**, etiqueta "webhook firmado (HTTPS)".
- **Frontend → Wompi**, etiqueta "checkout hospedado (HTTPS)".

### Consecuencias

- Se resuelve la latencia y la dependencia en el camino crítico.
- Se añade 1 tabla y 1 endpoint. Se renumera RF-012.1 (bis) como RF-016.x.
- El contenedor externo se llama **"Pasarela de Pagos (Wompi)"**.

### Nota de alcance (2026-10-28)

El estado de suscripción se consulta localmente al abrir sesión y al abrir turno, no por venta. Registrar una venta nunca consulta la suscripción, para no añadir una dependencia al camino crítico offline.
