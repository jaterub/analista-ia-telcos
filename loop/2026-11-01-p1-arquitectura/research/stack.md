# Research: stack, calidad de código, precios y segundo modelo (T1)

Fecha de consulta de todas las fuentes: **1-nov-2026**. Etiquetas: [HECHO] con fuente · [INFERENCIA] deducción mía · [HUECO] no confirmado.
Nota de método: la sesión solo deja abrir URLs que salen en buscadores; varias páginas oficiales (p. ej. GitHub de pgvector/PyMuPDF/pdfplumber) no se pudieron abrir y se usaron la ficha en PyPI/PGXN, el paper o una comparativa. Se indica en cada caso.

## 0. Tabla resumen (pieza → candidatas → recomendación)

| # | Pieza | Candidatas | Recomendación | Fuente principal |
|---|---|---|---|---|
| 1 | Lectura de PDF con tablas | Docling · pdfplumber · PyMuPDF · Camelot · unstructured · Tabula | **Docling** (MIT, TableFormer, página por elemento) + **pdfplumber** como contraste de cifras en P2 | arxiv.org/html/2501.17887v1 · camelot-py.readthedocs.io/en/latest/user/comparison.html |
| 2 | Almacén de datos (ruta DATOS) | Postgres · DuckDB | **Postgres** (mismo contenedor que pgvector), rol solo-lectura + validador sqlglot | duckdb.org/docs/2.0/connect/concurrency · dev.to/datanestdigital/...sqlglot-418j |
| 3 | Enrutador dato/texto | Haiku 5.5 con salida estructurada · reglas · LangGraph como contenedor | **Haiku 5.5 + JSON schema** dentro de un nodo condicional de LangGraph; reglas solo como atajo | platform.claude.com/docs/en/build-with-claude/structured-outputs |
| 4a | Embeddings locales | BGE-M3 · multilingual-e5-large · Qwen3-Embedding-0.6B | **BGE-M3** (MIT, 1024 dim, 8192 tok); Qwen3-0.6B como reto en el golden set | wz-it.com/en/blog/best-embedding-models-german/ · mteb-leaderboard.hf.space/models/BAAI/bge-m3 |
| 4b | BBDD vectorial | pgvector · Chroma | **pgvector** (ver tabla de criterios, sec. 4) | pgxn.org/dist/vector · cookbook.chromadb.dev/core/system_constraints/ |
| 4c | Híbrida + reordenado | FTS de Postgres + RRF · pg_textsearch (BM25) · BM25 de Chroma · BGE-M3 sparse; reranker bge-reranker-v2-m3 | **FTS nativo + vector + RRF en SQL**; BM25/reranker solo si la métrica de recuperación falla | pgxn.org/dist/vector · tigerdata.com/docs/use-timescale/latest/extensions/pg-textsearch |
| 5a | Capa común de modelo | Wrapper propio mínimo · LiteLLM | **Wrapper propio** (SDK Anthropic + SDK del 2º modelo) | docs.litellm.ai/docs/index.md |
| 5b | Segundo modelo (P4) | Gemini (nivel gratuito) · OpenAI | **Gemini** (`gemini-3.8-flash`, y `gemini-3.1-flash-lite` como barato) | ai.google.dev/pricing · developers.openai.com/api/docs/models/gpt-5.4-mini |
| 5c | Enrutado de modelos | Haiku 5.5 volumen / Sonnet 5.5-Opus 5.5 razonar | Haiku 5.5 para router/extracción; Opus 5.5 o Sonnet 5.5 para SQL y respuesta, **decidido midiendo en el golden set** | platform.claude.com/docs/en/about-claude/models/optimizing-for-cost-and-intelligence |
| 6a | Agente | LangGraph · funciones Python planas | **LangGraph** (MIT) desde P4; en P3 funciones planas | github.com/langchain-ai/langgraph |
| 6b | Evals | DeepEval · Ragas · promptfoo · harness propio | **Harness propio pytest** (recall@k, MRR, exactitud de ejecución SQL) + **DeepEval** para fidelidad | techsy.io/es/blog/frameworks-evaluacion-llm-codigo-abierto |
| 6c | Registro/observabilidad | Tabla propia en Postgres · Langfuse · LangSmith | **Tabla propia** `query_log` desde P3; **Langfuse** (MIT, autoalojado) en P5 | langfuse.com/resources/engineering/clarifications.md · langchain.com/pricing |
| 7 | API, Docker, caché | FastAPI + Docker Compose; caché exacta en Postgres + prompt caching de Anthropic | **FastAPI, 1 proceso uvicorn por contenedor**, Compose (api + postgres) | fastapi.tiangolo.com/de/deployment/server-workers/ · platform.claude.com/docs/en/build-with-claude/prompt-caching |
| 8 | Calidad de código | pytest · mutmut (WSL) / cosmic-ray · bandit · semgrep · pip-audit · gitleaks · GH Actions | **Todas**; mutmut en WSL o solo en CI Linux | secciones 8 |
| 9 | Precios | ver sec. 9 | — | platform.claude.com/docs/en/about-claude/pricing · ai.google.dev/pricing |

---

## 1. Lectura de PDF con tablas

