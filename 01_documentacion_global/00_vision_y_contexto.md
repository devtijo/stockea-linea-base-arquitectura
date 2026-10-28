# STOCKEA — Visión y contexto

## Visión del producto

STOCKEA es un punto de venta para tiendas de barrio y minimercados colombianos que funciona con o sin conexión a internet. El cajero sigue vendiendo aunque se caiga el WiFi, y todo se sincroniza solo al reconectar. No es una funcionalidad de respaldo: la operación offline es el modo normal de trabajo del sistema.

El diferenciador real es el motor de recomendaciones de compra. Cruza velocidad de venta, stock disponible y fechas de vencimiento para decirle al dueño qué comprar, cuánto y cuándo, antes de que el producto se pierda en la bodega. Las recomendaciones son explicables: "vendes 10 al día, quieres 7 días de cobertura, hoy tienes 20".

Se comercializa como SaaS con suscripción mensual escalonada, instalable como PWA desde el navegador y sin aplicación nativa. La meta de costo es USD 50 mensuales de infraestructura por cada 100 negocios activos, lo que condiciona cada decisión de arquitectura.

## El problema

Miles de tiendas de barrio venden perecederos sin ningún sistema de apoyo y pierden dinero por dos vías simultáneas:

- **Pierden ventas** cuando se cae el internet, porque un POS que exige conexión deja de funcionar justo cuando hay clientes esperando.
- **Pierden mercancía** porque nadie les avisa a tiempo qué está por vencer ni cuánto deben comprar de cada producto.

Los sistemas existentes resuelven uno u otro problema, rara vez los dos, y casi ninguno está pensado para conectividad intermitente ni para el presupuesto del pequeño comercio.

## Roles del sistema

Los tres roles son **acumulativos**: Cajero ⊂ Supervisor ⊂ Administrador. Un único usuario Administrador puede operar una tienda de un solo empleado sin perder funcionalidad (ADR-005).

| Rol | Responsabilidad | Interacción principal |
|---|---|---|
| **Cajero** | Operación diaria: registra ventas, aperturas y cierres de caja | Usa el Frontend (PWA) y **opera offline** |
| **Supervisor** | Control del inventario: entradas de mercancía, alertas de vencimiento, ajustes y resolución de discrepancias | Usa el Frontend; resuelve la cola de discrepancias (ADR-002) |
| **Administrador** | Decisiones estratégicas: dashboards, reportes, lista de reposición y gestión de la suscripción | Usa el Frontend; por roles acumulativos puede hacer todo lo del Supervisor y el Cajero |
