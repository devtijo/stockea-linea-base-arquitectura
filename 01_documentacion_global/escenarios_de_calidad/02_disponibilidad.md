
## 7. ESC-CAL-DIS-0001

Cuando el negocio pierda la conexión a internet el sistema debe continuar permitiendo registrar ventas, consultar inventario y realizar ajustes sin ninguna interrupción visible

| Campo               | Valor                        |
| ------------------- | ---------------------------- |
| Atributo de Calidad | Disponibilidad               |
| Prioridad           | Crítica                      |
| Dificultad / Riesgo | Muy Alta                     |
| RF relacionado      | RF-011.1 (Operación offline) |

### Definición del Escenario

| Parte | Componente          | Descripción                                                                                                                                                                                  |
| ----- | ------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1     | Origen              | Fallo de conectividad: el proveedor de internet del negocio sufre una interrupción prolongada durante el horario comercial.                                                                  |
| 2     | Estímulo            | El sistema pierde la conexión a internet mientras hay un turno de caja abierto con clientes esperando para pagar.                                                                            |
| 3     | Artefacto           | La aplicación completa: mecanismo de sincronización en segundo plano, almacenamiento local con catálogo de productos, lotes activos, precios vigentes y descuentos activos cacheados.        |
| 4     | Entorno             | El sistema venía operando en modo online y pierde la conectividad de forma inesperada.                                                                                                       |
| 5     | Respuesta           | El sistema sigue funcionando sin interrupción visible: las ventas se registran localmente, el inventario local se actualiza en tiempo real, y un indicador discreto muestra el modo offline. |
| 6     | Medida de Respuesta | 0 segundos de interrupción al perder la conexión, 100% de las funciones esenciales operativas offline, ninguna venta offline se pierde.                                                      |

### Notas de Arquitectura

| Aspecto                  | Descripción                                                                                                                                                                                                                                                                                          |
| ------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Justificación de Negocio | La operación offline es la promesa principal del producto frente a la competencia; en una tienda con conectividad intermitente, exigir internet para vender significa perder ventas reales.                                                                                                          |
| Tácticas Arquitectónicas | Arquitectura offline-first: el frontend opera siempre contra el almacenamiento local como fuente de verdad inmediata. Los datos maestros se precargan al iniciar sesión. Las operaciones de escritura se encolan localmente con fecha y hora y se sincronizan automáticamente al recuperar conexión. |
| Supuestos y Riesgos      | Se asume catálogo local actualizado al momento de perder conexión. Riesgo principal: dos dispositivos vendiendo offline del mismo lote generan sobreventa al sincronizar (ver ESC-CAL-DIS-0008). Riesgo secundario de datos obsoletos tras varios días sin sincronizar.                              |
| Prueba de Aceptación     | Al desconectar la red con un turno abierto, el cajero debe poder vender, consultar inventario y cerrar caja sin interrupción visible, con un indicador discreto de modo offline.                                                                                                                     |

---

## 8. ESC-CAL-DIS-0005

Cuando se restablezca la conexión a internet después de operar en modo offline el sistema debe iniciar la sincronización automáticamente sin que el usuario tenga que hacer nada

| Campo               | Valor                              |
| ------------------- | ---------------------------------- |
| Atributo de Calidad | Disponibilidad                     |
| Prioridad           | Crítica                            |
| Dificultad / Riesgo | Alta                               |
| RF relacionado      | RF-011.2 (Sincronizar operaciones) |

### Definición del Escenario

| Parte | Componente          | Descripción                                                                                                                                                                    |
| ----- | ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1     | Origen              | Evento de red: el sistema detecta que la conexión a internet se ha restablecido después de un período de operación offline.                                                    |
| 2     | Estímulo            | El equipo recupera la conectividad después de 2 horas de operación offline durante las cuales se registraron 15 ventas, 2 ajustes de inventario y 1 cierre de caja.            |
| 3     | Artefacto           | El módulo de sincronización del frontend y la API de sincronización del backend que reciben, validan y persisten las operaciones encoladas.                                    |
| 4     | Entorno             | El sistema transiciona de modo offline a modo online. El cajero puede estar atendiendo clientes al momento de la reconexión.                                                   |
| 5     | Respuesta           | La sincronización inicia sola en segundo plano, envía las operaciones en orden cronológico sin interrumpir al cajero, y avisa al finalizar que los datos quedaron respaldados. |
| 6     | Medida de Respuesta | Inicio en menos de 5 segundos, 0% de interrupción al cajero, 100% de operaciones sincronizadas correctamente, aviso de éxito en menos de 2 segundos.                           |

### Notas de Arquitectura

