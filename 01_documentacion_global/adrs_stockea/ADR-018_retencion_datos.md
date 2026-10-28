# ADR-018 — Retención y datos personales

| Campo                   | Valor                                |
| ----------------------- | ------------------------------------ |
| Estado                  | **Aprobado**                         |
| Escenarios relacionados | Ley 1581 de 2012, Código de Comercio |
| Atributos de calidad    | Cumplimiento legal, Seguridad        |

### Contexto

Había inconsistencia entre "5 años" y "10 años", y la Ley 1581 exige supresión de datos personales mientras el Código de Comercio exige conservación de registros mercantiles.

### Decisión

- **Plazo único: 10 años** (valor conservador alineado con el art. 60 del Código de Comercio; a validar con abogado). **El documento de restricciones ya fue actualizado de "5 años" a "10 años".**
- **Separar datos personales de registros mercantiles:** personales → anonimizar ante supresión; mercantiles → inmutables por el plazo legal.
- Política de tratamiento, consentimiento en el registro, cláusulas para transferencia internacional y encargados (Cloudflare, correo, push).
- Verificar si el RNBD aplica al tamaño del negocio.

### Consecuencias

- Diseño de datos más cuidadoso desde el principio. Legal ↑. Costo de almacenamiento ↑ marginal.

### Referencias

- Ley 1581 de 2012; Decreto 1377 de 2013; Art. 60 Código de Comercio de Colombia.

> **Nota:** esto no constituye asesoría legal. El plazo definitivo y el alcance del RNBD deben confirmarse con un abogado.
