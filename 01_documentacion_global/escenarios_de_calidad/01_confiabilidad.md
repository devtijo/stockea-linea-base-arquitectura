## 1. ESC-CAL-CON-0001

Cuando ocurra un corte de luz durante el registro de una venta el sistema debe garantizar que al reiniciar no queden datos a medias ni inventario desactualizado

| Campo               | Valor                                                    |
| ------------------- | -------------------------------------------------------- |
| Atributo de Calidad | Confiabilidad                                            |
| Prioridad           | Crítica                                                  |
| Dificultad / Riesgo | Alta                                                     |
| RF relacionado      | RF-004.1 (Registrar venta), RF-011.1 (Operación offline) |

### Definición del Escenario

| Parte | Componente          | Descripción                                                                                                                                                                                            |
| ----- | ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1     | Origen              | Evento externo: corte de energía eléctrica en el local del negocio durante el horario operativo.                                                                                                       |
| 2     | Estímulo            | Se interrumpe la alimentación eléctrica del equipo mientras el cajero está registrando una venta con 5 productos, 3 de los cuales ya habían sido agregados al carrito.                                 |
| 3     | Artefacto           | El módulo de ventas del frontend (almacenamiento local del dispositivo) y la capa de persistencia del backend (base de datos) que gestionan la transacción de venta y la actualización del inventario. |
| 4     | Entorno             | Operación normal del punto de venta durante horario comercial. La venta puede estar en modo online u offline al momento del corte.                                                                     |
| 5     | Respuesta           | Al reiniciar el equipo, la venta inconclusa no queda registrada y el inventario vuelve exactamente al estado previo al intento: sin descuentos parciales ni ventas a medias.                           |
| 6     | Medida de Respuesta | 0% de impacto en inventario, 0 registros de venta corruptos, sistema operativo en menos de 30 segundos tras el reinicio.                                                                               |

### Notas de Arquitectura

| Aspecto                  | Descripción                                                                                                                                                                                                                                                   |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Justificación de Negocio | Una venta a medias descuadra el inventario digital contra el físico sin forma de corregirlo, lo que destruye la confianza del dueño en el sistema.                                                                                                            |
| Tácticas Arquitectónicas | La venta y el ajuste de stock ocurren en una sola transacción atómica (todo o nada). Localmente, la venta solo se guarda al confirmar el pago, no al armar el carrito. Al iniciar, el sistema descarta cualquier operación local que haya quedado incompleta. |
| Supuestos y Riesgos      | Se asume que el almacenamiento local escribe de forma durable antes de confirmar. Riesgo de registro corrupto si el corte ocurre justo en el instante de escritura; se mitiga verificando la integridad al reiniciar.                                         |
| Prueba de Aceptación     | Dado un cajero vendiendo offline y un corte de energía después de agregar 3 de 5 productos pero antes de pagar, al reiniciar el inventario debe quedar igual al previo y no debe existir esa transacción.                                                     |

---

## 2. ESC-CAL-CON-0003

Cuando se registre una entrada de mercancía y ocurra un error de sistema el stock no debe actualizarse parcialmente la operación debe completarse o revertirse en su totalidad

| Campo               | Valor                                     |
| ------------------- | ----------------------------------------- |
| Atributo de Calidad | Confiabilidad                             |
| Prioridad           | Crítica                                   |
| Dificultad / Riesgo | Media                                     |
| RF relacionado      | RF-002.1 (Registrar entrada de mercancía) |

### Definición del Escenario

| Parte | Componente          | Descripción                                                                                                                                                                                                                 |
| ----- | ------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1     | Origen              | Error interno del sistema durante el procesamiento de una operación de entrada de mercancía.                                                                                                                                |
| 2     | Estímulo            | El supervisor confirma una entrada de 50 unidades de leche del proveedor con una fecha de vencimiento y el sistema falla a mitad del procesamiento después de crear el lote pero antes de actualizar el stock del producto. |
| 3     | Artefacto           | El servicio de dominio de entradas de mercancía y la base de datos que gestionan la creación del lote y la actualización del stock del producto en una misma transacción.                                                   |
| 4     | Entorno             | Operación normal del sistema en modo online.                                                                                                                                                                                |
| 5     | Respuesta           | La operación se revierte por completo: no queda lote ni entrada registrados, el stock no cambia, y el sistema informa que debe reintentarse.                                                                                |
| 6     | Medida de Respuesta | 0 registros huérfanos en la base de datos, stock idéntico al previo, mensaje de error en menos de 2 segundos.                                                                                                               |

