# Verification-2: drafts/arquitectura.md (T4, ciclo 2)
Resultado global: **FAIL** (1 fallo accionable, de coherencia interna; los 4 fallos del ciclo 1 están cerrados, las cifras no cambiaron).

Archivos leídos en el estado actual: `drafts/arquitectura.md` (273 líneas), `research/stack.md` (§1, §2, §10), `verification-1.md`.

## A. Fallos del ciclo 1
| # | Punto | Evidencia (línea del draft / stack.md) | Resultado |
|---|---|---|---|
| A1 | Pieza 9 con alternativa descartada, respaldada por stack.md §10 | Draft l.29: Flask (async sobre WSGI con un worker por petición; doc de inicio sin validación ni OpenAPI, marcado [INFERENCIA]), Litestar (válida sin ventaja clara), Django REST n/d. stack.md l.322 dice "uses one worker to handle one request/response cycle" y que la doc de inicio no documenta validación ni OpenAPI ("ausencia en la doc consultada, no prueba"). l.323 y l.325: Litestar "alternativa válida... no aporta ventaja clara". l.317 y l.330: Django REST n/d. l.321: FastAPI cubre async + Pydantic + OpenAPI. Nada afirmado que §10 no diga. | OK |
| A2a | Arista a `n/d` por "ningún trozo relevante" | Mermaid l.64: `X1 -- ningun trozo relevante --> ND2`. | OK |
| A2b | "ambos" con rama vacía | Sec. 3.4, l.109: respuesta parcial con la rama que tiene resultado; si las dos vacías, `n/d`. | OK |
| A2c | Nivel de `n/d` | Sec. 3.4, l.108: por afirmación; global solo si ninguna conserva fuente. **Pero** ver fallo F1: sec. 5.1 l.159 lo contradice. | **FALLO (coherencia)** |
| A2d | Errores SQL/API/JSON, sin telco/periodo | l.110 (SQL: 1 reintento, luego `n/d`), l.111 (API: 1 reintento, `n/d`, sin cambio de proveedor), l.112 (JSON inválido), l.113 (sin telco/periodo: `n/d` en DATOS, sin filtro en TEXTO). | OK |
| A2e | `version`/`vigente` y tabla `documentos` | l.87: `kpi_values(..., version, vigente)`, la consulta filtra `vigente = true`. l.88: tabla `documentos (documento PK, telco, tipo_doc, periodo_fiscal, url, hash_sha256, version, vigente)`. Coincide con lente 7 (l.172). | OK |
| A3 | Camelot / Tabula separados | Draft l.21: Camelot = Ghostscript + ~50-70 % en stream; Tabula = Java, rápida solo en plantillas fijas, "§1 no da dato de acierto bajo". stack.md l.35 (Camelot: ~50-70 %, Ghostscript) y l.37 (Tabula: MIT, rápida en plantillas fijas, requiere Java). | OK |
| A4 | "0 filas -> n/d" con `[INFERENCIA]` | Draft l.93: "(stack.md §2, última viñeta, marcada [INFERENCIA] allí)". stack.md l.60 es [INFERENCIA]. | OK |
| Plus | Frase del hash, sec. 7 | l.218: "un PDF re-descargado solo cuenta como el mismo documento si su hash SHA-256 coincide con el de `fuentes.csv`". Se entiende. | OK |

## B. Regresión
- **Otras 9 piezas:** releídas contra stack.md las filas citadas en el ciclo 1 (§1 Docling/PyMuPDF/pdfplumber, §4.1-4.4, §5.1-5.3, §6.2-6.3, §7). Texto y citas coinciden con lo verificado en el ciclo 1 (p. ej. Ragas 0,777, promptfoo 0,800, BGE-M3 1.024 dim, pgvector 0.8.7). OK.
- **Mermaid (revisión MANUAL, no hay mmdc):** `ND2` se usa en l.64 antes de su definición `ND2[n/d]` en l.65; es válido en Mermaid. `ND2` tiene salida (`ND2 --> LOG`, l.70), así que no queda huérfano. Todos los identificadores de aristas están definidos; subgraph cerrado; formas de nodo cerradas. OK (manual).
- **Coste sec. 8:** recalculado con Python: Haiku 0,0375, Sonnet 0,75, Opus 1,50; totales 13,406 y 25,406. El draft dice 0,04 / 0,75 / 1,50 y suma 13,4 y 25,4, con margen 17-33 $. Números sin cambios. OK.
- **Contrato de cita 5.1 vs `version`/`vigente`/`documentos`:** la cita sigue saliendo de `documento` y `pagina_pdf`, presentes en `kpi_values` y en los metadatos de trozos. `vigente` no altera el contrato. OK. **Pero** 5.1 l.159 contradice el nivel de `n/d` de 3.4 (F1).
- **Observación (no fallo):** el diagrama manda `D3 -- 0 filas --> ND1` y `X1 -- ningun trozo --> ND2` sin distinguir la ruta "ambos"; el texto de 3.4 (l.109) define el resultado parcial. Es una simplificación del diagrama; conviene una nota al pie. También hay `periodo` (kpi_values) frente a `periodo_fiscal` (documentos y metadatos de trozos): nombre distinto para el mismo campo.

## Fallos accionables
**F1 (coherencia, incumple la definición del caso límite c).** Sec. 5.1, l.159: "Sin fuente -> `n/d`: afirmación sin `fuente_id` válido, 0 filas SQL, contexto sin trozos relevantes o ruta `fuera_de_alcance` -> la respuesta es `n/d`". Sec. 3.4, l.108: el `n/d` es por afirmación; la respuesta global solo es `n/d` si ninguna afirmación conserva fuente. Y l.109 admite respuesta parcial en "ambos". Hay que alinear l.159 con 3.4 (por ejemplo: "afirmación sin `fuente_id` válido -> se descarta o se marca `n/d` (ver 3.4); respuesta global `n/d` solo si ninguna conserva fuente").
Opcional (no bloquea): unificar `periodo`/`periodo_fiscal` y añadir nota sobre "ambos" en el diagrama.

NO VERIFICABLE (igual que en el ciclo 1): "Docker 24.0.2" y la convención de página "aprobada por Javier" dependen de lo que confirme el orquestador.
