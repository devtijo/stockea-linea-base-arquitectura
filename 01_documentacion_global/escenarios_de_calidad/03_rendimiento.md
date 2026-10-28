## 12. ESC-CAL-REN-0021

Al registrar una venta con hasta 10 productos, el sistema debe completar la operación en un tiempo máximo de 3 segundos.

| Campo               | Valor                      |
| ------------------- | -------------------------- |
| Atributo de Calidad | Rendimiento                |
| Prioridad           | Crítica                    |
| Dificultad / Riesgo | Media                      |
| RF relacionado      | RF-004.1 (Registrar venta) |

### Definición del Escenario

|Parte|Componente|Descripción|
|---|---|---|
|1|Origen|Cajero que confirma el pago de una venta con múltiples productos durante el horario comercial.|
|2|Estímulo|El cajero presiona el botón de confirmar pago en una venta que contiene 10 productos distintos con cantidades variadas, algunos con descuento activo.|
|3|Artefacto|El flujo completo de procesamiento de venta: validación de stock por lote, cálculo de descuentos, actualización de lotes y stock, y generación de comprobante.|
|4|Entorno|Operación normal del punto de venta con cliente esperando frente al cajero. El sistema puede estar en modo online u offline.|
|5|Respuesta|La venta se registra completamente, el inventario se actualiza, el comprobante se genera y la interfaz confirma el éxito al cajero.|
|6|Medida de Respuesta|Tiempo total desde confirmar hasta ver la confirmación no mayor a 3 segundos.|

### Notas de Arquitectura

|Aspecto|Descripción|
|---|---|
|Justificación de Negocio|El cliente espera de pie frente al cajero; cualquier demora perceptible deteriora la experiencia y se compara negativamente con el método manual anterior.|
|Tácticas Arquitectónicas|En línea, se agrupan las actualizaciones de inventario en pocas consultas. Sin conexión, todo se procesa contra el almacenamiento local, que responde en milisegundos. Totales y descuentos se calculan en el dispositivo sin esperar al servidor.|
|Supuestos y Riesgos|Se asume que 10 productos es un máximo realista. Riesgo: productos que requieren fraccionarse entre varios lotes agregan pasos adicionales; se mitiga precargando los lotes activos por producto.|
|Prueba de Aceptación|Con 10 productos en el carrito, 3 con descuento y 2 que requieren fraccionamiento entre lotes, la venta debe completarse en no más de 3 segundos, online u offline.|

---

## 13. ESC-CAL-REN-0011

Cuando dos usuarios registren ventas simultáneamente el sistema debe procesar las transacciones sin errores ni pérdida de información

| Campo               | Valor                                                               |
| ------------------- | ------------------------------------------------------------------- |
| Atributo de Calidad | Rendimiento                                                         |
| Prioridad           | Alta                                                                |
| Dificultad / Riesgo | Alta                                                                |
| RF relacionado      | RF-004.1 (Registrar venta), RF-011.3 (Conflictos de sincronización) |

### Definición del Escenario

|Parte|Componente|Descripción|
|---|---|---|
|1|Origen|Dos cajeros operando simultáneamente en dos dispositivos distintos del mismo negocio.|
|2|Estímulo|Ambos cajeros registran ventas del mismo producto al mismo tiempo, descontando unidades del mismo lote.|
|3|Artefacto|La capa de persistencia y el servicio de dominio de ventas que gestionan la concurrencia sobre el inventario compartido.|
|4|Entorno|Operación en modo online con ambos dispositivos conectados al servidor.|
|5|Respuesta|Ambas ventas se procesan sin sobreventa, descontando el lote en orden de llegada, con un stock final consistente y sin errores visibles para ninguno de los cajeros.|
|6|Medida de Respuesta|Cada venta procesada en menos de 3 segundos, 0 sobreventas, 0 errores de concurrencia visibles, stock final exacto según ambas ventas.|

### Notas de Arquitectura

|Aspecto|Descripción|
|---|---|
|Justificación de Negocio|Con dos cajeros simultáneos es inevitable vender del mismo producto a la vez; sin buen manejo de concurrencia, el inventario digital se descuadra del físico.|
|Tácticas Arquitectónicas|Control de versión sobre el lote: si dos operaciones lo modifican a la vez, la segunda detecta el conflicto y reintenta automáticamente con el dato actualizado. Sin conexión, el conflicto se resuelve al sincronizar. Las transacciones son breves para minimizar el bloqueo.|
|Supuestos y Riesgos|Se asume que estas ventas concurrentes exactas son poco frecuentes. Riesgo: si el reintento falla repetidamente, la operación debe fallar con un mensaje claro en vez de quedar esperando indefinidamente.|
|Prueba de Aceptación|Con un lote de 10 unidades, si el cajero A vende 3 y el B vende 4 al mismo tiempo del mismo lote, ambas deben completarse, el lote quedar en 3, y el stock reflejar la resta total de 7.|

