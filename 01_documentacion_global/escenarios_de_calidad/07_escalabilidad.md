## 25. ESC-CAL-ESC-0008

Cuando el negocio decida abrir un segundo punto de venta el sistema debe soportar los datos de ambos puntos sin degradación del rendimiento en ninguno de ellos

| Campo               | Valor                        |
| ------------------- | ---------------------------- |
| Atributo de Calidad | Escalabilidad                |
| Prioridad           | Media                        |
| Dificultad / Riesgo | Baja                         |
| RF relacionado      | RF-013.1 (Registrar negocio) |

### Definición del Escenario

|Parte|Componente|Descripción|
|---|---|---|
|1|Origen|Un dueño que opera un minimercado exitoso y decide abrir un segundo punto de venta usando el sistema.|
|2|Estímulo|El administrador registra un segundo negocio en la plataforma y comienza a operar con catálogo y empleados independientes del primer negocio.|
|3|Artefacto|La arquitectura que gestiona múltiples negocios en la misma infraestructura con aislamiento estricto de datos.|
|4|Entorno|El primer negocio tiene 6 meses de operación con 500 productos, 100 lotes activos y 10.000 ventas históricas.|
|5|Respuesta|El segundo negocio opera de forma completamente independiente: sin datos visibles cruzados entre negocios y sin afectar el rendimiento del primero.|
|6|Medida de Respuesta|0 datos cruzados entre negocios, degradación menor al 10% en el primer negocio, segundo negocio operativo en menos de 10 minutos.|

### Notas de Arquitectura

|Aspecto|Descripción|
|---|---|
|Justificación de Negocio|Soportar múltiples negocios desde el inicio es la base del modelo comercial; si agregar uno afecta al otro, el producto no es escalable comercialmente.|
|Tácticas Arquitectónicas|Todas las entidades del dominio incluyen un identificador de negocio obligatorio, y todas las consultas filtran automáticamente por el negocio de la sesión activa, con índices optimizados para ese filtro. El backend puede escalar horizontalmente sin dependencia entre negocios.|
|Supuestos y Riesgos|Se asume independencia total entre negocios en esta etapa. Riesgo: una carga anormalmente alta de un negocio podría afectar a otros en la misma infraestructura; se mitiga con límites de consumo por negocio.|
|Prueba de Aceptación|Al registrar un negocio B junto a un negocio A con 6 meses de historia, B debe operar con su propio catálogo, empleados y ventas sin ver datos de A, y los tiempos de respuesta de A deben mantenerse dentro de los umbrales definidos.|