### Notas de Arquitectura

| Aspecto                  | Descripción                                                                                                                                                                                                                |
| ------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Justificación de Negocio | Un lote sin stock actualizado dispara alertas y recomendaciones erróneas; un stock sin lote impide rastrear vencimientos, anulando el diferenciador del producto.                                                          |
| Tácticas Arquitectónicas | Transacción única que envuelve la creación de la entrada, del lote y la actualización del stock, con reversión automática ante cualquier error. Las restricciones de la base de datos actúan como última línea de defensa. |
| Supuestos y Riesgos      | Se asume reversión correcta ante fallos. Riesgo: fallo de conexión que deje la transacción en estado indeterminado, requiriendo verificación al reconectar.                                                                |
| Prueba de Aceptación     | Si el sistema falla tras crear el lote pero antes de actualizar el stock, no debe existir ni lote ni entrada, y el stock debe permanecer igual al anterior.                                                                |


---

## 3. ESC-CAL-CON-0010

Cuando el sistema calcule los días restantes de vencimiento de un lote el resultado debe coincidir con la diferencia real entre la fecha actual y la fecha de vencimiento registrada

| Campo               | Valor                                                                           |
| ------------------- | ------------------------------------------------------------------------------- |
| Atributo de Calidad | Confiabilidad                                                                   |
| Prioridad           | Crítica                                                                         |
| Dificultad / Riesgo | Media                                                                           |
| RF relacionado      | RF-008.4 (Alerta vencimiento próximo), RF-009.3 (Riesgo vencimiento anticipado) |

### Definición del Escenario

| Parte | Componente          | Descripción                                                                                                                                                                                |
| ----- | ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1     | Origen              | El reloj interno del sistema al ejecutar el cálculo diario de días restantes para todos los lotes activos del negocio.                                                                     |
| 2     | Estímulo            | El sistema calcula los días restantes de vencimiento para múltiples lotes con fechas distintas en diferentes momentos del día, incluyendo el caso borde del día exacto de vencimiento.     |
| 3     | Artefacto           | El servicio de dominio que calcula los días restantes y determina el estado automático del lote (disponible, próximo a vencer, vencido, agotado).                                          |
| 4     | Entorno             | Operación normal del sistema. El cálculo puede ejecutarse a cualquier hora y debe ser consistente sin importar el momento.                                                                 |
| 5     | Respuesta           | Los días restantes reflejan siempre la diferencia real entre la fecha de vencimiento y la fecha actual, y el estado del lote se actualiza automáticamente según los umbrales configurados. |
| 6     | Medida de Respuesta | 100% de coincidencia entre el cálculo y la diferencia real de fechas, 0 lotes con valores o estados inconsistentes.                                                                        |

### Notas de Arquitectura

| Aspecto                  | Descripción                                                                                                                                                                                                              |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Justificación de Negocio | Este cálculo es la base de todo el motor de alertas y recomendaciones; un error de un solo día puede generar alertas falsas o dejar pasar un vencimiento real.                                                           |
| Tácticas Arquitectónicas | Pruebas exhaustivas de casos borde (día exacto de vencimiento, año bisiesto, cambio de hora). Las fechas se manejan en un formato estándar y se comparan solo por día calendario, sin hora, para evitar inconsistencias. |
| Supuestos y Riesgos      | Se asume el reloj del servidor sincronizado. Riesgo: equipo offline con fecha mal configurada; se corrige al sincronizar recalculando con la fecha real del servidor.                                                    |
| Prueba de Aceptación     | Un lote que vence el 2026-08-15 consultado el 2026-08-13 a distintas horas del día debe mostrar siempre 2 días restantes y el estado correspondiente, y aparecer como vencido el día 2026-08-16.                         |

---

## 4. ESC-CAL-CON-0014

Cuando se confirme una venta de 3 unidades de un producto con stock de 10 el sistema debe mostrar inmediatamente un stock de 7 sin necesidad de recargar la página

| Campo               | Valor                      |
| ------------------- | -------------------------- |
| Atributo de Calidad | Confiabilidad              |
| Prioridad           | Crítica                    |
| Dificultad / Riesgo | Media                      |
| RF relacionado      | RF-004.1 (Registrar venta) |