| Candidata | Pros | Contras | Fuente |
|---|---|---|---|
| **Docling** | [HECHO] Licencia MIT del código (los modelos tienen licencias propias). Tabla vía TableFormer. DoclingDocument guarda nº de página y bounding boxes. Chunker integrado. OCR. Funciona en Windows, Python ≥3.10. | [HECHO] CPU x86: mediana 0,79 s/página, media 3,1 s/página, p95 16,3 s (paper v1, 2025; versión actual puede diferir). Exportar a Markdown pierde páginas: hay que guardar el JSON. | arxiv.org/html/2501.17887v1 · github.com/docling-project/docling · github.com/docling-project/docling/discussions/1012 |
| **pdfplumber** | [HECHO] MIT, sin Java, control fino (tolerancias, recorte por bbox, depuración visual) | [HECHO] Sin soporte de PDF escaneado; tablas sin bordes solo parcial; necesita ajustes por maquetación | camelot-py.readthedocs.io/en/latest/user/comparison.html · invoicedataextraction.com/blog/python-pdf-table-extraction-invoices |
| **PyMuPDF** | [HECHO] Rápido (inferencia: es C/MuPDF; sin benchmark citado) | [HECHO] AGPL o licencia comercial; buscador de tablas solo geométrico | pymupdftest.readthedocs.io/en/stable/intro.html · camelot-py.readthedocs.io/... |
| **Camelot** | [HECHO] MIT; modo lattice bueno con bordes | [HECHO] modo stream (sin bordes) puede bajar a ~50-70 % de acierto (fuente de terceros, facturas); necesita Ghostscript en la imagen; una página cada vez | invoicedataextraction.com/blog/python-pdf-table-extraction-invoices |
| **unstructured** | [HECHO] Apache 2.0 | [HECHO] Tablas como HTML/texto, no DataFrame. No evaluada más a fondo | camelot-py.readthedocs.io/... |
| **Tabula (tabula-py)** | [HECHO] MIT, rápida en plantillas fijas | [HECHO] Requiere Java; sin PDF escaneado | camelot-py.readthedocs.io/... |

**Recomendación: Docling** como lector único (texto + tablas + página por elemento → alimenta ruta DATOS y ruta TEXTO). Guardar el `DoclingDocument` en JSON, no en Markdown. Contraste de cifras clave con pdfplumber en P2 (muestra pequeña). PyMuPDF queda descartado por AGPL salvo que se decida otra cosa.
- [INFERENCIA] Con la media de 3,1 s/página, 1.000 páginas ≈ 52 min de CPU x86. Nº total de páginas del corpus: depende de T2 (n/d aquí).
- [HECHO] `page_no` parece 1-based (el ejemplo del maintainer da "Page Numbers: [1]" para la 1.ª página; no lo declara explícitamente). Fuente: discussions/1012. [INFERENCIA] Coincide con la convención "página PDF 1-based" del plan; verificar en P3 con un PDF real.

## 2. Ruta de DATOS: almacén, text-to-SQL, validación

| Candidata | Pros | Contras | Fuente |
|---|---|---|---|
| **Postgres** (mismo servidor que pgvector) | [INFERENCIA] Una sola BBDD en todo el sistema (ponytail); rol con solo `SELECT` como barrera real de seguridad; mismo contenedor Docker | Requiere Docker/Postgres en el PC: **n/d** (brief) | brief.md |
| **DuckDB** | [HECHO] Embebido, sin servidor. Varios procesos pueden leer en modo `READ_ONLY` | [HECHO] Un solo proceso puede escribir; varios workers en lectura-escritura quedan fuera del modelo soportado (Quack/DuckLake en 2.0). [HECHO] En DuckDB `read_csv`/`read_parquet` son SELECT con acceso a disco: hay que bloquearlos por nombre | duckdb.org/docs/2.0/connect/concurrency · dev.to/datanestdigital/a-read-only-sql-guard-for-llm-generated-queries-with-sqlglot-418j |

**Recomendación: Postgres**, tabla larga `kpi_values(telco, periodo, kpi, valor, unidad, perímetro, documento, página_pdf, origen)`. Porqué: cada fila lleva documento+página (contrato de cita) y se evita una segunda tecnología. DuckDB sería válido para prototipar la extracción; no aporta a producción aquí.

**Validación de la consulta generada** [HECHO, fuente: dev.to/datanestdigital/...sqlglot-418j, artículo de un autor individual]:
1. Parsear con sqlglot (dialecto del motor) y rechazar lo no parseable, varias sentencias o raíz distinta de SELECT.
2. Recorrer todo el árbol: rechazar nodos de escritura (CTE con INSERT, SELECT INTO, COPY, DROP…).
3. Lista blanca de tablas (excluyendo nombres de CTE).
4. Tope de filas que el modelo no pueda subir (envolver en SELECT … LIMIT y ejecutar el SQL regenerado).
5. Rol de BBDD de solo lectura (el artículo lo presenta como complemento obligatorio).
6. Fijar la versión de sqlglot y mantener tests de consultas maliciosas: los nombres de nodos cambian entre versiones.
7. Evaluación: comparar **resultados** con consultas de referencia (no texto SQL), redondeando decimales.
- [INFERENCIA] Más allá del SQL válido, la capa de respuesta debe devolver `n/d` si la consulta da 0 filas (regla propia del proyecto: sin fuente → `n/d`).

## 3. Enrutador dato/texto

| Opción | Pros | Contras | Fuente |
|---|---|---|---|
| **Clasificador con Haiku 5.5 + salida estructurada** | [HECHO] Anthropic indica Haiku 5.5 para clasificación, extracción y enrutado. Salida JSON con enum garantizada por esquema, disponible en todos los modelos | [HECHO] Sin `minimum/maximum` en el esquema (validar confianza en código); el modelo puede cambiar mayúsculas del enum (comparar sin distinguir caso); incompatible con la función Citations de la API | platform.claude.com/docs/en/about-claude/models/overview · .../build-with-claude/structured-outputs |
| **Reglas (regex/palabras clave)** | [INFERENCIA] Coste y latencia cero, determinista | Frágil con preguntas mixtas ("¿cómo define Orange OIBDA y cuánto fue en 2025?"). Sin fuente externa | — |
| **LangGraph como router** | [HECHO] MIT, utilizable sin LangChain, ejecución duradera y humano en el bucle | [HECHO] El contenido de la página de persistencia no cubre aristas condicionales (no verificado aquí en doc oficial); es el contenedor, no el clasificador | github.com/langchain-ai/langgraph · docs.langchain.com/oss/python/langgraph/durable-execution |

