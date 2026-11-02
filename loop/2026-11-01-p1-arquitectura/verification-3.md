# Verification 3 — arquitectura.md (ciclo 3)
Resultado: **PASS**

## Comprobaciones
| # | Qué | Método | Esperado | Obtenido | ✓/✗ |
|---|---|---|---|---|---|
| 1 | Contradicción `n/d` (3.4 vs 5.1) cerrada | Lectura de l.108, l.109 y l.159; Grep de otras frases sobre n/d global | Ambas: n/d por afirmación; global solo si ninguna queda respaldada | Coinciden; sin tercera frase contradictoria | ✓ |
| 2 | `periodo` vs `periodo_fiscal` | Grep "periodo" en todo el archivo | Campos/columnas unificados; resto justificado | Todo nombre de campo/columna es `periodo_fiscal`; quedan `periodo` en l.57 (etiqueta Mermaid), l.77 y l.113 (prosa). Cosmético | ✓ (menor) |
| 3a | 10 piezas intactas con citas a stack.md | Grep de filas `\| N \|` (N=1..10) y revisión de la tabla l.19-30; 56 menciones a stack.md | 10 filas con cita | 10 filas, citas `stack.md §N` presentes | ✓ |
| 3b | Sec. 8 coste (números) | Recalculo con Python | Coinciden con lo escrito | Haiku 0,0375; Sonnet 0,75; Opus 1,5; Fable 3,75; Gemini 3.8 0,28125; lite 0,10. P3 7,5/15; P4 4,59/8,34; P5 0,5625; P6 0,75/1,5; totales 13,41/25,41 (escrito ≈13 / ≈25, suma 13,4 / 25,4) | ✓ |
| 3c | Precios coinciden con stack.md §9 | Grep stack.md l.222-223 | Haiku 0,10/0,50 (≤100k), 0,50/2,50 (>100k) | Coincide | ✓ |
| 3d | Mermaid parsea | Revisión manual l.38-71 | Bloque cerrado, nodos/etiquetas sin rotura | Un solo bloque ```mermaid cerrado; subgraph/end correcto; etiquetas con `<br/>` y `->` dentro de `[...]` intactas (l.43 `Tablas -> filas de KPI` sin comillas: `->` no rompe en corchetes, igual que en ciclos previos); l.57 etiqueta íntegra | ✓ |

## Evidencia
**1. n/d**
- l.108 (3.4): "Nivel de `n/d`: es **por afirmación**. Una afirmación sin `fuente_id` válido se descarta o se marca `n/d`; la respuesta global es `n/d` solo si ninguna afirmación conserva fuente."
- l.159 (5.1): "una afirmación sin `fuente_id` válido se marca `n/d` y se omite (nivel por afirmación, sec. 3.4); la respuesta global es `n/d` solo si ninguna afirmación queda respaldada, hay 0 filas SQL, el contexto no tiene trozos relevantes o la ruta es `fuera_de_alcance`".
- Misma regla. Las condiciones extra de 5.1 (0 filas, sin trozos, fuera_de_alcance) coinciden con l.50, 56, 64 del Mermaid y l.80, 93, 101; no contradicen. l.109 (ambas ramas vacías → n/d global) es consistente.
- Nota: l.277 dice que 5.1 se alineó con 3.4; confirmado en el texto, no solo por la nota.

**2. periodo**
- `periodo_fiscal`: l.22, 25, 43, 87, 88, 98, 167, 172, 194 (campo/columna/filtro).
- `periodo` suelto: l.57 es la etiqueta "filtro telco / periodo" y nombra el mismo filtro que l.25 ("filtro por `telco`/`periodo_fiscal`") y l.98. Es el mismo concepto, por tanto una inconsistencia de nomenclatura, pero en un texto de etiqueta de diagrama, sin backticks. l.77 ("de una telco y un periodo") y l.113 ("sin periodo identificable") son lenguaje natural sobre la pregunta del usuario, no nombres de campo: uso legítimo. No hay ambigüedad real: el contexto deja claro que es `periodo_fiscal`.
- Clasificación: cosmético; solo l.57 es estrictamente inconsistente.

**3. Regresión**
- El borrador no está versionado en git (sin historial del archivo), así que no hay diff; la regresión se comprobó contra los criterios de ciclos previos (contenido y cálculos), no contra un baseline binario.
- Cálculo ejecutado: `0.25*0.1+0.025*0.5=0.0375`, etc.; `5*s+3*g=4.59375`, `5*o+3*g=8.34375`, `15*h=0.5625`; totales `13.40625` y `25.40625`.

## Fallos
- **Bloqueantes:** ninguno.
- **Menores / cosméticos:**
  1. l.57: "filtro telco / periodo" debería ser `periodo_fiscal` para ser coherente con el resto (l.43, 87, 98).
  2. l.277 (Cambios ciclo 3) deja anotado "no tocar" l.57, decisión consciente del builder; sigue siendo inconsistencia menor.
  3. Limitación de método: la regresión no tiene baseline en git.
