## 19. ESC-CAL-AUD-0005

Cuando un cajero confirme una venta el sistema debe guardar automáticamente su identificador, la fecha, la hora y el listado de productos vendidos con cantidades y precios

| Campo               | Valor                       |
| ------------------- | --------------------------- |
| Atributo de Calidad | Capacidad para ser auditado |
| Prioridad           | Crítica                     |
| Dificultad / Riesgo | Baja                        |
| RF relacionado      | RF-004.1 (Registrar venta)  |

### Definición del Escenario

|Parte|Componente|Descripción|
|---|---|---|
|1|Origen|Cajero que confirma el pago de una venta en el punto de venta.|
|2|Estímulo|El cajero registra una venta de 3 productos con método de pago efectivo y confirma la transacción.|
|3|Artefacto|El servicio de dominio de ventas que persiste la transacción con todos los metadatos de auditoría (quién, cuándo, qué, cómo).|
|4|Entorno|Operación normal del punto de venta en modo online u offline.|
|5|Respuesta|El sistema guarda automáticamente el cajero responsable, la fecha y hora exacta, el turno asociado, el detalle de cada producto (con lote, precio, descuento y subtotal), método de pago, total y cambio, sin que el cajero deba ingresar nada adicional.|
|6|Medida de Respuesta|100% de ventas con auditoría completa, 0 ventas sin cajero o sin fecha/hora, sin agregar más de 100 milisegundos al proceso.|

### Notas de Arquitectura

|Aspecto|Descripción|
|---|---|
|Justificación de Negocio|El dueño necesita saber quién vendió qué, cuándo y a qué precio para controlar el manejo del dinero; sin trazabilidad automática, el sistema no sirve como herramienta de control.|
|Tácticas Arquitectónicas|Los campos de auditoría se asignan automáticamente a partir de la sesión activa y del reloj del sistema, nunca de datos enviados por el dispositivo del usuario. Sin conexión se usa la fecha local, validada por el servidor al sincronizar.|
|Supuestos y Riesgos|Se asume que la sesión identifica correctamente al usuario y al negocio. Riesgo: un reloj local desconfigurado en modo offline podría registrar una hora incorrecta; se mitiga validando la coherencia temporal al sincronizar.|
|Prueba de Aceptación|Tras confirmar una venta de 3 productos, el historial del día debe mostrar cajero, hora exacta, productos con cantidades, precios y descuentos, método de pago y total, sin que el cajero haya digitado nada de eso.|

---

## 20. ESC-CAL-AUD-0016

Cuando un cajero abra la caja el sistema debe registrar su usuario, la fecha, la hora y el monto inicial declarado sin posibilidad de modificación posterior

| Campo               | Valor                       |
| ------------------- | --------------------------- |
| Atributo de Calidad | Capacidad para ser auditado |
| Prioridad           | Crítica                     |
| Dificultad / Riesgo | Baja                        |
| RF relacionado      | RF-005.1 (Apertura de caja) |

### Definición del Escenario

|Parte|Componente|Descripción|
|---|---|---|
|1|Origen|Cajero que inicia su turno de trabajo y registra la apertura de caja.|
|2|Estímulo|El cajero abre la caja declarando un monto inicial de $200.000 en efectivo.|
|3|Artefacto|El servicio de dominio de turnos de caja que persiste el registro de apertura con todos los campos de auditoría marcados como inmodificables.|
|4|Entorno|Operación normal del punto de venta al inicio del turno.|
|5|Respuesta|El sistema registra la apertura con usuario, fecha, hora y monto inicial, y ese registro queda inmediatamente imposible de modificar, incluso para el administrador.|
|6|Medida de Respuesta|100% de registros de apertura completos, 0 registros modificados después de creados, cualquier intento de cambio es rechazado.|

### Notas de Arquitectura

|Aspecto|Descripción|
|---|---|
|Justificación de Negocio|El monto inicial es la referencia para calcular la diferencia al cierre; si pudiera modificarse después, un cajero con faltante podría ocultarlo, y un cajero honesto quedaría desprotegido ante acusaciones falsas.|
|Tácticas Arquitectónicas|No existe forma de editar ni eliminar este registro una vez creado, ni desde la interfaz ni directamente sobre la base de datos: los permisos de modificación están deshabilitados a ese nivel. Cualquier corrección exige cerrar el turno y abrir uno nuevo.|
|Supuestos y Riesgos|Se asume que el cajero declara el monto real. Riesgo: el sistema no puede verificar si el monto declarado es falso; esto es una limitación del modelo de negocio, mitigable con verificaciones físicas periódicas del dueño.|
|Prueba de Aceptación|Tras abrir la caja con $200.000, cualquier intento de modificar el monto (por interfaz, API o base de datos) debe rechazarse y el registro debe conservar sus valores originales.|

---

## 21. ESC-CAL-AUD-0021

