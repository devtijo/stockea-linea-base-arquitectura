# ADR-004 — Inmutabilidad de registros de auditoría y separación de datos personales

| Campo                   | Valor                                  |
| ----------------------- | -------------------------------------- |
| Estado                  | **Aprobado (corregido)**               |
| Escenarios relacionados | ESC-CAL-AUD-0016, ESC-CAL-AUD-0021     |
| Atributos de calidad    | Capacidad para ser auditado, Seguridad |

### Contexto

Los registros de ventas, ajustes de inventario, aperturas de caja y cambios de precio deben ser inalterables para cumplir el Código de Comercio de Colombia y para que el dueño confíe en el sistema. A la vez, la Ley 1581 de 2012 obliga a atender el derecho de supresión de datos personales. Ambas obligaciones conviven y hay que separarlas (ver ADR-018).

### Decisión

**Inmutabilidad a nivel de aplicación + base de datos**, con **separación entre datos personales y registros mercantiles**:

- **Datos personales** (usuario, cliente, proveedor): sujetos a derecho de supresión → se **anonimizan**, no se borran.
- **Registros mercantiles** (ventas, ajustes, cierres): **inmutables** por el plazo legal (ver ADR-018).

### Justificación

La doble protección (API + BD) cubre el requisito legal y de confianza sin la complejidad de blockchain (sobreingeniería para el MVP) y sin el hueco de seguridad de confiar solo en la capa de aplicación.

### Tácticas arquitectónicas

- **API sin endpoints destructivos** sobre entidades de auditoría (sin PUT/PATCH/DELETE).
- **Permisos de BD restrictivos**: el rol de aplicación tiene `REVOKE UPDATE, DELETE` sobre tablas de auditoría; solo `INSERT` y `SELECT`.
- **Patrón de corrección**: toda corrección es un `AjusteInventario(CORRECCION)` que referencia al original, creando una cadena auditable.
- **Tabla `usuario_anonimizado`** con hash del documento original para mantener integridad referencial sin exponer datos personales.
- Entidades de dominio sin setters para campos inmutables tras la creación.

### Consecuencias

- Los errores no se "borran"; se corrigen con registros adicionales.
- El volumen de datos crece más rápido que con registros editables.
- Se requiere un **procedimiento documentado** para responder solicitudes de titulares (acceso, rectificación, supresión) sin romper la inmutabilidad mercantil.

### Enmienda (2026-10-28) — Historial de precios y corrección de inventario

- Se añade **`historial_precio`** a las tablas **inmutables** (insert-only, con trigger que bloquea `UPDATE` y `DELETE`). `producto.precio_venta` pasa a ser una **proyección** de la última fila. Cambiar un precio es insertar, nunca actualizar (ADR-021).
- Se corrige la referencia a `AjusteInventario(CORRECCION)` en el "Patrón de corrección": el modelo usa **`MovimientoInventario` de tipo `CORRECCION`**, append-only y con `operacion_id`.