|Aspecto|Descripción|
|---|---|
|Justificación de Negocio|Las ventas offline son ingresos reales; si no se sincronizan bien, el dueño pierde visibilidad y los reportes de caja no cuadran con la realidad.|
|Tácticas Arquitectónicas|Las operaciones pendientes se envían automáticamente al detectar conexión, en orden cronológico, marcándose como completadas para evitar duplicados. Si una operación falla, las demás siguen procesándose y esa se reintenta. El proceso es idempotente: reenviar la misma operación no la duplica.|
|Supuestos y Riesgos|Se asume que el servidor procesa en el orden recibido. Riesgo: si la conexión se corta a mitad de la sincronización, lo pendiente permanece en cola sin pérdida; los conflictos se manejan sin bloquear el resto.|
|Prueba de Aceptación|Tras 2 horas offline con 15 ventas, al reconectar la sincronización debe iniciar sola, completarse en menos de 30 segundos, y el cajero debe poder seguir vendiendo durante todo el proceso.|

---

## 9. ESC-CAL-DIS-0008

Cuando dos dispositivos que operaron offline vendan del mismo lote, el sistema debe aceptar ambas ventas, reasignar los lotes con FEFO y registrar una discrepancia visible para el supervisor.

| Campo               | Valor                                                |
| ------------------- | ---------------------------------------------------- |
| Atributo de Calidad | Disponibilidad                                       |
| Prioridad           | Alta                                                 |
| Dificultad / Riesgo | Muy Alta                                             |
| RF relacionado      | RF-011.3 (Gestionar discrepancias de sincronización) |


### Definición del Escenario

| Parte | Componente          | Descripción                                                                                                                                                                                                                                                                                                      |
| ----- | ------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1     | Origen              | Dos dispositivos del mismo negocio que operaron offline simultáneamente y vendieron del mismo lote.                                                                                                                                                                                                              |
| 2     | Estímulo            | El dispositivo A sincroniza una venta de 8 unidades del lote X (10 disponibles). Minutos después, B sincroniza una venta de 5 unidades del mismo lote, que en el servidor solo tiene 2 disponibles.                                                                                                              |
| 3     | Artefacto           | El servicio de sincronización del backend, que valida cada operación contra el estado real del inventario y reasigna lotes con FEFO.                                                                                                                                                                             |
| 4     | Entorno             | Fase de sincronización posterior a un período de operación offline en dos dispositivos del mismo negocio.                                                                                                                                                                                                        |
| 5     | Respuesta           | Ambas ventas se aceptan. La de B se reasigna con FEFO: toma las 2 unidades que quedan del lote X y, si existe otro lote, completa con él. La parte que ningún lote cubre se guarda como cantidad sin cobertura y como discrepancia ABIERTA, y se notifica al supervisor. El lote nunca queda con saldo negativo. |
| 6     | Medida de Respuesta | 0 ventas perdidas, 0 lotes con saldo negativo, 100% de faltantes registrados como discrepancia, aviso al supervisor en menos de 5 segundos, discrepancias resueltas en 1 día o menos                                                                                                                             |

### Notas de Arquitectura

| Aspecto                  | Descripción                                                                                                                                                                                                                                                                                                                                                                     |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Justificación de Negocio | Una venta es un hecho del mundo real: el cliente ya pagó y se llevó el producto. Rechazarla descuadra la caja y destruye la confianza; el problema es de inventario, no de venta, y se corrige con trazabilidad.                                                                                                                                                                |
| Tácticas Arquitectónicas | Identificador de operación generado en el cliente (idempotencia) y número de secuencia por dispositivo. Reasignación FEFO en el servidor (ADR-003). Estados por operación: ACEPTADA, REASIGNADA, DISCREPANCIA. Restricción CHECK (cantidad_disponible >= 0) en el lote. Cola de discrepancias visible al supervisor, resuelta con un ajuste CORRECCION (ADR-002 y su enmienda). |
| Supuestos y Riesgos      | Se asume que las discrepancias son poco frecuentes. Riesgo: con muchos dispositivos offline a la vez aumentan; se mitiga con la cola de discrepancias y la prueba de carga SPK-002b.                                                                                                                                                                                            |
| Prueba de Aceptación     | Con un lote X de 10 unidades y sin otro lote, si A sincroniza 8 y luego B sincroniza 5: ambas ventas quedan aceptadas, X queda en 0, la venta de B guarda 3 unidades sin cobertura, existe una discrepancia ABIERTA de 3 y el supervisor la cierra con un ajuste CORRECCION.                                                                                                    |

---

## 10. ESC-CAL-DIS-0029

