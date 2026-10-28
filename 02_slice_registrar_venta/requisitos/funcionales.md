# Requisitos funcionales — slice "Registrar venta"

Solo los requisitos que toca este slice.

| ID | Nombre | Descripción |
|---|---|---|
| **RF-004.1** | Registrar venta | El sistema deberá permitir al cajero registrar una venta con uno o varios productos, aplicando el precio configurado y el descuento activo si existe, con cálculo automático del total y el cambio. |
| **RF-011.1** | Registrar operaciones en modo offline | El sistema deberá permitir al cajero, supervisor y administrador registrar ventas, ajustes de inventario y entradas de mercancía sin conexión, almacenando cada operación localmente con identificador único y marca de tiempo hasta que se restablezca la conexión. |
| **RF-011.3** | Gestionar conflictos de sincronización | El sistema deberá aceptar siempre toda venta sincronizada, por ser un hecho real. Cuando el stock de un lote ya no alcance, el servidor reasignará la venta a los siguientes lotes por vencimiento más próximo (FEFO); si ningún lote alcanza, registrará una discrepancia de inventario visible al supervisor, quien la resuelve con un ajuste de tipo `CORRECCION` (ADR-002). |
| **RF-005.1** | Registrar apertura de caja | El sistema deberá permitir al cajero registrar la apertura de caja con usuario, fecha, hora exacta y monto inicial declarado. Este registro es inmodificable una vez guardado. |
| **RF-002.1** | Registrar entrada de mercancía | El sistema deberá permitir al supervisor registrar una entrada de mercancía especificando producto, proveedor, número de factura, cantidad, precio de costo y fecha de vencimiento, generando automáticamente un lote asociado. |

**Dependencias del slice:** RF-004.1 requiere un turno abierto (RF-005.1) y lotes existentes (RF-002.1). RF-011.1 y RF-011.3 definen su comportamiento sin conexión.

[Catálogo completo de requisitos funcionales](../../01_documentacion_global/requisitos/requisitos_funcionales.md)