### Definición del Escenario

| Parte | Componente          | Descripción                                                                                                                                                                                                                                       |
| ----- | ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1     | Origen              | Cajero que confirma el pago de una venta desde el punto de venta.                                                                                                                                                                                 |
| 2     | Estímulo            | Se confirma una venta de 3 unidades de un producto que tiene 10 unidades de stock disponible distribuidas en un lote con vencimiento próximo (7 unidades) y otro lote más lejano (3 unidades).                                                    |
| 3     | Artefacto           | El servicio de dominio de ventas que asigna contablemente el descuento de stock por lote siguiendo la regla de vencimiento más próximo primero y la actualización del stock calculado del producto.                                               |
| 4     | Entorno             | Operación normal del punto de venta en modo online.                                                                                                                                                                                               |
| 5     | Respuesta           | El sistema asigna contablemente el descuento al lote más próximo (FEFO), para efectos de auditoría y trazabilidad, aunque físicamente el cliente haya cogido otro lote y actualiza el stock total del producto de forma inmediata en la interfaz. |
| 6     | Medida de Respuesta | Actualización de stock en menos de 2 segundos, el lote asignado en la base de datos es el más próximo, el stock total siempre coincide con la suma de los lotes activos.                                                                          |

### Notas de Arquitectura

| Aspecto                  | Descripción                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| ------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Justificación de Negocio | Vender primero lo más próximo a vencer reduce la merma. Sin embargo, como se documenta en el análisis del diferenciador, en una tienda de barrio el sistema **no puede controlar físicamente** qué lote toma el cliente. Por lo tanto, el sistema realiza una **asignación contable FEFO** para mantener la trazabilidad de auditoría y el stock total, mientras que la **rotación física real** es responsabilidad del supervisor, asistido por el motor de alertas                                                                                                                                                                    |
| Tácticas Arquitectónicas | Los lotes se consultan ordenados por fecha de vencimiento más próxima para la **asignación contable** del descuento. Si la cantidad excede un lote, se fracciona automáticamente entre varios (ADR-003). Toda la operación (venta, ajuste de lotes y recálculo de stock) ocurre en una sola transacción atómica (ADR-009) y se registra en el historial de movimientos (ADR-004).                                                                                                                                                                                                                                                       |
| Supuestos y Riesgos      | **Supuesto:** Se asume que la asignación contable FEFO refleja el comportamiento esperado, aunque no garantiza la realidad física.                  **Riesgo 1:** El cliente puede tomar un lote diferente al asignado, lo que genera una diferencia entre el stock físico y el contable, la cual debe ser corregida por el supervisor mediante un ajuste manual (ADR-002).                              **Riesgo 2:** Sobreventa si dos cajeros venden del mismo lote a la vez sin control de concurrencia (mitigado con bloqueo pesimista `FOR UPDATE` en el flujo online y con reasignación FEFO/discrepancias en el flujo offline). |
| Prueba de Aceptación     | Con un lote A de 7 unidades (vence antes) y un lote B de 3, al vender 3 unidades el lote A debe quedar en 4 en la **base de datos contable**, el B intacto, y el stock total en 7, reflejado en menos de 2 segundos. La verificación es a nivel de datos, no de estantería                                                                                                                                                                                                                                                                                                                                                              |

---

## 5. ESC-CAL-CON-0025

Cuando el dueño apruebe un descuento del 20% sobre un producto el cajero debe ver el precio descontado de forma automática al momento de registrar la venta sin necesidad de calcularlo manualmente

| Campo               | Valor                                                    |
| ------------------- | -------------------------------------------------------- |
| Atributo de Calidad | Confiabilidad                                            |
| Prioridad           | Alta                                                     |
| Dificultad / Riesgo | Media                                                    |
| RF relacionado      | RF-006.2 (Aprobar descuento), RF-004.1 (Registrar venta) |


### Definición del Escenario

