# Critique ciclo 1 — arquitectura.md (P1, T4)

Veredicto: **REVISE**

Conteo: 1 S1 · 3 S2 · 2 S3

Base de juicio: brief DoD punto 1 + las 7 lentes de CONTEXTO + criterio ponytail + "producción real / defensa en entrevista". No se re-verifican cifras ni trazabilidad (ya PASS en verification-3.md); esto es el juicio adversarial de diseño.

## DoD punto 1 (es lo único que cubre este entregable)
- Piezas del sistema (10) con herramienta + porqué + alternativa descartada: **CUMPLE** — sec. 2, tabla de 10 piezas; cada una con fila de stack.md y alternativa.
- Capa común de modelo (Claude/Gemini intercambiables): **CUMPLE** — sec. 5, `Protocol LLM` + dos adaptadores + config.toml.
- Estructura de carpetas del repo: **CUMPLE** — sec. 7.
- Dos rutas con enrutador delante (datos por consulta generada / texto por RAG): **CUMPLE en forma, con un hueco S1 en la ruta DATOS** — sec. 3; ver S1-1.
- Tabla lente 1-7 → respuesta o hueco declarado: **CUMPLE** — sec. 6, con veredicto "prueba de concepto" declarado.

El DoD está cubierto formalmente. El veredicto REVISE no es por una casilla vacía, sino por un hueco funcional no declarado en la pieza más arriesgada y por tres incoherencias/omisiones que un entrevistador senior atacaría.

## Hallazgos

**S1-1: La carga de `kpi_values` (cómo una celda de tabla PDF se convierte en una fila normalizada) está dada por hecha, sin mecanismo y sin declararse como hueco.**
Dónde: sec. 3.2 punto 2 ("las tablas que Docling detecta se convierten en filas") y nodo del Mermaid "Tablas -> filas de KPI" (l.43). Sec. 9 (Huecos) no lo lista.
Por qué importa: es el punto donde mueren los proyectos de "RAG para números". Docling devuelve una tabla como rejilla de celdas; convertirla en `(telco, periodo_fiscal, kpi, valor, unidad, perimetro)` exige decidir qué celda es qué KPI, mapear el rótulo del informe al nombre canónico del glosario, y resolver perímetro/unidad/periodo. Eso es una decisión de arquitectura (¿extracción determinista con plantilla por telco?, ¿LLM Haiku como propone stack.md §5.3?, ¿mapeo manual?) y además la de mayor tasa de error de toda la ruta DATOS. El documento declara huecos menores (k, umbrales, imagen Docker) pero omite el mayor, y lo presenta como resuelto ("se convierten en filas"). Sin esto, la mitad del sistema (ruta DATOS) no es construible desde este doc y quien lo monte en P3 no sabe que ahí está el riesgo. Es el caso "describe el trabajo en vez de serlo".
Criterio de aceptación de la corrección: nombrar el enfoque de extracción+normalización (quién mapea rótulo→KPI canónico y cómo se asigna perímetro/unidad), aunque el detalle fino se difiera a P2/P3, y declararlo explícitamente como el hueco/riesgo de mayor prioridad (no basta "contraste con pdfplumber", que valida la cifra leída, no el mapeo semántico).

**S2-1: La búsqueda léxica (FTS nativo + RRF) no aborda que el corpus es multilingüe.**
Dónde: pieza 5 (l.25) y sec. 3.3; lente 2 (sec. 6) trata idiomas solo del lado embeddings.
Por qué importa: el corpus abarca 4 idiomas (Telefónica ES, Deutsche Telekom DE, Vodafone EN, Orange FR). El FTS nativo de Postgres aplica stemming por configuración de idioma (`to_tsvector('spanish'|'english'|...)`); una sola config degrada el brazo léxico de la híbrida para los demás idiomas. El propio research (§4.4, fila pg_textsearch) señala "un índice parcial por idioma en tablas multilingüe". El dense (BGE-M3) es cross-lingual, pero la híbrida presume FTS sin decir con qué config(s) ni cómo se asigna idioma por documento. Un entrevistador lo pregunta de inmediato. Afecta a recall de la ruta TEXTO, que es RRF(denso, léxico).
Criterio de aceptación: declarar cómo se trata el idioma en el FTS (config por documento vía campo `idioma`, o asumir siglas/léxico neutro y declarar la limitación), o marcarlo hueco explícito.

