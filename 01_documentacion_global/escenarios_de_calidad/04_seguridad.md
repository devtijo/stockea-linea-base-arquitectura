## 16. ESC-CAL-SEG-0004

Cuando un usuario no realice actividad durante 15 minutos el sistema debe cerrar la sesión automáticamente para proteger el acceso al negocio

| Campo               | Valor                    |
| ------------------- | ------------------------ |
| Atributo de Calidad | Seguridad                |
| Prioridad           | Alta                     |
| Dificultad / Riesgo | Baja                     |
| RF relacionado      | RF-007.6 (Cerrar sesión) |

### Definición del Escenario

|Parte|Componente|Descripción|
|---|---|---|
|1|Origen|El reloj interno del sistema que monitorea la actividad de la sesión del usuario.|
|2|Estímulo|Han transcurrido 15 minutos sin que el usuario realice ninguna interacción con el sistema.|
|3|Artefacto|El módulo de gestión de sesiones del frontend que controla el tiempo de inactividad y la invalidación de la sesión.|
|4|Entorno|El sistema está activo con una sesión abierta y el usuario se ausentó del equipo sin cerrar sesión.|
|5|Respuesta|La sesión se cierra automáticamente y redirige al login, el turno de caja abierto permanece intacto para que el cajero lo cierre al volver, y el carrito sin confirmar se descarta.|
|6|Medida de Respuesta|Cierre a los 15 minutos (±10 segundos), redirección en menos de 1 segundo, el turno de caja no se cierra automáticamente.|

### Notas de Arquitectura

|Aspecto|Descripción|
|---|---|
|Justificación de Negocio|En una tienda el equipo puede quedar accesible para otras personas; el cierre automático protege los datos sin depender de que el usuario recuerde cerrar sesión.|
|Tácticas Arquitectónicas|Un contador de inactividad se reinicia con cualquier interacción; al expirar, la sesión se invalida localmente y se redirige al login. El turno de caja nunca se cierra solo porque el monto final debe declararlo la persona real. Sin conexión, el cierre de sesión ocurre localmente sin depender del servidor.|
|Supuestos y Riesgos|Se asume 15 minutos como umbral adecuado. Riesgo: perder un carrito con productos ya agregados; se puede mitigar con un aviso previo de 60 segundos antes del cierre.|
|Prueba de Aceptación|Con sesión y turno abiertos, tras 15 minutos sin actividad el sistema debe cerrar sesión, redirigir al login, mantener el turno abierto y descartar la venta sin confirmar.|

---

## 17. ESC-CAL-SEG-0006

Cuando un cajero intente eliminar o ajustar el inventario manualmente el sistema debe impedir la operación y mostrar un mensaje indicando que no tiene permisos para esa acción

| Campo               | Valor                         |
| ------------------- | ----------------------------- |
| Atributo de Calidad | Seguridad                     |
| Prioridad           | Alta                          |
| Dificultad / Riesgo | Baja                          |
| RF relacionado      | RF-002.3 (Ajustar inventario) |
### Definición del Escenario

|Parte|Componente|Descripción|
|---|---|---|
|1|Origen|Usuario con sesión activa y rol de Cajero, autenticado en el sistema durante su turno de caja.|
|2|Estímulo|El cajero intenta eliminar un producto o modificar manualmente la cantidad disponible de un lote desde la interfaz de inventario, sin contar con el permiso correspondiente a su rol.|
|3|Artefacto|El módulo de control de acceso y autorización del backend, que valida el rol del usuario antes de ejecutar cualquier operación sobre el inventario.|
|4|Entorno|El sistema opera normalmente, en línea o sin conexión; el bloqueo debe comportarse igual en ambos casos.|
|5|Respuesta|El sistema rechaza la operación antes de afectar el inventario, avisa al cajero que no tiene permisos, y registra el intento con usuario, fecha, hora y acción intentada.|
|6|Medida de Respuesta|0% de impacto en el inventario, aviso en menos de 1 segundo, 100% de intentos registrados en el log de auditoría.|

### Notas de Arquitectura

