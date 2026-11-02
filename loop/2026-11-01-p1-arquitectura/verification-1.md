# Verification-1: drafts/arquitectura.md (T4)
Resultado global: **FAIL** (4 fallos accionables, ninguno de cifras; los precios y sumas cuadran).

Archivos leídos en el estado actual: `drafts/arquitectura.md` (253 líneas), `research/stack.md` (311 líneas), `plan.md` (T4), `brief.md` (DoD 1).

## Resumen de comprobaciones
| # | Qué | Método | Resultado |
|---|---|---|---|
| 1 | Trazabilidad 10 piezas -> stack.md | Contraste cita a cita | 9 OK, 1 con fallo menor (imprecisión pieza 1), más 1 fallo de cita sin etiqueta (3.2 punto 6) |
| 2 | Aritmética de coste sec. 8 | Recalculado con Python | OK (13,41 / 25,41 $) |
| 3 | Completitud DoD T4 | Lista contra plan.md y brief.md | FALLO: pieza 9 sin alternativa descartada |
| 4 | Casos límite | Lectura de sec. 3, 5.1, diagrama | FALLO: 5 casos sin tratar |
| 5 | Mermaid | Revisión manual (no hay mmdc instalado) | OK (manual) |
| 6 | Puntos del builder | Líneas 159 y 205 | (a) OK; (b) solo redacción |

## 1. Trazabilidad decisión -> fuente
Cada pieza contrastada con la sección o fila citada de stack.md:
- **1 Ingesta**: OK en lo esencial. stack.md §1 fila Docling: "MIT... TableFormer... nº de página... Exportar a Markdown pierde páginas" (línea 32). PyMuPDF "AGPL o licencia comercial" (línea 34). pdfplumber "tablas sin bordes solo parcial" (línea 33).
  - **FALLO menor (línea 21 del draft):** "Camelot y Tabula: Ghostscript y Java, y acierto bajo en tablas sin bordes (§1)". stack.md línea 35 dice lo del acierto bajo (50-70 %) solo de Camelot (modo stream). Tabula (línea 37) es "MIT, rápida en plantillas fijas" y "Requiere Java; sin PDF escaneado". No hay dato de acierto bajo en tablas sin bordes para Tabula, y no necesita Ghostscript.
- **2 Troceado**: OK. §4.1 (líneas 76-78): `chunk.meta.doc_items[].prov[].page_no`, metadatos [INFERENCIA]. El descarte por tamaño fijo está etiquetado [INFERENCIA] sin medir, igual que en la fuente.
- **3 Embeddings**: OK. Tabla §4.2 (líneas 84-87): BGE-M3 1.024 dim, 8.192 tok, MIT. e5 512 tok. Jina CC BY-NC. Límite 2.000 dim en §4.3 fila Escala (línea 101). Fijar la revisión del modelo está en la línea 90.
- **4 BBDD vectorial**: OK. pgvector 0.8.7 (línea 96). Filas "Filtro por metadatos", "Persistencia" (Chroma "No es seguro entre procesos con escritores", línea 98) y "Valor en producción" (línea 102). DuckDB un solo escritor, §2 (línea 48).
- **5 Recuperación**: OK. Recomendación §4.4 (línea 117) y fila "Híbrida" §4.3 (línea 100). pg_textsearch: licencia y Docker [HUECO] (línea 113). Reranker: tamaño y velocidad [HUECO] (línea 115). OIBDA/léxico [INFERENCIA] (línea 117).
- **6 LLM**: OK. §5.1 wrapper propio y LiteLLM sin licencia ni Gemini (líneas 126-129). §5.2 `gemini-3.8-flash` y `3.1-flash-lite` (línea 142). §5.3 modelo más barato que cumple, coste por tarea completada (línea 147). Citations incompatible con structured outputs (línea 129). `gpt-5.4-mini` como reserva (línea 143). IDs de la tabla §9 (líneas 222-226) coinciden con los de sec. 5 del draft.
- **7 Agente**: OK. §6.1 línea 158 y §3 línea 70. El [HUECO] de aristas condicionales está en las líneas 68 y 71.
- **8 Evals**: OK. Ragas: sin commits desde feb-2026, 0,777 a una respuesta incorrecta (línea 165). promptfoo: 0,800 y npm (línea 166). LangSmith: autoalojado solo en Enterprise (línea 179). Langfuse MIT (línea 178).
- **9 API**: la cita es correcta (§7 primera viñeta, línea 187). El problema es de completitud; ver punto 3.
- **10 Despliegue**: OK. "Composición mínima" [INFERENCIA] (línea 189). Licencia de Docker Desktop (línea 188). La alternativa DuckDB está en la línea 190.
- **Resto de citas muestreadas**: lente 2 (3,1 s/página, ~52 min; líneas 32 y 40), lente 3 (1M tokens, línea 228), lente 7 (caché a/b/c, línea 191), tokenizadores +30 % (línea 192), Haiku tarifa alta (línea 228), Gemini duplica el 1-ene-2027 (líneas 139 y 233). Todas coinciden.
- **FALLO menor (draft línea 91, sec. 3.2 punto 6):** "0 filas -> n/d (stack.md §2, última viñeta)". Esa viñeta (stack.md línea 60) está marcada [INFERENCIA] ("regla propia del proyecto"). El draft la cita sin la etiqueta, como si la fuente la respaldara como hecho.
- **Observación (no es fallo):** draft línea 132 etiqueta "[DECIDIDO]" el no usar Citations. stack.md lo marca [INFERENCIA] (línea 129); solo la incompatibilidad (error 400) es [HECHO]. Si Javier no lo cerró, la etiqueta debería ser otra.
- **NO VERIFICABLE:** "Docker 24.0.2 instalado (dato del usuario, 2-nov)" (draft líneas 34 y 244) y "convención de página aprobada por Javier (2-nov)" (líneas 133 y 252). No constan en stack.md ni en brief.md. stack.md y brief.md siguen diciendo "n/d". Depende de que el orquestador lo confirme.

