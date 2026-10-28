# Requisitos no funcionales clave

Cada uno se verifica con el escenario de calidad indicado; el detalle de los seis componentes está en [`escenarios_calidad/`](../escenarios_calidad/).

| Atributo | Requisito | Medida | Escenario |
|---|---|---|---|
| **Rendimiento** | Registrar una venta de hasta 10 productos debe completarse sin que el cliente note espera | ≤ 3 segundos desde confirmar hasta ver la confirmación, online u offline | ESC-CAL-REN-0021 |
| **Disponibilidad** | El sistema debe operar sin conexión y sincronizar solo al reconectar | Operación offline completa (vender, consultar, cerrar caja); sincronización inicia en < 5 s; 0 ventas perdidas | ESC-CAL-DIS-0001, ESC-CAL-DIS-0008 |
| **Seguridad** | Aislamiento entre negocios y protección de la sesión en dispositivos compartidos | 100% del tráfico cifrado; `negocio_id` + Row-Level Security en todas las consultas; cierre de sesión por 15 min de inactividad | ESC-CAL-SEG-0004, ESC-CAL-SEG-0006 |
| **Usabilidad** | El cajero debe poder operar sin entrenamiento formal y enterarse de las alertas sin buscarlas | Alertas de stock y vencimiento entregadas por Web Push; estado de sincronización siempre visible | ESC-CAL-USA-0017 |
