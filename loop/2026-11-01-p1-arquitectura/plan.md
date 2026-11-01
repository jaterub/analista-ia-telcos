# Plan: P1 — arquitectura, KPIs, calidad de código y plan por fases P2-P6 del analista IA de telcos

Base: `loop/2026-11-01-p1-arquitectura/brief.md` + `CONTEXTO.md`. Rutas relativas a `/home/claude/analista-ia-telcos/loop/2026-11-01-p1-arquitectura/`.

## Ambigüedades que cambian el plan (para el orquestador)
1. **"4 trimestrales" en Vodafone.** Vodafone no publica 4 resultados trimestrales completos: año fiscal a 31-mar, resultados anuales (may) y semestrales (nov), y *trading updates* Q1/Q3 con menos detalle. A 1-nov-2026 el H1 FY27 puede no estar publicado aún. Recomendación aplicada en el plan: "4 trimestrales" = las 4 publicaciones de resultados periódicas más recientes de cada telco (incluidos trading updates), anotando el periodo fiscal real de cada una. Si Javier quiere otra cosa (p. ej. mismo trimestre natural para las 4), cambia T2 y el golden set.
2. **Convención de "página".** Página impresa en el PDF ≠ índice de página del visor. Recomendación aplicada: citar siempre índice PDF 1-based (`p. PDF`) y, si difiere, la impresa entre paréntesis. Afecta al golden set, al glosario y al contrato de citas del analista en P3.
3. **Ubicación final** de glosario y golden set: el DoD solo fija `docs/arquitectura.md`. Se resuelve en T4 (estructura de carpetas del repo); T5 y T6 escriben en `drafts/` y el orquestador integra en la ruta que fije T4.

## Tareas

### T1 — Investigación de stack, herramientas de calidad y precios [tipo: research] [depende de: —]
Hijo del check: P1.2 (y alimenta P1.4 y P3.1).
Cubre, por pieza: parsing de PDF con tablas (p. ej. pymupdf/pdfplumber/docling/unstructured), troceado, embeddings locales (modelos multilingües ES/EN/DE/FR, tamaño, licencia), BBDD vectorial **pgvector vs Chroma** (hipótesis abierta: filtros por metadatos telco/periodo/página, persistencia, operación en Docker, valor en entrevistas/producción), recuperación (híbrida BM25+vector, reranker local), capa común de modelo Claude/Gemini, LangGraph, evals y observabilidad (p. ej. Ragas/DeepEval/promptfoo, Langfuse/LangSmith), FastAPI + Docker; herramientas de calidad (pytest, mutation testing p. ej. mutmut, escáneres bandit/semgrep/pip-audit/gitleaks, CI GitHub Actions); precios oficiales de Claude API y Gemini API.
Entregable: `research/stack.md`
DoD:
- Cada pieza tiene ≥2 candidatas con pros/contras y 1 recomendación con porqué (sin decidir lo ya cerrado: Claude principal, Gemini en P4, embeddings locales, K8s al final de P6).
- pgvector vs Chroma resuelto con criterios explícitos y fuente por criterio.
- Cada afirmación técnica y cada precio lleva URL de doc oficial + fecha de consulta (1-nov-2026); precios en $/MTok input/output por modelo.
Evidencia de verificación: tabla pieza → candidatas → URL → fecha; capturas textuales de las cifras de precio con su URL.

### T2 — Localizar los PDFs públicos reales (20 documentos) [tipo: research] [depende de: —]
Hijo del check: P1.3 (bloquea golden set) y define datos de P2.
Para Telefónica, Vodafone, Orange y Deutsche Telekom: último informe anual + 4 publicaciones de resultados periódicas más recientes (ver ambigüedad 1). Por documento: telco, tipo, periodo fiscal real, fecha de publicación, URL directa al PDF (web de inversores, CNMV o SEC 20-F), nº de páginas, si se descarga desde la sesión en la nube, y además dónde está el apartado de APM/definiciones de KPIs.
Entregable: `research/fuentes-pdf.md`
DoD:
- 4 telcos × 5 documentos = 20 filas; si alguno no existe o no se alcanza, fila con `n/d` y motivo (no se rellena).
- Cada URL abierta y comprobada en esta ejecución (código HTTP o descarga correcta registrada).
- Calendario fiscal de cada telco explícito (Vodafone cierra 31-mar; resto año natural).
Evidencia de verificación: log de comprobación por URL (estado, tamaño, nº páginas), fecha de consulta.

### T3 — Definiciones de KPIs en las fuentes de cada telco [tipo: research] [depende de: T2]
Hijo del check: P1.3.
Extraer de los documentos de T2 (secciones APM / glosario / notas) la definición literal de OIBDA, EBITDAaL / EBITDA AL, FCF (y variantes: FCF reportado, FCF after leases, etc.), CAPEX (con/sin espectro, con/sin arrendamientos) y accesos/clientes (qué cuenta cada telco). Además, 30-40 cifras candidatas a golden set (valor, unidad, periodo, documento, página PDF) cubriendo las 4 telcos.
Entregable: `research/kpis-definiciones.md`
DoD:
- Matriz 6 KPIs × 4 telcos completa; cada celda con cita literal + documento + página PDF + URL, o `n/d` con motivo.
- ≥30 cifras candidatas, ≥6 por telco, con página PDF y tipo de origen (texto o tabla).
- Diferencias de perímetro y de periodo fiscal anotadas por cifra.
Evidencia de verificación: cita literal copiada del PDF por celda y por cifra; página PDF e impresa cuando difieran.