Cuando el módulo de recomendaciones presente un error el sistema debe mantener el módulo de ventas operativo sin afectar el registro de transacciones del cajero

| Campo               | Valor                      |
| ------------------- | -------------------------- |
| Atributo de Calidad | Disponibilidad             |
| Prioridad           | Alta                       |
| Dificultad / Riesgo | Media                      |
| RF relacionado      | RF-004.1 (Registrar venta) |


### Definición del Escenario

|Parte|Componente|Descripción|
|---|---|---|
|1|Origen|Fallo interno de un módulo secundario del sistema durante la operación del negocio.|
|2|Estímulo|El motor de recomendaciones de compra presenta un error no recuperable que impide generar la lista de reposición priorizada.|
|3|Artefacto|El módulo de ventas y el módulo de inventario como funcionalidades prioritarias que deben permanecer operativas ante fallos de otros módulos.|
|4|Entorno|Operación normal del punto de venta con clientes esperando para pagar.|
|5|Respuesta|El cajero sigue vendiendo y consultando inventario sin interrupción; solo la sección de recomendaciones del panel muestra un aviso de módulo no disponible.|
|6|Medida de Respuesta|0 segundos de interrupción en ventas, 100% de funciones de venta e inventario operativas, aviso visible en menos de 1 segundo.|

### Notas de Arquitectura

|Aspecto|Descripción|
|---|---|
|Justificación de Negocio|La capacidad de vender es lo más crítico del negocio; un fallo en recomendaciones nunca puede impedir cobrar.|
|Tácticas Arquitectónicas|Arquitectura modular con aislamiento de fallos: cada módulo opera de forma independiente y se degrada sin propagar el error. Cada sección del panel se renderiza por separado con su propio manejo de errores.|
|Supuestos y Riesgos|Se asume que ventas no depende directamente de recomendaciones. Riesgo: un fallo en la base de datos compartida afectaría a todos los módulos simultáneamente; el modo offline entra como mitigación.|
|Prueba de Aceptación|Ante una excepción no recuperable del motor de recomendaciones, el cajero debe seguir vendiendo, el supervisor consultando inventario, y el panel mostrar el aviso solo en esa sección.|

---

## 11. ESC-CAL-DIS-0033

Cuando el sistema se ejecute en un equipo con 4GB de RAM y procesador básico debe cargar todos los módulos en menos de 5 segundos sin bloqueos ni errores de memoria

| Campo               | Valor                        |
| ------------------- | ---------------------------- |
| Atributo de Calidad | Disponibilidad               |
| Prioridad           | Alta                         |
| Dificultad / Riesgo | Media                        |
| RF relacionado      | RF-011.1 (Operación offline) |


### Definición del Escenario

|Parte|Componente|Descripción|
|---|---|---|
|1|Origen|Un equipo de gama baja típico del mercado objetivo (recursos de memoria y procesamiento modestos).|
|2|Estímulo|El dueño opera el sistema en un equipo con estas especificaciones mínimas durante una jornada completa de 8+ horas.|
|3|Artefacto|La aplicación completa, incluyendo el mecanismo de sincronización, el almacenamiento local y todas las operaciones de renderizado y cálculo del cliente.|
|4|Entorno|El equipo ejecuta simultáneamente el sistema, el navegador con otras pestañas y una hoja de cálculo.|
|5|Respuesta|Todos los módulos cargan rápido, las operaciones de venta responden con tiempos similares a un equipo más potente, y no hay bloqueos ni errores de memoria durante la jornada.|
|6|Medida de Respuesta|Carga inicial menor a 5 segundos, consumo de memoria controlado tras 8 horas, 0 errores de memoria, degradación no mayor al 20% frente a un equipo con más recursos.|

### Notas de Arquitectura

|Aspecto|Descripción|
|---|---|
|Justificación de Negocio|El mercado objetivo usa equipos de gama baja adquiridos hace años; exigir hardware moderno excluiría a la mayoría del mercado potencial.|
|Tácticas Arquitectónicas|Carga inicial liviana con lo esencial primero y el resto bajo demanda. Limpieza periódica de datos locales antiguos, listas largas renderizadas solo en lo visible, y manejo cuidadoso de recursos para evitar fugas de memoria.|
|Supuestos y Riesgos|Se asume un mínimo de recursos y un disco funcional. Riesgo: discos más lentos podrían ralentizar el almacenamiento local; se mitiga probando en equipos representativos del mercado.|
|Prueba de Aceptación|En un equipo modesto con varias pestañas abiertas, tras 8 horas continuas de uso no debe haber bloqueos, errores de memoria ni degradación progresiva.|

---