**Recomendación:** nodo de LangGraph con arista condicional cuyo clasificador es Haiku 5.5 con esquema `{ruta: dato|texto|ambos|fuera_de_alcance}`. "ambos" ejecuta las dos rutas y fusiona. Reglas solo como pre-filtro opcional si el log demuestra tráfico trivial. [INFERENCIA] El enrutador se mide con sus propias preguntas del golden set (etiqueta dato/texto ya prevista en el plan, T6).
- [HUECO] No verifiqué la API exacta de aristas condicionales de LangGraph en doc oficial.

## 4. Ruta de TEXTO (RAG)

### 4.1 Troceado con metadatos
- [HECHO] Docling pasa el DoclingDocument a un chunker que emite trozos con metadatos; página por item en `chunk.meta.doc_items[].prov[].page_no`. Fuentes: arxiv.org/html/2501.17887v1 · github.com/docling-project/docling/discussions/1012 (el nombre `HybridChunker` sale en esa discusión).
- [INFERENCIA] Metadatos por trozo: `telco, periodo_fiscal, documento, tipo_doc, pagina_pdf (lista), seccion`. Tablas: indexar su Markdown como trozo propio y, en paralelo, volcarlas a `kpi_values` (ruta de datos).
- Alternativas no evaluadas a fondo (ingesta no descarta): troceado por tamaño fijo (LangChain splitters) → [INFERENCIA] rompe tablas y secciones; sin fuente.

### 4.2 Embeddings locales multilingües

| Modelo | Dim. | Tokens máx. | Licencia | Pesos FP16 aprox. | Notas |
|---|---|---|---|---|---|
| **BGE-M3** | 1.024 | 8.192 | MIT | 1,1 GB | 568M parámetros; vectores densos y dispersos con un modelo; ~35 idiomas incl. inglés, francés, alemán (el listado visible no confirma español: **[HUECO]**) |
| multilingual-e5-large | 1.024 | 512 | MIT | 1,1 GB | Corta la entrada a 512 tokens; hay que poner prefijos en la app |
| Qwen3-Embedding-0.6B | 1.024 | 32.768 | Apache 2.0 | 1,2 GB | Hermano 4B: mejor en alemán (MIRACL de 62,98) pero 8 GB y 2.560 dim. |
| Jina v3/v5 | — | — | CC BY-NC | — | **Descartado:** licencia no comercial |

Fuentes: wz-it.com/en/blog/best-embedding-models-german/ (12-oct-2026; autor vende servicios relacionados → sesgo posible; recomienda probar con datos propios) · mteb-leaderboard.hf.space/models/BAAI/bge-m3.
**Recomendación: BGE-M3.** Porqué: MIT, 1.024 dim (cabe en índice de pgvector, límite 2.000 para `vector`), contexto largo, sirve de base para el reranker de la misma familia. Reto: probar Qwen3-Embedding-0.6B con el golden set (recall@k). [HECHO] Cambiar de modelo obliga a reindexar todo; fijar revisión del modelo (wz-it). Coste de almacenamiento en pgvector: ~4,1 GB por millón de trozos de 1.024 dim sin índices (wz-it).

### 4.3 BBDD vectorial: pgvector vs Chroma (criterios con fuente)

| Criterio | pgvector | Chroma | Fuente |
|---|---|---|---|
| Versión/licencia | [HECHO] 0.8.7 (1-oct-2026), licencia PostgreSQL | [HECHO] Apache 2.0 | pgxn.org/dist/vector · trychroma.com/products/chromadb |
| Filtro por metadatos (telco/periodo/página) | [HECHO] `WHERE` SQL normal. Con índice HNSW/IVFFlat el filtro se aplica tras el índice y puede dar pocos resultados; se corrige con `hnsw.iterative_scan` (strict/relaxed_order), índices parciales o particiones | [HECHO] `where` con `==, !=, >, <, is_in, contains, regex`, `&` y `\|` (doc de la Search API, pestaña Cloud; si aplica igual en local: **[HUECO]**) | pgxn.org/dist/vector · docs.trychroma.com/cloud/search-api/filtering |
| Persistencia | [HECHO] Es Postgres: transaccional, mismo backup que los datos | [HECHO] `PersistentClient` en proceso o servidor cliente/servidor. **No es seguro entre procesos con escritores sobre la misma ruta local** | cookbook.chromadb.dev/core/ · cookbook.chromadb.dev/core/system_constraints/ |
| Docker | [HECHO] Extensión de Postgres. [HUECO] Imagen oficial de pgvector no confirmada: la búsqueda solo devolvió imágenes de terceros | [HECHO] Admite pip, npm o Docker; nodo único | trychroma.com/products/chromadb |
| Híbrida (léxico+vector) | [HECHO] Combinar con FTS de Postgres y fusionar (RRF) o cross-encoder; también pg_textsearch (BM25, PG 17/18) | [HECHO] Su página de producto declara denso+disperso (BM25, SPLADE)+texto+regex+metadatos en una consulta; [HECHO, fuente de terceros] reranking o fusión de fuentes requieren componentes externos | tigerdata.com/docs/use-timescale/latest/extensions/pg-textsearch · trychroma.com/products/chromadb · edana.ch/en/2026/02/08/... |
| Escala/operación | [HECHO] Parámetros HNSW (m=16, ef_construction=64, ef_search=40 por defecto) e IVFFlat; límite de indexado 2.000 dim `vector`, 4.000 `halfvec` | [HECHO] Escala vertical en nodo único; el segmento del índice carga en memoria; dimensión fija tras la 1.ª escritura; cambiar de modelo = colección nueva. [HECHO, tercero] sin réplica nativa | pgxn.org/dist/vector · cookbook.chromadb.dev/core/system_constraints/ · edana.ch/... |
| Valor en producción | [INFERENCIA] Reutiliza operación de Postgres ya conocida (backup, roles, monitorización); un solo sistema para datos+vectores+log | [HECHO, tercero] Adecuada para PoC y validación rápida; para producción el artículo recomienda evaluar pgvector entre otros | edana.ch/... |
| Valor en entrevistas | [INFERENCIA] Permite explicar filtro+índice, recall vs filtros, SQL con rol de solo lectura, RRF; sin fuente | [INFERENCIA] Arranque más rápido; menos que explicar | — |