### T4 — Documento de arquitectura y stack [tipo: build] [depende de: T1]
Hijo del check: P1.2 (DoD 1).
Entregable: `drafts/arquitectura.md` (destino final `docs/arquitectura.md`)
DoD:
- Las 10 piezas del DoD (ingesta, troceado, embeddings, BBDD vectorial, recuperación, LLM, agente, evals, API, despliegue) con herramienta + porqué + alternativa descartada, cada porqué enlazado a `research/stack.md`.
- Capa común de modelo Claude/Gemini definida (interfaz mínima, cómo se cambia de proveedor) y contrato de cita (documento + página PDF; sin fuente → `n/d`).
- Estructura de carpetas del repo (incluida ubicación final de glosario, golden set y datos de P2) y diagrama Mermaid del flujo.
- Estimación de coste API por fase con precios de T1 y nota de que el tope se decide en P3.
Evidencia de verificación: cada decisión trazable a una fila de `research/stack.md`; ninguna herramienta sin fuente.

### T5 — Glosario de equivalencias de KPIs [tipo: build] [depende de: T3]
Hijo del check: P1.3 (DoD 2). Paralelizable con T6.
Entregable: `drafts/glosario-kpis.md`
DoD:
- Por KPI: definición en cada telco, veredicto comparable / comparable con ajuste / no comparable y porqué.
- Toda definición cita documento + página PDF + URL (heredada de T3); nada sin fuente.
- Sección de reglas de comparación que el analista aplicará (periodo fiscal, perímetro, NIIF 16, espectro).
Evidencia de verificación: el verifier contrasta ≥1 cita por telco contra el PDF.

### T6 — Golden set inicial (~20 preguntas) [tipo: build] [depende de: T2, T3]
Hijo del check: P1.3 (DoD 3). Paralelizable con T5.
Entregable: `drafts/golden-set.jsonl` (o `.csv`, formato máquina reutilizable en P5) + `drafts/golden-set.md` legible.
Campos: id, pregunta, respuesta, unidad, telco, periodo, documento, página PDF, URL, tipo (cifra en texto / cifra en tabla / definición / comparación / sin respuesta → `n/d`), dificultad.
DoD:
- 20 ± 2 preguntas; ≥4 por telco; ≥3 de tablas; ≥2 de comparación entre telcos; ≥2 cuya respuesta correcta es `n/d`.
- Cada respuesta verificable abriendo el PDF en la página indicada.
- Sin datos corporativos no públicos.
Evidencia de verificación: el verifier abre cada PDF y registra cifra encontrada vs esperada por pregunta (n/20 correctas).

### T7 — Calidad de código: ponytail + diseño del examen de revisión [tipo: build] [depende de: T1, T4]
Hijo del check: P1.4 (DoD 4).
Entregable: `drafts/calidad-codigo.md`
DoD:
- Definición operativa de ponytail como checklist del critic (5-10 criterios verificables con ejemplo sí/no).
- Diseño del examen: formato, nº y tipos de fallos sembrados (bugs lógicos + seguridad: inyección de prompt, secretos, SQL/path, validación de entrada, dependencias), cómo se siembran sin dejar rastro, plantilla de respuesta de Javier, rúbrica y corrección.
- Encaje con herramientas de T1 (tests, mutation testing, escáneres, CI) y con P3.1.
- Un ejemplo de examen corto completo (fragmento + fallos + solución).
Evidencia de verificación: el verifier ejecuta mentalmente o en sandbox el ejemplo y comprueba que cada fallo sembrado existe y que la solución lo corrige.

### T8 — Plan por fases P2-P6 + cierre de P1 [tipo: build] [depende de: T4, T5, T6, T7]
Hijos del check: plan por fases (DoD 5) + 5 preguntas de entrevista y propuesta Telefónica de P1.
Entregable: `drafts/plan-fases.md` + `drafts/cierre-p1.md`
DoD:
- P2-P6: por fase, objetivo en 1 frase + 3-5 hijos (mini-bucle, ya podados) + entregable + DoD + dependencias; P3 incluye hijo "montaje de calidad"; K8s como ampliación opcional al final de P6; Gemini en P4 con comparativa sobre el golden set.
- Cada fase con cierre estándar: examen (formato de T7) + 5 preguntas de entrevista + propuesta aplicable en Telefónica (sin datos corporativos).
- `cierre-p1.md`: 5 preguntas de entrevista con respuesta, basadas en decisiones reales de T4-T7, y 1 propuesta aplicable en Telefónica con datos públicos o genéricos.
- Coherencia: ninguna herramienta en el plan que no esté en `arquitectura.md`.
Evidencia de verificación: cruce herramienta-por-herramienta plan vs arquitectura; cada respuesta de entrevista enlaza a la decisión que la sostiene.