|Aspecto|Descripción|
|---|---|
|Justificación de Negocio|Solo supervisores y administradores deben poder modificar el inventario manualmente, ya que afecta stock, rotación y recomendaciones; permitirlo a cualquiera expondría al negocio a pérdidas no auditables.|
|Tácticas Arquitectónicas|Control de acceso por roles acumulativos, validado siempre en el backend y no solo en la interfaz, para que no pueda evadirse manipulando la aplicación. Cada intento no autorizado queda registrado de forma permanente.|
|Supuestos y Riesgos|Se asume que el cajero tiene solo acceso de lectura al inventario. Riesgo: si la validación solo estuviera en el frontend, podría evadirse; también debe validarse localmente en modo offline sin depender del servidor.|
|Prueba de Aceptación|Un cajero que intente eliminar un producto o ajustar un lote debe recibir el rechazo, el aviso en menos de 1 segundo y quedar registrado, tanto online como offline.|

---

## 18. ESC-CAL-SEG-0010

Cuando los datos se sincronicen con la nube la transmisión debe realizarse a través de un canal cifrado para proteger la información sensible del negocio en tránsito

| Campo               | Valor                              |
| ------------------- | ---------------------------------- |
| Atributo de Calidad | Seguridad                          |
| Prioridad           | Crítica                            |
| Dificultad / Riesgo | Baja                               |
| RF relacionado      | RF-011.2 (Sincronizar operaciones) |

### Definición del Escenario

| Parte | Componente          | Descripción                                                                                                                                                                                                                                                                   |
| ----- | ------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1     | Origen              | El módulo de sincronización del sistema que envía datos desde el equipo local hacia el servidor en la nube.                                                                                                                                                                   |
| 2     | Estímulo            | El sistema inicia la sincronización de ventas, ajustes de inventario y datos del negocio con el servidor al recuperar la conexión a internet.                                                                                                                                 |
| 3     | Artefacto           | El canal de comunicación entre la aplicación y el servidor por donde viajan los datos del negocio, incluyendo ventas, precios, inventario y datos de usuarios.                                                                                                                |
| 4     | Entorno             | Transición de modo offline a modo online en una red potencialmente no confiable (wifi público, red compartida del local).                                                                                                                                                     |
| 5     | Respuesta           | Todo el tráfico que cruza la red viaja cifrado con TLS (cliente → Cloudflare → Nginx en modo Full (strict) con Origin CA) y el sistema rechaza cualquier conexión sin cifrar. El tramo Nginx → Backend es HTTP interno dentro de la misma máquina y no atraviesa ninguna red. |
| 6     | Medida de Respuesta | 100% de comunicaciones por red cifradas, 0 transmisiones en texto plano fuera del host, certificados válidos y no autofirmados en producción, origen accesible solo desde Cloudflare.                                                                                         |

### Notas de Arquitectura

| Aspecto                  | Descripción                                                                                                                                                                                                                                                                                                                        |
| ------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Justificación de Negocio | La sincronización transporta información financiera y datos personales; interceptarla en una red insegura violaría la protección de datos personales y destruiría la confianza en el sistema.                                                                                                                                      |
| Tácticas Arquitectónicas | Cloudflare en modo Full (strict) con Origin CA, redirección obligatoria de HTTP a HTTPS y HSTS. Firewall del VPS que acepta 80/443 solo desde rangos de Cloudflare (o Authenticated Origin Pulls), sin exponer la IP del origen. Renovación automática de certificados. En desarrollo se usan certificados de prueba equivalentes. |
| Supuestos y Riesgos      | Se asume que Cloudflare, como encargado del tratamiento (ADR-018), descifra el tráfico en el borde. Riesgo: si la IP del origen se filtra, se evade el WAF; se mitiga con el firewall y los Authenticated Origin Pulls. Riesgo: certificado vencido; se mitiga con renovación automática y monitoreo                               |
| Prueba de Aceptación     | Al sincronizar sobre una red wifi compartida, el 100% del tráfico viaja cifrado, un intento por HTTP se redirige a HTTPS y una petición directa a la IP del origen es rechazada.                                                                                                                                                   |

---