**Recomendación: pgvector.** Porqué: (a) filtros por telco/periodo son predicados SQL, (b) un solo Postgres sirve a vectores, tablas de KPIs y log por consulta, (c) hay una escritura concurrente segura que Chroma local no garantiza entre procesos. Coste: mantener Postgres+Docker en el PC (**n/d** si ya lo tiene; preguntar antes de P3, según brief). Chroma queda como plan B para un prototipo de 1 día.
Nota de proceso (lente 3): [INFERENCIA] si T2 mide que el corpus entero cabe en el contexto (los modelos Claude actuales tienen 1M tokens, ver sec. 9), la BBDD vectorial sería innecesaria para el uso real; se mantiene porque el objetivo del proyecto es aprender/mostrar RAG. Debe declararse como decisión de aprendizaje en `arquitectura.md`.

### 4.4 Recuperación híbrida y reordenado

| Opción | Hechos | Fuente |
|---|---|---|
| FTS nativo de Postgres | [HECHO] Configuraciones de idioma predefinidas; [HECHO, nota mía de conocimiento general, no de la página] `ts_rank` no es BM25 | postgresql.org/docs/current/textsearch-intro.html |
| pg_textsearch (BM25) | [HECHO] Índice BM25 por idioma (`english`, `german`, `french`, `simple`…); un índice parcial por idioma en tablas multilingüe; autoalojado: Postgres 17/18 + `shared_preload_libraries`; ejemplo con pgvector y RRF. [HUECO] Licencia y soporte en Docker no indicados | tigerdata.com/docs/use-timescale/latest/extensions/pg-textsearch |
| BGE-M3 sparse | [HECHO] Un modelo da denso+disperso | wz-it.com/... |
| Reranker bge-reranker-v2-m3 | [HECHO] Apache-2.0, multilingüe, `FlagReranker(..., use_fp16=True)`. [HUECO] Tamaño y velocidad en CPU no figuran | huggingface.co/BAAI/bge-reranker-v2-m3 (README) |

**Recomendación:** empezar con denso (BGE-M3) + FTS + RRF en SQL; añadir pg_textsearch o reranker **solo si** recall@k del golden set lo pide (orden de construcción de CONTEXTO.md: mejorar solo la pieza que falla). [INFERENCIA] Siglas como OIBDA/EBITDAaL favorecen el componente léxico.

---

## 5. Capa común de modelo y segundo modelo

### 5.1 Capa común
| Opción | Pros | Contras | Fuente |
|---|---|---|---|
| **Wrapper propio** (función `complete(modelo, mensajes, esquema)` sobre SDK de Anthropic + SDK del 2.º proveedor) | [INFERENCIA] Ponytail; sin dependencia extra; control de citas/caché | Hay que mantener 2 adaptadores | — |
| **LiteLLM** | [HECHO] Interfaz única estilo OpenAI para 100+ modelos, Router con reintentos/fallbacks, callback de coste | [HECHO] La página no declara licencia ni lista Gemini explícitamente (muestra Vertex AI Gemini). [INFERENCIA] Traducir a formato OpenAI puede ocultar funciones propias de Anthropic (caché, structured outputs) | docs.litellm.ai/docs/index.md |

**Recomendación: wrapper propio de ~1 función + un `Protocol`.** [HECHO] Relevante: structured outputs y Citations de Claude son incompatibles (error 400) — structured outputs: platform.claude.com/docs/en/build-with-claude/structured-outputs. [INFERENCIA] Por eso el contrato de cita doc+página debe construirse en la aplicación desde los metadatos del trozo recuperado, no con la función Citations; así es idéntico en Claude y Gemini.

### 5.2 Segundo modelo: Gemini (gratis) vs OpenAI (de pago)

| Aspecto | Gemini | OpenAI |
|---|---|---|
| Modelos candidatos y precio ($/MTok in/out, estándar) | `gemini-3.8-flash` 0,75 / 3,75 · `gemini-3.1-flash-lite` 0,25 / 1,50 · `gemini-3.5-flash-lite` 0,30 / 2,50 · `gemini-3.1-pro-preview` 2,00 / 12,00 (≤200k) — ai.google.dev/pricing?hl=fi | `gpt-5.4-mini` 0,75 / 4,50 (400k contexto) — developers.openai.com/api/docs/models/gpt-5.4-mini · `gpt-5.5` 5,00 / 30,00 (1,05M contexto) — developers.openai.com/api/docs/models/gpt-5.5 · `gpt-5.4-nano` 0,20 / 1,25 (morphllm.com/openai-api-pricing, tercero, actualizado 31-oct-2026) |
| Nivel gratuito | [HECHO] Sí: `3.8-flash`, `3.6-flash`, `3.5-flash-lite`, `3.1-flash-lite`, Gemma 4, algunos de embeddings y audio. De pago: `Gemini 4 Argon` (2,00/10,00) y otros. Fuente: ai.google.dev/pricing?hl=fi · apidog.com/blog/gemini-api-free-tier/ (22-oct-2026) | [HECHO] La guía de límites lista un nivel Free (geografía permitida, tope $100/mes). [HUECO] Qué modelos incluye, límites de RPM/TPM y si España cumple: no figuran. Pasar a Build exige $5 de crédito, tope $500/mes — developers.openai.com/api/docs/guides/rate-limits |
| Límites numéricos (RPM/TPM/RPD) | **[HUECO]** Google no los publica en la doc; se consultan en AI Studio (ai.google.dev/gemini-api/docs/rate-limits vía mirror; fast.io/resources/gemini-api-rate-limit/, 16-sep-2026). Tier 1 = cuenta de facturación, tope $250 | [HUECO] Free: no listado. Build: 5.000 RPM; 1M TPM (Astra/Sol/Terra), 2M (Luna) |
| Condiciones de datos | [HECHO] Nivel gratis: Google puede usar prompts/respuestas para mejorar productos y revisores humanos pueden leerlos. En UE/UK/CH se aplican las condiciones del servicio de pago incluso a la cuota gratuita, y "productos para usuarios allí solo con servicios de pago"; la clave gratis vale para pruebas personales. Fuente: apidog.com (tercero que cita docs de Google); ai.google.dev/pricing | [HUECO] No consultado |
| Cambios de precio anunciados | [HECHO] `3.8-flash` y `3.6-flash` **duplican** el 1-ene-2027 (0,75→1,50 in; 3,75→7,50 out). `3.5-flash-lite` y `3.1-flash-lite` sin cambios previstos | [HUECO] No consultado |
| Facturación | [HECHO] Empieza al vincular cuenta y prepagar ≥$5; el uso de la API Gemini no entra en el crédito de $300 de Google Cloud; si el saldo llega a 0, las claves dejan de funcionar (HTTP 402) | [HUECO] |