---

## 14. ESC-CAL-REN-0029

Con una base de datos de 3 años de registros, las consultas de inventario no deben superar 3 segundos

| Campo               | Valor                                                       |
| ------------------- | ----------------------------------------------------------- |
| Atributo de Calidad | Rendimiento                                                 |
| Prioridad           | Alta                                                        |
| Dificultad / Riesgo | Media                                                       |
| RF relacionado      | RF-010.1 (Dashboard inventario), RF-004.2 (Consultar venta) |

### Definición del Escenario

|Parte|Componente|Descripción|
|---|---|---|
|1|Origen|Administrador que consulta el inventario o genera reportes en un negocio con varios años de operación.|
|2|Estímulo|El administrador consulta el inventario completo o genera un reporte de ventas del mes en un sistema con 3 años de datos históricos acumulados.|
|3|Artefacto|La capa de consulta del backend y la base de datos con tablas de productos, lotes, ventas e historial con alto volumen de registros.|
|4|Entorno|Operación normal en modo online con 10.000 productos registrados, más de 1.000 lotes activos y más de 50.000 registros de ventas acumulados.|
|5|Respuesta|Las consultas de inventario y los reportes se generan dentro de los tiempos establecidos sin degradación perceptible respecto a una base recién instalada.|
|6|Medida de Respuesta|Inventario en menos de 3 segundos, reportes mensuales en menos de 5 segundos, búsquedas en menos de 2 segundos, degradación máxima del 30%.|

### Notas de Arquitectura

|Aspecto|Descripción|
|---|---|
|Justificación de Negocio|Un sistema que se vuelve lento con el tiempo pierde adopción y el dueño vuelve a métodos manuales.|
|Tácticas Arquitectónicas|Índices en las columnas de mayor consulta, paginación obligatoria en listados largos, precálculo de agregados para evitar sumas en tiempo real, y posible particionamiento futuro de tablas grandes.|
|Supuestos y Riesgos|Se asume que la base de datos con índices adecuados soporta este volumen. Riesgo: reportes con consultas complejas podrían exceder el umbral; se mitiga con vistas precalculadas y cálculos pesados en segundo plano.|
|Prueba de Aceptación|Con 3 años de datos, 10.000 productos y 50.000 ventas, el inventario debe responder en menos de 3 segundos y el reporte mensual en menos de 5 segundos.|

---

## 15. ESC-CAL-REN-0037

Al modificar la cantidad de un producto en la venta el sistema debe recalcular el total en menos de 1 segundo

| Campo               | Valor                      |
| ------------------- | -------------------------- |
| Atributo de Calidad | Rendimiento                |
| Prioridad           | Alta                       |
| Dificultad / Riesgo | Baja                       |
| RF relacionado      | RF-004.1 (Registrar venta) |

### Definición del Escenario

|Parte|Componente|Descripción|
|---|---|---|
|1|Origen|Cajero que modifica la cantidad de un producto dentro del carrito de venta.|
|2|Estímulo|El cajero cambia la cantidad de un producto de 2 a 5 unidades mientras el carrito ya contiene otros 4 productos con sus subtotales calculados.|
|3|Artefacto|El componente de frontend que gestiona el carrito de venta y los cálculos de subtotales, descuentos y total en tiempo real.|
|4|Entorno|Operación normal del punto de venta con cliente presente.|
|5|Respuesta|El subtotal y el total se recalculan al instante, manteniendo correctamente aplicado cualquier descuento activo.|
|6|Medida de Respuesta|Actualización visible en menos de 1 segundo, sin errores de redondeo mayores a 1 peso.|

### Notas de Arquitectura

|Aspecto|Descripción|
|---|---|
|Justificación de Negocio|El recálculo inmediato evita errores de cobro y descuadres de caja; una demora perceptible se siente "trabada" y frustra al cajero en horas pico.|
|Tácticas Arquitectónicas|Todos los cálculos del carrito ocurren en el dispositivo del cajero, sin consultar al servidor, usando un manejo de decimales preciso. El carrito se guarda de forma permanente solo al confirmar el pago.|
|Supuestos y Riesgos|Se asume que precios y descuentos vigentes están cacheados localmente durante la venta. Riesgo mínimo: un cambio de precio aprobado a mitad de venta no se reflejaría hasta la siguiente.|
|Prueba de Aceptación|Con un carrito de $25.000, al cambiar un producto de $5.000 de 2 a 5 unidades, el total debe pasar a $40.000 en menos de 1 segundo sin recargar la pantalla.|

---