### T9 — Verificación y crítica [tipo: verificación — pasos 4-5 del loop] [depende de: T4, T5, T6, T7, T8]
Entregable: `verification-N.md` (verifier) y luego `critique-N.md` (critic, criterio ponytail).
DoD:
- Verifier: abre los PDFs y comprueba 20/20 respuestas del golden set y ≥1 cita por telco del glosario; comprueba URLs y precios de T1; casos límite (preguntas `n/d`, páginas impresas vs PDF, documento duplicado, Vodafone fuera de año natural).
- Critic: APPROVE contra DoD 1-5 del brief; S2 abiertos resueltos o declarados como limitaciones.
- Máx. 3 ciclos sumando verify + critic; si no, escala.
Evidencia de verificación: tabla pregunta → cifra esperada → cifra en PDF → OK/KO; veredictos PASS/APPROVE.

## Grafo y camino crítico
```
T1 ──► T4 ──► T7 ──┐
 └───────────►T7   ├─► T8 ──► T9
T2 ──► T3 ──► T5 ──┤
        └───► T6 ──┘
T2 ──────────► T6
```
Camino crítico: **T2 → T3 → T6 → T8 → T9** (la localización y lectura de PDFs es lo más lento y arriesgado). Rama paralela: T1 → T4 → T7.

## Paralelizable
- Ola 1: **T1 ‖ T2** (research independientes).
- Ola 2: **T4 ‖ T3** (T4 solo necesita T1; T3 solo T2).
- Ola 3: **T5 ‖ T6 ‖ T7** (T7 necesita T4; T5 y T6 necesitan T3).
- Ola 4: T8. Ola 5: T9.

Barrera investigación → formato: T4 no empieza sin `research/stack.md`; T5/T6 no empiezan sin `research/kpis-definiciones.md`; T8 no empieza sin los cuatro drafts.

## Riesgos
1. **Tablas en PDF** (cifras clave en tablas multipágina, celdas fusionadas, notas al pie; extracción y número de página erróneos) → señal temprana: en T3, % de cifras candidatas que solo están en tablas y casos donde la página impresa no coincide con la PDF; si >50% en tablas, T1 debe priorizar parser con soporte de tablas y el golden set lo refleja.
2. **Definiciones de KPIs distintas entre telcos** (EBITDAaL de Vodafone/DT vs OIBDA de Telefónica, FCF con/sin espectro y arrendamientos, "accesos" vs "clientes") → señal temprana: en T3, celdas de la matriz con `n/d` o con perímetros incompatibles; si un KPI no es comparable en ≥2 telcos, el golden set no debe incluir comparaciones directas de ese KPI.
3. **Calendarios fiscales distintos** (Vodafone FY a 31-mar, trading updates en lugar de trimestrales; resto año natural) → señal temprana: en T2, columna "periodo fiscal real" con desalineaciones; preguntas del golden set que mezclen periodos sin declararlo.
4. **Precios API cambiantes** (y nombres de modelos) → señal temprana: en T1, fecha de consulta y URL oficial por precio; el coste en `arquitectura.md` se marca "a re-verificar en P3".
5. **Acceso a PDFs desde la sesión en la nube** (webs de inversores con 403, JavaScript o PDFs no directos) → señal temprana: primeras 4 URLs de T2; si fallan, usar CNMV/SEC (20-F de Telefónica, Vodafone, Orange) o anotar `n/d` y pedir a Javier descarga manual.

## Ajustes del orquestador (1-nov, con Javier)
- T1 añade: elegir el segundo modelo del analista (P4) comparando nivel gratuito de la API de Gemini vs precio de la API de OpenAI; enrutado de modelos (barato para volumen, Claude para razonar).
- T2 se amplía a las 4 telcos: tabla telco × publicación (fecha, periodo real) × KPIs que trae cada publicación. Vodafone: sus 4 publicaciones periódicas más recientes, con periodo fiscal real.
- Cita de página: página del PDF, con la impresa entre paréntesis si no coinciden (pendiente de ok final).
- Modelos por tarea: T2 en Haiku; resto según definición de cada agente (critic en Opus). Pendiente: piloto de T2 con Codex (ChatGPT) para no gastar suscripción de Claude.

## Re-planificación por las lentes de producción RAG (1-nov, 21:19)
Motivo: Javier añade a CONTEXTO.md las lentes 1-7, obligatorias antes de construir. No cambia el grafo de tareas; cambia el contenido:
- T1 añade la **ruta de datos** (lente 1): extracción de tablas de PDF a un almacén estructurado (candidatas p. ej. DuckDB / Postgres), generación de consulta por el modelo (text-to-SQL) y **enrutador** dato/texto; métricas por pieza (lente 5); registro por consulta (lente 6); caché y modelo barato (lente 7).
- T2 añade el nº de páginas total del corpus para decidir la lente 3 (¿cabe en el contexto del modelo?).
- T4 incluye una tabla lente 1-7 → respuesta o hueco declarado.
- T6 pasa a 30-50 preguntas, etiquetadas dato/texto, con preguntas sin respuesta en el corpus.