## 2. Coste (sec. 8): recalculado con Python
Supuesto: 0,25 MTok de entrada y 0,025 MTok de salida por pasada. Precios de stack.md §9.

| Modelo | Calculado | Draft |
|---|---|---|
| Haiku 5.5 | 0,0375 | 0,04 (0,0375) OK |
| Sonnet 5.5 | 0,75 | 0,75 OK |
| Opus 5.5 | 1,50 | 1,50 OK |
| Fable 5.1 | 3,75 | 3,75 OK |
| Gemini 3.8-flash | 0,28125 | 0,28 OK |
| Gemini 3.1-flash-lite | 0,10 | 0,10 OK |

Fases:

| Fase | Sonnet | Opus |
|---|---|---|
| P3 | 7,5 | 15,0 |
| P4 | 4,594 | 8,344 |
| P5 | 0,5625 | 0,5625 |
| P6 | 0,75 | 1,5 |
| **Total** | **13,406** | **25,406** |

- El draft dice ≈13 / ≈25 con sumas 13,4 y 25,4: OK.
- El margen +30 % da 17,43 y 33,03, y el draft dice "17-33": OK.
- Los precios de entrada del draft coinciden con stack.md §9, incluidos los de caché (0,01 = 0,1×0,10; 0,10 = 0,05×2).
- El supuesto de tokens está declarado como "suposición mía". P4 se calcula con 5 + 3 pasadas y P5 con 15 pasadas Haiku, ambas suposiciones declaradas. **Resultado: OK.**

## 3. Completitud del DoD de T4 (plan.md líneas 42-50 y brief.md línea 11)
| Elemento | Estado |
|---|---|
| 10 piezas con herramienta + porqué + enlace a stack.md | Presentes (sec. 2). |
| **Alternativa descartada por pieza** | **FALLO en la pieza 9 (API, línea 29):** "Nada más evaluado. [HUECO] no hay segunda candidata". Está declarado con honestidad, pero el DoD exige alternativa descartada por cada una de las 10 piezas. |
| Dos rutas + enrutador | Sec. 3, 3.1-3.3 y diagrama. OK. |
| Capa común con interfaz mínima y cambio de proveedor | Sec. 5: `Protocol LLM` más `config.toml`. OK. |
| Contrato de cita | Sec. 5.1. OK. |
| Tabla de las 7 lentes con hueco declarado | Sec. 6, 7 filas. OK. |
| Estructura de carpetas, con glosario, golden set y data | Sec. 7. OK. |
| Diagrama Mermaid | Sec. 3. OK. |
| Coste por fase y nota del tope en P3 | Sec. 8 (línea 239). OK. |
| "Ninguna herramienta sin fuente" | OK, con las excepciones etiquetadas. Pieza 9: FastAPI viene de CONTEXTO, no de stack.md (declarado). |

## 4. Casos límite
Tratados:
- **Fuera de alcance:** ND0 (diagrama) y línea 79.
- **0 filas SQL:** línea 91, diagrama (`D3 -- 0 filas --> ND1`) y línea 146.
- **SQL rechazado por sqlglot:** ND1.
- **Sin fuente_id válido:** línea 145-146.
- **Pregunta mixta:** ruta `ambos` (línea 78).
- **Confianza baja:** se trata como `ambos` (línea 80).
- **Celdas ilegibles:** no se cargan (línea 87).

