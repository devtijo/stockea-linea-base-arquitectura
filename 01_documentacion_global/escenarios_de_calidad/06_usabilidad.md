## 23. ESC-CAL-USA-0001

Cuando un cajero nuevo use el sistema por primera vez debe poder completar el registro de una venta de un solo producto en menos de 30 minutos sin necesidad de un manual

| Campo               | Valor                      |
| ------------------- | -------------------------- |
| Atributo de Calidad | Usabilidad                 |
| Prioridad           | Crítica                    |
| Dificultad / Riesgo | Media                      |
| RF relacionado      | RF-004.1 (Registrar venta) |

### Definición del Escenario

|Parte|Componente|Descripción|
|---|---|---|
|1|Origen|Cajero nuevo sin experiencia previa con el sistema ni con software de gestión en general.|
|2|Estímulo|El dueño le pide al cajero nuevo que registre una venta de un solo producto el primer día de trabajo.|
|3|Artefacto|La interfaz completa del módulo de ventas: login, selección de turno, búsqueda de producto, carrito, confirmación de pago y comprobante.|
|4|Entorno|Primer uso del sistema por parte del cajero. No ha recibido capacitación previa ni ha leído un manual.|
|5|Respuesta|El cajero logra iniciar sesión, abrir el turno, buscar el producto, agregarlo al carrito, confirmar el pago y generar el comprobante sin ayuda externa ni errores críticos.|
|6|Medida de Respuesta|Primera venta completada en menos de 30 minutos, no más de 5 clics una vez abierto el turno, ninguna pantalla requiere conocimientos técnicos.|

### Notas de Arquitectura

|Aspecto|Descripción|
|---|---|
|Justificación de Negocio|La alta rotación de personal en tiendas de barrio hace que una curva de aprendizaje larga sea un bloqueador real de adopción del sistema.|
|Tácticas Arquitectónicas|Flujo de venta lineal sin bifurcaciones, con lenguaje cotidiano en vez de términos técnicos, mensajes de error claros con la acción correctiva, y botones principales bien diferenciados por color y tamaño.|
|Supuestos y Riesgos|Se asume que el cajero sabe leer y usar un teclado y un mouse. Riesgo: dificultades de lectoescritura podrían extender el tiempo; se mitiga acompañando todo texto clave con iconos.|
|Prueba de Aceptación|Un cajero sin capacitación previa debe lograr registrar una venta completa de un producto en menos de 30 minutos, sin ayuda ni bloqueos por falta de comprensión.|

---

## 24. ESC-CAL-USA-0017

Cuando el sistema detecte productos próximos a vencer debe mostrar la alerta con el nombre del producto, los días restantes y una acción sugerida directamente visible en el dashboard

| Campo               | Valor                                                          |
| ------------------- | -------------------------------------------------------------- |
| Atributo de Calidad | Usabilidad                                                     |
| Prioridad           | Alta                                                           |
| Dificultad / Riesgo | Media                                                          |
| RF relacionado      | RF-008.4 (Alerta vencimiento), RF-010.1 (Dashboard inventario) |

### Definición del Escenario

|Parte|Componente|Descripción|
|---|---|---|
|1|Origen|El sistema detecta que un lote de queso tiene 3 días para vencer con 15 unidades sin vender.|
|2|Estímulo|El administrador abre el dashboard del sistema al inicio del día.|
|3|Artefacto|El componente de alertas del dashboard que muestra notificaciones priorizadas con información accionable.|
|4|Entorno|Operación normal al inicio de la jornada comercial.|
|5|Respuesta|El dashboard muestra la alerta con producto, días restantes, unidades y valor en riesgo, y una acción sugerida visible, priorizada por color según urgencia frente a otras alertas.|
|6|Medida de Respuesta|El administrador identifica el producto y la acción sugerida en menos de 10 segundos, sin navegar a otra pantalla, con alertas ordenadas de mayor a menor urgencia.|

### Notas de Arquitectura

|Aspecto|Descripción|
|---|---|
|Justificación de Negocio|Las alertas convierten datos operativos en decisiones; si no son claras y accionables de un vistazo, el dueño las ignora y el producto se vence.|
|Tácticas Arquitectónicas|Cada alerta muestra en un solo componente el producto, días restantes, cantidad y valor en riesgo, con un botón de acción directa. Se usan colores tipo semáforo según urgencia, acompañados de iconos para accesibilidad, y el botón lleva directo a la acción sin pasos intermedios.|
|Supuestos y Riesgos|Se asume que el administrador revisa el dashboard al menos una vez al día. Riesgo: si no lo revisa, la alerta pasa desapercibida; se contempla a futuro un aviso por otro canal.|
|Prueba de Aceptación|Con un lote de queso a 3 días de vencer y 15 unidades disponibles, el dashboard debe mostrar la alerta con su nivel de urgencia, producto, días restantes, unidades, valor y un botón de acción visible sin necesidad de scroll ni navegación.|

---