**Recomendación: Gemini** (`gemini-3.8-flash` como comparable al tier medio y `gemini-3.1-flash-lite` como el barato). Porqué: (1) el golden set son 30-50 preguntas sobre datos públicos, así que el nivel gratuito basta para la comparativa de P4 y el uso real es de personal; (2) CONTEXTO.md lista "Gemini/Vertex" como hueco del CV [CONTEXTO.md]; (3) el riesgo de datos del nivel gratuito es bajo porque los documentos son públicos, pero las **preguntas** del usuario también viajarían: no meter datos de Telefónica (regla del proyecto). Contras a vigilar: límites numéricos no publicados (puede dar 429 en una pasada de evaluación; medir en P4), el aviso UE sobre uso del nivel gratis, la duplicación de precio el 1-ene-2027 para `3.8-flash` (P4 caerá en fecha por decidir; sin consecuencia si se usa `3.1-flash-lite` o el gratis).
- [INFERENCIA] OpenAI es la alternativa limpia si Gemini da 429 o sus condiciones UE chocan: `gpt-5.4-mini` a 0,75/4,50. Con 50 preguntas × ~5.000 tokens in × 500 out, coste ≈ 0,19 $ + 0,11 $ ≈ 0,3 $ por pasada (cálculo propio, tamaño de prompt asumido).
- [HUECO] "Gemini/Vertex": el producto Vertex AI no se investigó; la API de Gemini (AI Studio) no es Vertex.

### 5.3 Enrutado de modelos (lente 7)
- [HECHO] Anthropic: primero caché y batch; luego elegir el modelo más barato que cumple; compara por coste por tarea completada, no por token; Haiku 5.5 para alto volumen comprobable (GPQA Diamond 85 % vs 91 % de Opus 5.5 y Sonnet 5.5); cascada "esfuerzo bajo primero, reintentar fallos en alto" ahorró ~40 % en SWE-bench Pro (cifras internas de Anthropic, direccionales). Fuente: platform.claude.com/docs/en/about-claude/models/optimizing-for-cost-and-intelligence.
- [HECHO] La misma página trata los patrones advisor/orquestador como "uso con cautela".
- [INFERENCIA] Propuesta: Haiku 5.5 = enrutador + extracción de cifras desde tablas + preguntas triviales; Sonnet 5.5 u Opus 5.5 = text-to-SQL y respuesta final con cita; Fable 5.1 solo si Opus falla en el golden set. Se decide con la curva coste/acierto del golden set, no a priori.

---

## 6. Agente, evals, observabilidad

### 6.1 LangGraph
- [HECHO] MIT; ejecución duradera, humano en el bucle, memoria corta y larga; usable sin LangChain. Fuente: github.com/langchain-ai/langgraph.
- [HECHO] Checkpointers guardan estado por hilo (`thread_id`) y stores guardan datos entre hilos; ambos se pasan en `compile()`. Fuente: docs.langchain.com/oss/python/langgraph/durable-execution.
- Alternativa: funciones Python planas ([INFERENCIA], ponytail). **Recomendación:** funciones planas en P3, LangGraph en P4 (decidido en CONTEXTO: A8-L2).

### 6.2 Evals (métrica por pieza, lente 5)

| Candidata | Hechos | Fuente |
|---|---|---|
| **DeepEval** 4.1.5 | Apache-2.0; encaja con pytest (`assert_test`, `deepeval test run`); métricas fidelidad, relevancia, precisión/recall contextual, G-Eval. En el test de Techsy: juez secuencial, 126,8 s para 10 casos; detectó la respuesta irrelevante en 2 de 3 casos. Desarrollado por Confident AI (socio de pago de Techsy) | techsy.io/es/blog/frameworks-evaluacion-llm-codigo-abierto · analyticsvidhya.com/blog/2026/07/rag-evaluation-frameworks/ |
| **Ragas** 0.4.3 | Apache-2.0; métricas RAG (fidelidad, relevancia, recall de contexto). **Riesgo:** repo movido a `vibrantlabsai`, sin commits desde 24-feb-2026; exige fijar `langchain-community<0.4`; dio 0,777 a una respuesta incorrecta | techsy.io/... |
| **promptfoo** 0.121.20 | MIT; YAML + `npx promptfoo eval`; sale con código ≠0 si falla; instalación vía npm; dio 0,800 a una respuesta incorrecta | techsy.io/... |
| **Harness propio (pytest)** | [INFERENCIA] recall@k y MRR sobre el golden set (doc+página correctos), exactitud de ejecución SQL, tasa de abstención correcta; deterministas, sin juez LLM | — |