Sin tratar, o inconsistentes entre texto y diagrama:
- **a)** Texto 3.3 punto 4: "ningún trozo supera el umbral -> n/d". El diagrama no tiene esa arista: `X1 -- trozos --> G` no tiene salida a n/d.
- **b)** Ruta `ambos` con una rama vacía y la otra con datos (por ejemplo 0 filas SQL pero hay trozos de texto). No se define si la respuesta es parcial o `n/d`. El diagrama manda `D3 -- 0 filas --> ND1` incondicionalmente, aunque la otra rama tenga contenido.
- **c)** Granularidad del `n/d` (línea 146): "afirmación sin fuente_id válido -> la respuesta es n/d". No dice si es la respuesta entera o solo esa afirmación.
- **d)** Errores de ejecución distintos de "rechazada" o "0 filas": SQL que falla o agota el tiempo en Postgres, fallo o timeout de la API del modelo (enrutador o generación), JSON no válido del modelo, 429 de Gemini. No hay ruta ni `n/d` ni reintento definidos.
- **e)** Pregunta ambigua por falta de parámetros (sin telco o sin periodo), distinta de la ambigüedad dato/texto. El enrutador solo cubre la confianza de ruta. No se dice qué pasa si el filtro `telco/periodo` no se puede extraer, ni si hay varias filas del mismo KPI en distintos documentos (cifras reexpresadas).
- **f)** `kpi_values` (línea 86) no incluye `vigente`/versión, pero la lente 7 (línea 159) exige retirar versiones obsoletas con "columna `vigente` por documento" en una tabla `documentos` que no aparece en ninguna parte de la estructura ni de `db/init.sql`. El SQL generado podría mezclar versiones.

## 5. Sintaxis Mermaid (revisión MANUAL; no hay mermaid-cli instalado)
Revisados línea a línea los nodos y aristas del bloque (líneas 38-70).
- `flowchart TD` correcto.
- `subgraph OFFLINE["..."]` con título entre comillas y `end` presente.
- Formas de nodo bien cerradas: `[...]`, `{...}`, `[(...)]`.
- Todas las aristas son de la forma `-->` o `-- etiqueta -->`.
- Los `<br/>` son válidos.
- No aparecen paréntesis ni comillas dentro de etiquetas sin comillas, que son lo que suele romper el parser. Lo único dudoso es `->` dentro de la etiqueta de `T[Tablas -> filas de KPI...]`. Es válido, pero por precaución podría ir entre comillas.
- Todos los identificadores usados en aristas están definidos (A, B, C, T, E, PV, KV, Q, R, ND0-ND2, D1-D3, X1, G, V, ANS, LOG). **Resultado: OK (manual).**

## 6. Puntos marcados por el builder
- **(a) OK.**
  - Línea 159: el reindexado por hash y `vigente` aparece en la columna de respuesta de la lente 7. La columna "Hueco declarado" de la misma fila lo etiqueta: "el reindexado incremental es diseño mío [INFERENCIA], sin fuente en stack.md". Queda declarado, no presentado como respaldado.
  - Línea 205: `data/` con PDFs fuera de git lleva "[INFERENCIA] sin fuente en stack.md; es decisión de proyecto".
  - Aviso menor: en el árbol (líneas 180-183), `data/` aparece como "DECIDIDO" sin matiz. Es coherente con la etiqueta de la línea 205 solo si Javier lo decidió. Eso depende del orquestador.
- **(b) Solo redacción.** Línea 205: "Orden de los PDFs: un dato solo es reproducible si el hash coincide." "Orden de los PDFs" es un residuo sin sentido, y el resto de la frase no depende de él. La afirmación sustantiva (reproducibilidad condicionada al hash SHA-256 de `fuentes.csv`) es coherente con la línea 181. No cambia el sentido, pero debe reescribirse, por ejemplo como "Reproducibilidad: un dato solo es reproducible si el hash del PDF coincide con `fuentes.csv`".

## Fallos accionables para el builder
1. **Pieza 9 (draft línea 29):** añadir una alternativa descartada con razón y fuente, o declarar en el brief/plan que el DoD se relaja para esta pieza. Si se mantiene sin alternativa, el DoD "10 piezas con alternativa" no se cumple.
2. **Pieza 1 (draft línea 21):** separar Camelot y Tabula. El "acierto bajo en tablas sin bordes" y el Ghostscript son solo de Camelot (stack.md líneas 35 y 37). Tabula requiere Java y no tiene dato de acierto en la fuente.
3. **Sec. 3.2 punto 6 (draft línea 91):** añadir [INFERENCIA] a la regla "0 filas -> n/d". stack.md línea 60 la marca así.
4. **Casos límite (sec. 3 y 5.1, diagrama):**
   - Añadir a X1 la arista a n/d por falta de trozos relevantes, que el texto ya declara.
   - Definir qué pasa en `ambos` si falla una de las dos ramas.
   - Definir la granularidad del `n/d` (respuesta entera o afirmación).
   - Definir el tratamiento de errores de ejecución SQL, fallos o timeouts de API y JSON inválido.
   - Definir el caso de pregunta sin `telco` o `periodo` extraíble.
   - Añadir `vigente`/versión a `kpi_values` (o declarar la tabla `documentos`) para el caso de varias filas del mismo KPI.
5. **Línea 205 (redacción):** reescribir la frase "Orden de los PDFs: ..." (ver 6b).
6. **Orquestador:** confirmar o retirar "Docker 24.0.2" y la convención de página aprobada. No constan en brief.md ni en stack.md, que dicen "n/d". Es NO VERIFICABLE desde los documentos.