|Parte|Componente|Descripción|
|---|---|---|
|1|Origen|Administrador que aprueba un descuento del 20% sobre un lote de yogurt próximo a vencer.|
|2|Estímulo|El cajero registra una venta del producto que tiene un descuento activo vigente.|
|3|Artefacto|El módulo de ventas que consulta descuentos activos por lote y aplica automáticamente el porcentaje aprobado al precio unitario al agregar el producto al carrito.|
|4|Entorno|Operación normal del punto de venta. El descuento fue aprobado previamente por el administrador y está vigente.|
|5|Respuesta|El precio con descuento se aplica automáticamente sin que el cajero pueda ignorarlo, y el comprobante refleja el precio original, el porcentaje y el precio final.|
|6|Medida de Respuesta|Precio con descuento visible en menos de 1 segundo, diferencia máxima de 1 peso por redondeo, 100% de las ventas con descuento activo reflejadas en el comprobante.|

### Notas de Arquitectura

|Aspecto|Descripción|
|---|---|
|Justificación de Negocio|Si el descuento no se aplica solo, el producto expira sin venderse; si se aplica mal, el negocio pierde más de lo previsto. Ambos casos destruyen la confianza en el motor de descuentos.|
|Tácticas Arquitectónicas|Al agregar el producto se verifica si el lote tiene un descuento vigente y se aplica automáticamente, sin intervención del cajero, usando un cálculo decimal preciso. El descuento queda congelado en el detalle de la venta.|
|Supuestos y Riesgos|Se asume un único descuento activo por lote a la vez. Riesgo: si el descuento se aprueba con el producto ya en el carrito, se decidió aplicar el precio vigente solo al confirmar la venta, no al agregarlo.|
|Prueba de Aceptación|Con un producto de $5.000 y descuento activo del 20%, al agregarlo el precio mostrado debe ser $4.000, el comprobante debe indicarlo, y el inventario debe descontar del lote correspondiente.|

---

## 6. ESC-CAL-CON-0029

Cuando el sistema genere una recomendación de compra la velocidad de venta calculada debe basarse en las ventas reales del período configurado sin incluir devoluciones ni ajustes negativos

| Campo               | Valor                                                         |
| ------------------- | ------------------------------------------------------------- |
| Atributo de Calidad | Confiabilidad                                                 |
| Prioridad           | Crítica                                                       |
| Dificultad / Riesgo | Alta                                                          |
| RF relacionado      | RF-009.1 (Velocidad de venta), RF-009.4 (Lista de reposición) |

### Definición del Escenario

|Parte|Componente|Descripción|
|---|---|---|
|1|Origen|Proceso automático del sistema que genera recomendaciones de compra periódicamente.|
|2|Estímulo|El sistema ejecuta el cálculo de recomendaciones de compra para todos los productos activos del negocio basándose en ventas, stock actual y vencimientos.|
|3|Artefacto|El motor de recomendaciones que cruza historial de ventas, stock actual y fecha de vencimiento de los lotes para generar la lista de reposición priorizada.|
|4|Entorno|Operación normal del sistema en modo online con datos sincronizados.|
|5|Respuesta|La velocidad de venta se calcula solo con datos reales recientes; los productos en riesgo de vencer antes de venderse no generan recomendación de compra, sino que se envían al motor de descuentos; los productos sin historial muestran "sin datos suficientes" en vez de un número.|
|6|Medida de Respuesta|100% de recomendaciones basadas en datos recientes, 0 recomendaciones contradictorias con productos por vencer, 0 errores por falta de datos.|

### Notas de Arquitectura

|Aspecto|Descripción|
|---|---|
|Justificación de Negocio|Las recomendaciones son el diferenciador principal del producto; datos desactualizados llevan a sobre-stock o desabastecimiento, y recomendar comprar algo que ya se está venciendo destruiría la credibilidad del sistema.|
|Tácticas Arquitectónicas|El cálculo usa solo ventas del período configurado y devuelve un estado descriptivo cuando no hay datos suficientes. Antes de recomendar, se valida si el stock alcanza a venderse antes del vencimiento del lote más próximo; si no alcanza, se redirige al motor de descuentos. El stock usado se congela al generar la recomendación para poder reproducirla después.|
|Supuestos y Riesgos|Se asume un período de cálculo configurable (7 días por defecto), lo que puede subestimar productos de venta estacional. Riesgo adicional con datos offline no sincronizados que generen recomendaciones incompletas.|
|Prueba de Aceptación|Un producto con stock que cubre pocos días de venta y un lote por vencer en más días no debe generar recomendación de compra; uno con stock que no alcanza a venderse antes del vencimiento debe redirigirse al motor de descuentos.|

---