**S2-2: Incoherencia: el esquema del enrutador no produce `telco`/`periodo_fiscal`, pero la ruta TEXTO y los casos límite asumen que el enrutador los extrae.**
Dónde: esquema en sec. 3.1 (`{ruta, confianza}`) vs. sec. 3.3 punto 1 ("filtrados por telco/periodo_fiscal cuando el enrutador los extrae"), nodo X1 del Mermaid ("filtro telco / periodo", l.57) y sec. 3.4 ("Pregunta sin telco o sin periodo identificable").
Por qué importa: el filtro por metadatos es la ventaja central con la que se justifica pgvector/Postgres, y tres puntos del diseño dependen de que el enrutador extraiga telco/periodo, pero el esquema definido no lo hace y no hay otro paso que lo produzca. En DATOS el filtro va en el SQL generado (coherente), pero en TEXTO la extracción queda indefinida. Es una contradicción interna que rompe la ruta TEXTO tal como está dibujada.
Criterio de aceptación: o el esquema del enrutador incluye `telco?`/`periodo_fiscal?` (extracción de slots), o se define un paso de extracción aparte; que 3.1, 3.3, el Mermaid y 3.4 digan lo mismo.

**S2-3: Red de seguridad asimétrica: `fuera_de_alcance` es un camino a `n/d` sin salvaguarda, contra el principio declarado "coste de más, nunca respuesta de menos".**
Dónde: sec. 3.1 ("fuera_de_alcance: Responde n/d sin gastar más llamadas") frente a la regla de confianza baja → `ambos` (misma sec.).
Por qué importa: el diseño protege bien contra falsos positivos (confianza baja y JSON inválido → `ambos`), pero un `fuera_de_alcance` emitido con confianza alta y erróneo (pregunta respondible clasificada fuera de alcance) devuelve `n/d` sin intentar recuperación: es exactamente una "respuesta de menos", el peor fallo para un sistema cuyo valor es responder lo respondible. No está declarado como riesgo ni medido aparte (la accuracy de ruta global puede ocultar esta asimetría). Es la pregunta de entrevista "¿y si el router rechaza una pregunta válida?".
Criterio de aceptación: declarar el riesgo y cómo se acota (p. ej. medir recall de `fuera_de_alcance` como métrica propia en el golden set, o umbral que ante duda no rechace), coherente con el principio enunciado.

**S3-1: El nodo "Verificación de cita" (V) solo comprueba que el `fuente_id` existe, no que el fragmento respalde la afirmación.**
Dónde: nodo V del Mermaid (l.63) y sec. 5.1 ("Un `fuente_id` que no esté entre los trozos entregados se rechaza").
Por qué importa: en tiempo de ejecución el sistema garantiza que la cita apunta a un trozo entregado, no que ese trozo sustente la afirmación (fidelidad). La fidelidad se difiere a DeepEval en evals (offline). Es defendible en P3/ponytail, pero el nombre "Verificación de cita" sugiere una garantía que no da; para "producción real" conviene declararlo como limitación de runtime.
Criterio de aceptación: una frase que aclare que V valida existencia/pertenencia de la fuente, no respaldo semántico, y que la fidelidad se mide offline.

**S3-2: Posible sobre-ingeniería (anti-ponytail) del versionado para P3: `documentos` + `version`/`vigente` + reindexado por hash sobre 20 PDFs estáticos.**
Dónde: sec. 3.2 (tabla `documentos`, filtro `vigente = true` en cada consulta), sec. 6 lente 7.
Por qué importa: con 20 PDFs que cambian trimestralmente, en la versión mínima de P3 re-ingestar todo es trivial; la maquinaria de hash + version + vigente es operación (lente 7) que podría entrar en P6. Además obliga al text-to-SQL a conocer y filtrar `vigente` siempre, añadiendo superficie. Está declarado [INFERENCIA] y es ligero, por eso S3, no S2.
Criterio de aceptación: justificar por qué `version`/`vigente` son necesarios ya en P3 (p. ej. porque llegan 4 periodos del mismo doc), o diferir el versionado a cuando una métrica/operación lo pida (ponytail).

## Juicio global
El diseño es sobrio y bien fundado en stack.md (pgvector, wrapper propio, diferir LangGraph/Langfuse/reranker, rol de solo lectura + sqlglot): aguanta una entrevista en casi todo y declara con honestidad su estatus de "prueba de concepto". Pero no aguanta producción real tal cual en la ruta DATOS: la extracción/normalización de KPIs —su pieza más crítica y frágil— está dada por hecha y sin declarar (S1-1), y la ruta TEXTO tiene una incoherencia de extracción de filtros y un FTS ciego al multilingüismo. Son correcciones acotadas, no un rediseño.

Ruta escrita: C:/Users/34680/Documents/Claude/Projects/analista-ia-telcos/loop/2026-11-01-p1-arquitectura/critique-1.md