Todas las comparativas: 10 casos, juez en la nube, **no** prueban juez local (techsy.io); datos de métricas de acierto solo orientativos.
**Recomendación:** harness propio para recuperación (¿está la página correcta entre los k primeros?), exactitud SQL (cifra exacta) y abstención; **DeepEval** solo para fidelidad de la generación (¿lo respalda el contexto?). Ragas descartado por mantenimiento dudoso.
- [INFERENCIA] Métrica por pieza: troceado (¿el trozo conserva la tabla y su página?), recuperación (recall@k), generación (fidelidad + cita correcta), router (accuracy de ruta), SQL (exactitud de resultado).

### 6.3 Observabilidad y registro por consulta (lente 6)

| Candidata | Hechos | Fuente |
|---|---|---|
| **Tabla propia `query_log` en Postgres** | [INFERENCIA] Campos: pregunta, ruta, fragmentos+scores, contexto enviado, SQL generado, respuesta, citas, tokens, coste, latencia, versión del corpus. Cubre lente 6 sin infraestructura nueva | — |
| **Langfuse** | [HECHO] Núcleo MIT; todas las funciones abiertas desde 4-jun-2025 (LLM-as-judge, playground, colas de anotación); autoalojado sin cuota. Stack: web, worker, ClickHouse, Postgres, Redis, almacenamiento de blobs; Docker Compose en una VM sin alta disponibilidad. Acción de GitHub para evaluar en CI y bloquear merges. Datasets desde trazas | langfuse.com/resources/engineering/clarifications.md |
| **LangSmith** | [HECHO] Developer: $0, 5.000 trazas base/mes, 1 asiento. Plus: $39/asiento/mes, 10.000 trazas. Autoalojado solo aparece bajo Enterprise | langchain.com/pricing |

**Recomendación:** tabla propia desde P3 (valor en entrevistas: saber qué se registra); **Langfuse** en P5 por ser MIT y autoalojable, aceptando su peso (ClickHouse+Redis+blobs en el PC). LangSmith descartado: autoalojado solo en Enterprise y datos en su nube.
- [HUECO] No verifiqué el consumo de RAM del stack Langfuse en un PC doméstico.

---

## 7. FastAPI + Docker y caché (lente 7)
- [HECHO] FastAPI: `fastapi run --workers N` / `uvicorn --workers N` para varios procesos en una máquina; **en contenedores, un proceso uvicorn por contenedor** y escalar con más contenedores. Fuente: fastapi.tiangolo.com/de/deployment/server-workers/.
- [HECHO] Docker Desktop es gratuito para uso personal, educación, OSS no comercial y empresas <250 empleados y <$10 M de ingresos. Fuente: docs.docker.com/subscription/desktop-license/. [INFERENCIA] Aplica al PC personal de Javier; no usarlo con datos de Telefónica.
- [INFERENCIA] Composición mínima: `postgres` (con pgvector) + `api`; P5 añade Langfuse; K8s aparcado (decidido).
- Alternativa: DuckDB embebido + un contenedor (descartada arriba por concurrencia).
- **Caché:** (a) [HECHO] caché de prompt de Anthropic: mínimo 512 tokens en Haiku 5.5, Sonnet 5.5, Opus 5.5; TTL 5 min por defecto (1 h con coste extra); lectura 0,1× del precio de entrada (Haiku 5.5), 0,05× (Opus 5.5 y Sonnet 5.5); escritura 1,25× (5 min) o 2× (1 h) — platform.claude.com/docs/en/build-with-claude/prompt-caching; (b) Batch API −50 % (acumulable con caché) — platform.claude.com/docs/en/about-claude/pricing; útil para pasadas del golden set; (c) [INFERENCIA] caché exacta de respuestas en Postgres con clave (pregunta normalizada, versión del corpus); invalidar al reindexar. Caché semántica: no investigada → **[HUECO]**.
- [HECHO, fuente corregida por el orquestador según el propio researcher] Los tokenizadores de Claude 4.7+ generan ~30 % más tokens para el mismo texto (platform.claude.com/docs/en/about-claude/pricing); esto afecta a las estimaciones de coste.

---

## 8. Calidad de código (examen, P3.1)

| Herramienta | Qué hace | Hechos | Fuente |
|---|---|---|---|
| **pytest** | Tests | [HECHO] Referencia oficial de uso en CI de GitHub | docs.pytest.org/en/stable/getting-started.html (enlace de resultado; no abierto) · docs.github.com/en/actions/use-cases-and-examples/building-and-testing/building-and-testing-python |
| **mutmut** 3.8.1 (28-oct-2026) | Mutation testing | [HECHO] BSD-3-Clause; Python ≥3.10; ejecuta pytest; **no corre en Windows nativo (necesita `fork`), usar WSL** | github.com/boxed/mutmut · pypi.org/project/mutmut |
| cosmic-ray | Mutation testing | [HECHO, fuente del competidor pytest-gremlins] Multiplataforma, distribuible con Celery, no respeta fixtures/markers de pytest | pytest-gremlins.readthedocs.io/en/latest/guide/comparison/ |
| mutatest | Mutation testing | [HECHO, misma fuente] Inactivo: última versión 3.1.0 en 2022 → **descartada** | ídem |
| pytest-gremlins | Plugin pytest | [HECHO, misma fuente, de su propio mantenedor] Nativo pytest y multiplataforma; no evaluado más | ídem |
| **bandit** | Linter de seguridad estático | [HECHO] Apache-2.0; construye AST y ejecuta plugins; imagen en `ghcr.io/pycqa/bandit/bandit`. [HUECO] lista de comprobaciones no verificada en doc | github.com/pycqa/bandit |
| **semgrep (CE)** | SAST con reglas | [HECHO] CLI LGPL 2.1; reglas con su propia licencia (comprobar `metadata.license`); en CI por Docker `semgrep/semgrep` o `semgrep scan --config`. En CE **no** hay SCA, secretos ni comentarios en PR | docs.semgrep.dev/deployment/oss-deployment |
| **pip-audit** 2.10.1 (10-jun-2026) | Dependencias vulnerables | [HECHO] Apache 2.0; fuentes PyPI (por defecto) y OSV; sale con 1 si hay vulnerabilidades; acción `pypa/gh-action-pip-audit`; hook pre-commit | pypi.org/project/pip-audit |
| **gitleaks** | Secretos | [HECHO, tercero] MIT; analiza historial y hook pre-commit; la acción oficial `gitleaks-action@v2` pide secret `GITLEAKS_LICENSE` en **repos de organización**; no se dice si cuentas personales la necesitan | appsecsanta.com/gitleaks (actualizado 18-jun-2026) |
| **GitHub Actions** | CI | [HECHO] Runners `ubuntu-latest`/`windows-latest`; plantilla Python con caché pip y matriz. [HECHO] Cuentas individuales Free/Pro: "más de 33 horas" de cómputo incluidas (el número de minutos no figura). Repos públicos: gratis | docs.github.com/en/actions/use-cases-and-examples/building-and-testing/building-and-testing-python · github.com/resources/insights/2026-pricing-changes-for-github-actions |

