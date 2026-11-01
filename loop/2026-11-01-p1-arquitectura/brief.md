# Brief — P1 Brief, plan y arquitectura (1-nov-2026)

**Nivel 3** (varias tareas, investigación, decisión de diseño que condiciona P2-P6). Paso 0 "Aclarar" hecho en chat con Javier mediante compilar-intención; el loop arranca en planificar.

## Objetivo
Diseñar la arquitectura y el plan por fases (P2-P6) de un analista con IA que responde preguntas sobre los informes **públicos** de Telefónica, Vodafone, Orange y Deutsche Telekom, citando siempre documento y página.

**Meta de fondo:** que Javier salga preparado para producción real (situación lo más realista posible) y para entrevistas técnicas de AI Agent Engineer.

## Definition of Done
1. **Arquitectura y stack** (`docs/arquitectura.md`): piezas del sistema (ingesta, troceado, embeddings, BBDD vectorial, recuperación, LLM, agente, evals, API, despliegue) y, por cada una, herramienta elegida + porqué + alternativa descartada. Incluye capa común de modelo (Claude / Gemini intercambiables) y estructura de carpetas del repo. Por la lente 1, dos rutas con enrutador delante: **datos** (KPIs extraídos de las tablas a un almacén estructurado y consulta generada por el modelo) y **texto** (RAG para definiciones y notas). Incluye tabla lente 1-7 de CONTEXTO.md → respuesta o hueco declarado.
2. **Glosario de equivalencias de KPIs** (OIBDA, EBITDAaL, EBITDA AL, FCF, CAPEX, accesos) entre las 4 telcos: qué es comparable, qué no y por qué, con fuente pública citada.
3. **Set de evaluación** (lente 4): 30-50 preguntas con respuesta correcta, documento y página, sobre informes públicos reales; etiquetadas dato/texto; incluye preguntas sin respuesta en el corpus (el analista debe abstenerse). Hueco declarado: no hay usuarios reales; se usan las preguntas que haría un controller financiero (Javier).
4. **Diseño del examen de revisión de código**: formato, cómo se siembran bugs/fallos de seguridad, cómo se corrige; ponytail como criterio del critic.
5. **Plan por fases P2-P6** con los hijos de cada fase (según mini-bucle) y el cierre estándar: examen + 5 preguntas de entrevista con respuesta + propuesta aplicable en Telefónica. P1 incluye sus propias 5 preguntas y propuesta.
6. Verifier PASS + critic APPROVE.

## Restricciones
- Solo información pública; nunca datos corporativos de Telefónica.
- LLM: Claude API (key de Anthropic de Javier). Gemini como segundo modelo intercambiable en P4 (comparativa con el golden set). Kubernetes aparcado como ampliación opcional al final de P6.
- Embeddings en local (sin API). BBDD vectorial a decidir con porqué (hipótesis inicial: pgvector; alternativa Chroma).
- Datos P2: último informe anual + 4 resultados trimestrales por telco.
- Desarrollo P3-P6 en el PC personal de Javier.
- Dedicación: más de 5 h/día, 5 días/semana. No recortar alcance por horas: ordenar por dependencias y valor.
- Coste API: P1-P2 no usan API. Tope de gasto mensual se decide al llegar a P3. Precios actuales a verificar en fuente oficial.
- Código: ponytail (mínimo estilo senior). El código lo escribe la IA y Javier lo audita.
- Toda respuesta del analista: cita documento + página; sin fuente → `n/d`.

## Conexiones y herramientas que ya tiene
- GitHub (repo privado `jaterub/analista-ia-telcos`), Claude (suscripción + key API), Python, Power BI. Docker y Postgres en PC personal: **n/d** (preguntar antes de P3).

## Dónde corre
- P1: sesión en la nube (Claude Code remoto), solo documentos y búsqueda web.
- P3-P6: terminal del PC personal.

## Plan de evidencia
- Cada elección de stack: enlace a doc oficial o fuente viva, fecha de consulta.
- Glosario: definición de cada KPI citada del informe de cada telco (documento + página o URL).
- Golden set: cada respuesta con documento, página y URL del PDF; el verifier abre el PDF y comprueba cifra y página.
- Precios API: URL oficial y fecha.

## Cambios (1-nov, 21:19)
- Añadidas las lentes de producción RAG de CONTEXTO.md: arquitectura con rutas datos/texto y enrutador; set de evaluación 30-50 con preguntas sin respuesta. Motivo: Javier fija las lentes como obligatorias antes de construir.

## Contexto de solo lectura
- `CONTEXTO.md` (objetivo, reglas, palabras propias, check).
- `memory/`: no existe todavía.
