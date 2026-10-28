# ADR-006 — Motor de recomendaciones de compra basado en reglas

| Campo                   | Valor            |
| ----------------------- | ---------------- |
| Estado                  | **Aprobado**     |
| Escenarios relacionados | ESC-CAL-CON-0029 |
| Atributos de calidad    | Confiabilidad    |

### Contexto

STOCKEA genera recomendaciones de compra cruzando velocidad de venta, stock actual y vencimientos. Se decide entre reglas deterministas y ML.

### Decisión

**Motor basado en reglas deterministas.**

### Justificación

Para un MVP, la transparencia y la testeabilidad superan la precisión teórica del ML. El dueño necesita entender **por qué** se le recomienda una cantidad ("vendes 10/día, quieres 7 días de stock, hoy tienes 20"), algo que una caja negra no ofrece. El ML exige meses de datos y experiencia que el equipo no tiene.

### Tácticas arquitectónicas

- Fórmula: `cantidadSugerida = (velocidadPromedio × díasObjetivo) − stockActual`; si ≤ 0, no se recomienda.
- Protección contra división por cero (producto sin ventas → "sin datos suficientes").
- Regla anti-contradicción (RN-REC-008): si la cobertura supera los días restantes del lote más próximo, el producto va al motor de descuentos, no al de recomendaciones.
- Niveles de urgencia: `CRITICO` (<3 días), `ALTO` (<7 días), `MEDIO` (resto).
- Snapshot inmutable del stock al generar la recomendación.
- **El motor corre en el Worker/Jobs**, no en el Backend; los resultados se persisten en PostgreSQL.

### Consecuencias

- Recomendaciones explicables y auditables.
- No se detectan patrones estacionales complejos en el MVP (mejorable en iteración 2 con promedio ponderado).
- El período de cálculo (por defecto 7 días) es configurable por el administrador.