**Recomendación:** pytest + mutmut + bandit + pip-audit + gitleaks como puerta en CI (Linux, `ubuntu-latest`, donde mutmut sí funciona); semgrep con reglas del registro como complemento de seguridad. En el PC de Javier (Windows), mutmut solo en WSL; el CI es la vía principal. Gitleaks: usar el binario o `pre-commit` si la acción pide licencia.
- [INFERENCIA] El repo es privado (brief): los minutos de Actions cuentan contra la cuota; comprobar en la cuenta antes de P3.1.
- [HUECO] Valor de "minutos incluidos" para GitHub Free individual en 2026: no figura en las páginas abiertas.

---

## 9. Precios oficiales (USD por millón de tokens, consulta 1-nov-2026)

### Claude (platform.claude.com/docs/en/about-claude/pricing; la página no lleva fecha y remite a claude.com/pricing)
| Modelo | ID API (de .../models/overview) | Entrada | Salida | Lectura de caché | Batch (in/out) |
|---|---|---|---|---|---|
| **Claude Haiku 5.5** (≤100k tokens de prompt) | `claude-haiku-5-5` | 0,10 | 0,50 | 0,01 | 0,05 / 0,25 |
| Claude Haiku 5.5 (>100k) | — | 0,50 | 2,50 | 0,05 | 0,25 / 1,25 |
| **Claude Sonnet 5.5** | `claude-sonnet-5-5` | 2 | 10 | 0,10 | 1 / 5 |
| **Claude Opus 5.5** (modelo por defecto según Anthropic) | `claude-opus-5-5` | 4 | 20 | 0,20 | 2 / 10 |
| Claude Fable 5.1 (el más capaz) | `claude-fable-5-1` | 10 | 50 | 0,25 | 5 / 25 |
| Claude Haiku 4.5 (anterior) | — | 1 | 5 | 0,10 | 0,50 / 2,50 |
Los cuatro modelos 5.x: contexto 1M tokens, salida máx. 128K (.../models/overview). Mythos, Opus 5/4.x, Sonnet 5/4.x existen en la tabla con precios mayores; omitidos por no ser candidatos. [HECHO] Haiku 5.5 cobra la tarifa alta si el prompt (incluida caché) supera 100k tokens.

### Gemini (ai.google.dev/pricing?hl=fi, estándar, de pago; mismo texto recogido 1-nov-2026)
| Modelo | Gratis | Entrada | Salida |
|---|---|---|---|
| `gemini-3.8-flash` | Sí | 0,75 | 3,75 (duplica el 1-ene-2027) |
| `gemini-3.6-flash` | Sí | 0,75 | 3,75 (duplica el 1-ene-2027) |
| `gemini-3.5-flash-lite` | Sí | 0,30 | 2,50 |
| `gemini-3.1-flash-lite` | Sí | 0,25 (texto/imagen/vídeo) | 1,50 |
| `gemini-3.1-pro-preview` | No | 2,00 (≤200k) / 4,00 (>200k) | 12,00 / 18,00 |
| `gemini-4-argon` | No | 2,00 | 10,00 |
[HUECO] La página está truncada tras el 100.000.º carácter (143.326 en total); solo se leyó lo anterior y lo posterior en parte. Las tarifas Batch/Flex/Priority y de embeddings no se recogieron.

### OpenAI (candidato rechazado como 2.º modelo)
| Modelo | Entrada | Salida | Fuente |
|---|---|---|---|
| `gpt-5.4-mini` | 0,75 (caché 0,075) | 4,50 | developers.openai.com/api/docs/models/gpt-5.4-mini |
| `gpt-5.5` | 5,00 (caché 0,50) | 30,00 (×2 entrada y ×1,5 salida si >272K tokens) | developers.openai.com/api/docs/models/gpt-5.5 |
| `gpt-5.4-nano` | 0,20 | 1,25 | morphllm.com/openai-api-pricing (tercero; verificado por ellos con docs el 29-31-oct-2026) |
| `gpt-5-mini` / `gpt-5-nano` | 0,25 / 0,05 | 2,00 / 0,40 | misma fuente de tercero |
| `gpt-6-*` (Astra 10/50, Sol 2/10, Terra 1/5, Luna 0,10/0,50) | — | — | misma fuente de tercero; no verificado en doc oficial |

---