Cuando un administrador intente modificar o eliminar una venta confirmada o un ajuste de inventario, el sistema debe rechazarlo y exigir un registro correctivo que referencie al original

| Campo               | Valor                           |
| ------------------- | ------------------------------- |
| Atributo de Calidad | Capacidad para ser auditado     |
| Prioridad           | Crítica                         |
| Dificultad / Riesgo | Baja                            |
| RF relacionado      | RNF-017 (Registro de auditoría) |

### Definición del Escenario

|Parte|Componente|Descripción|
|---|---|---|
|1|Origen|Cualquier usuario del sistema, incluido el administrador con el rol de mayor privilegio.|
|2|Estímulo|El administrador intenta modificar o eliminar una venta ya confirmada, un ajuste de inventario ya registrado o un registro del historial de precios.|
|3|Artefacto|La capa de persistencia que protege la inmutabilidad de los registros de auditoría a nivel de aplicación y de base de datos.|
|4|Entorno|Operación normal del sistema en modo online.|
|5|Respuesta|El sistema rechaza la operación, informa que los registros históricos son inmodificables por política del sistema, y registra el intento con usuario, fecha y hora. Cualquier corrección debe hacerse mediante un nuevo registro que referencie al original.|
|6|Medida de Respuesta|0 registros de auditoría modificados o eliminados, 100% de intentos rechazados y registrados, mensaje de rechazo en menos de 1 segundo.|

### Notas de Arquitectura

|Aspecto|Descripción|
|---|---|
|Justificación de Negocio|La trazabilidad legal exige conservar registros mercantiles inalterados por años; permitir borrarlos destruiría la credibilidad del sistema como herramienta de auditoría.|
|Tácticas Arquitectónicas|No existen funciones de edición ni eliminación para estas entidades en ningún nivel del sistema, incluyendo los permisos de base de datos del usuario de la aplicación. Las correcciones se modelan como nuevos registros que referencian al original con un motivo explícito.|
|Supuestos y Riesgos|Se asume que no hay acceso directo a producción por fuera de la aplicación. Riesgo: un acceso directo a la base de datos podría modificar registros; se mitiga restringiendo y auditando ese acceso.|
|Prueba de Aceptación|Un administrador que intente eliminar una venta antigua o modificar un ajuste pasado debe recibir el rechazo en ambos casos, con el mensaje de inmutabilidad y el intento registrado.|

---

## 22. ESC-CAL-AUD-0027

Cuando el dueño consulte el historial de un lote específico, debe ver su entrada, ventas, ajustes, descuentos y estado actual

| Campo               | Valor                                       |
| ------------------- | ------------------------------------------- |
| Atributo de Calidad | Capacidad para ser auditado                 |
| Prioridad           | Alta                                        |
| Dificultad / Riesgo | Media                                       |
| RF relacionado      | RF-002.4 (Consultar movimientos inventario) |

### Definición del Escenario

|Parte|Componente|Descripción|
|---|---|---|
|1|Origen|Administrador que necesita investigar por qué un lote de yogurt llegó a vencimiento con unidades sin vender.|
|2|Estímulo|El administrador consulta la trazabilidad completa de un lote específico desde su ingreso hasta su estado actual.|
|3|Artefacto|El módulo de consulta de historial que cruza entradas de mercancía, lotes, ventas, ajustes y descuentos para reconstruir el ciclo de vida completo de un lote.|
|4|Entorno|Operación normal del sistema en modo online.|
|5|Respuesta|El sistema muestra el ciclo completo del lote: entrada (proveedor, cantidad, costo, vencimiento, responsable), ventas que descontaron de él, ajustes realizados, descuentos aplicados y estado actual.|
|6|Medida de Respuesta|100% de operaciones visibles sin faltantes, respuesta en menos de 3 segundos, la suma de salidas más el saldo actual coincide con la cantidad inicial del lote.|

### Notas de Arquitectura

|Aspecto|Descripción|
|---|---|
|Justificación de Negocio|Entender por qué un lote se venció con unidades restantes permite mejorar futuras decisiones de compra; sin esta trazabilidad, las pérdidas por vencimiento quedan sin explicación.|
|Tácticas Arquitectónicas|Cada venta y ajuste queda vinculado al lote específico, permitiendo reconstruir el historial con una sola consulta. La cantidad inicial del lote nunca cambia, sirviendo de referencia fija para validar consistencia.|
|Supuestos y Riesgos|Se asume que todas las operaciones referencian correctamente su lote. Riesgo: un error en la lógica de descuento por vencimiento mostraría datos inconsistentes; se mitiga validando que la cantidad inicial siempre cuadre con lo disponible más lo salido.|
|Prueba de Aceptación|Para un lote con ventas, un ajuste por daño y unidades restantes, la consulta debe mostrar todas las ventas con cajero y fecha, el ajuste con su motivo y responsable, y el estado actual con la fecha de vencimiento.|

---