## Contradicciones / avisos entre fuentes
- El apidog.com (22-oct-2026) y la doc de Google coinciden en que ya no hay tabla numérica de RPM/RPD gratuitos; fast.io (16-sep-2026) también. Por tanto el "límite actual" gratuito de Gemini es **n/d** numéricamente; solo está el listado de modelos con precio "gratis".
- Una versión en caché de la tabla de precios de Gemini (variante `?authuser=3`) mostraba modelos 2.5/2.0/1.5 con precios antiguos; está obsoleta frente a `?hl=fi`, que lista 3.x/4. Se descartó la antigua.
- Ragas se promociona en comparativas de julio-2026, pero Techsy (oct-2026) indica repo sin commits desde feb-2026 → se prioriza la fuente más reciente.

## Huecos
- [HUECO] Docker/Postgres en el PC de Javier: n/d (brief); decide si pgvector entra tal cual o se necesita instalación.
- [HUECO] Límites numéricos del nivel gratuito de Gemini (RPM/TPM/RPD): no publicados; consultarlos en AI Studio con la clave de Javier.
- [HUECO] Nivel gratuito de OpenAI: modelos incluidos, límites y elegibilidad en España no figuran.
- [HUECO] Español en la lista de idiomas de BGE-M3: el listado abierto muestra "~35 idiomas" y "+16 más"; no se confirma el español. Verificar en la ficha del modelo.
- [HUECO] Imagen Docker oficial de pgvector y soporte de pg_textsearch en Docker.
- [HUECO] Tamaño/velocidad en CPU de bge-reranker-v2-m3; latencia real de Docling en el hardware de Javier (el único dato es de un paper de 2025 con CPU x86 genérica).
- [HUECO] Aristas condicionales de LangGraph: no verificadas en doc oficial.
- [HUECO] Licencia de LiteLLM y de pg_textsearch: las páginas abiertas no la dan.
- [HUECO] Caché semántica y consumo de RAM de Langfuse no investigados.
- [HUECO] Páginas oficiales de GitHub de pgvector, PyMuPDF y pdfplumber no abiertas (fuera de la lista permitida de la sesión); datos de PGXN, readthedocs y comparativas.
- [HUECO] Minutos de GitHub Actions incluidos en GitHub Free individual.
- [HUECO] Tarifas Batch, embeddings y límites de Gemini de pago no recogidos (página truncada).
- [HUECO] Vertex AI no investigado.

## Fuentes
- https://platform.claude.com/docs/en/about-claude/pricing
- https://platform.claude.com/docs/en/about-claude/models/overview
- https://platform.claude.com/docs/en/about-claude/models/optimizing-for-cost-and-intelligence
- https://platform.claude.com/docs/en/build-with-claude/prompt-caching
- https://platform.claude.com/docs/en/build-with-claude/structured-outputs
- https://ai.google.dev/pricing?hl=fi
- https://ai.google.dev/gemini-api/docs/pricing?authuser=3 (descartada, obsoleta)
- https://ai.google.dev/gemini-api/docs/rate-limits?authuser=7&hl=hi
- https://apidog.com/blog/gemini-api-free-tier/
- https://fast.io/resources/gemini-api-rate-limit/
- https://developers.openai.com/api/docs/models/gpt-5.4-mini
- https://developers.openai.com/api/docs/models/gpt-5.5
- https://developers.openai.com/api/docs/guides/rate-limits
- https://morphllm.com/openai-api-pricing
- https://pgxn.org/dist/vector
- https://cookbook.chromadb.dev/core/ · https://cookbook.chromadb.dev/core/system_constraints/
- https://docs.trychroma.com/cloud/search-api/filtering · https://www.trychroma.com/products/chromadb
- https://edana.ch/en/2026/02/08/pros-and-cons-of-chromadb-for-retrieval-augmented-generation-great-for-getting-started-but-risky/
- https://arxiv.org/html/2501.17887v1 · https://github.com/docling-project/docling · https://github.com/docling-project/docling/discussions/1012
- https://pymupdftest.readthedocs.io/en/stable/intro.html
- https://camelot-py.readthedocs.io/en/latest/user/comparison.html
- https://invoicedataextraction.com/blog/python-pdf-table-extraction-invoices
- https://wz-it.com/en/blog/best-embedding-models-german/
- https://mteb-leaderboard.hf.space/models/BAAI/bge-m3
- https://huggingface.co/BAAI/bge-reranker-v2-m3/blame/main/README.md
- https://www.tigerdata.com/docs/use-timescale/latest/extensions/pg-textsearch
- https://www.postgresql.org/docs/current/textsearch-intro.html
- https://duckdb.org/docs/2.0/connect/concurrency
- https://dev.to/datanestdigital/a-read-only-sql-guard-for-llm-generated-queries-with-sqlglot-418j
- https://docs.litellm.ai/docs/index.md
- https://github.com/langchain-ai/langgraph · https://docs.langchain.com/oss/python/langgraph/durable-execution
- https://techsy.io/es/blog/frameworks-evaluacion-llm-codigo-abierto · https://www.analyticsvidhya.com/blog/2026/07/rag-evaluation-frameworks/
- https://langfuse.com/resources/engineering/clarifications.md · https://www.langchain.com/pricing
- https://fastapi.tiangolo.com/de/deployment/server-workers/ · https://docs.docker.com/subscription/desktop-license/
- https://github.com/boxed/mutmut · https://pypi.org/project/mutmut · https://pytest-gremlins.readthedocs.io/en/latest/guide/comparison/
- https://github.com/pycqa/bandit · https://docs.semgrep.dev/deployment/oss-deployment · https://pypi.org/project/pip-audit · https://appsecsanta.com/gitleaks
- https://docs.github.com/en/actions/use-cases-and-examples/building-and-testing/building-and-testing-python · https://docs.github.com/en/get-started/learning-about-github/faq-about-changes-to-githubs-plans · https://github.com/resources/insights/2026-pricing-changes-for-github-actions
- Internos: /home/claude/analista-ia-telcos/CONTEXTO.md · brief.md · plan.md
