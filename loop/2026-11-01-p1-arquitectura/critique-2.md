# Critique ciclo 2 — arquitectura.md (P1, T4)

Veredicto: **APPROVE** (con 1 S2 nuevo, acotado, declarable como limitación conocida)

Conteo: 0 S1 · 1 S2 (nuevo) · 2 S3 (nuevos) + 2 S3 (ciclo 1, siguen a retro)

Base de juicio: brief DoD punto 1 + 7 lentes de CONTEXTO + ponytail + "producción real / entrevista". No re-verifico cifras ni trazabilidad (PASS en verification-4.md). No re-litigo lo que Javier decidió declarar como hueco.

## DoD punto 1 (único entregable cubierto)
- 10 piezas con herramienta + porqué + alternativa: **CUMPLE** — sec. 2.
- Capa común Claude/Gemini intercambiables: **CUMPLE** — sec. 5 (`Protocol LLM` + 2 adaptadores + config.toml).
- Estructura de carpetas: **CUMPLE** — sec. 7.
- Dos rutas con enrutador (datos por SQL / texto por RAG): **CUMPLE** — sec. 3; el hueco S1 del ciclo 1 ya está declarado como riesgo prioritario (ver abajo).
- Tabla lente 1-7 → respuesta o hueco: **CUMPLE** — sec. 6, con veredicto "prueba de concepto" declarado.

## Estado de los hallazgos del ciclo 1

- **S1-1 (extracción de KPIs celda→fila):** RESUELTO como declaración deliberada. Sec. 3.2 pto 2 ([HUECO, riesgo prioritario]), nodo T del Mermaid (l.43) y sec. 9 pto 10. La red es suficiente: nombra 3 enfoques (plantilla determinista / Haiku rótulo→KPI / carga manual), criterio de decisión (fiabilidad vs coste vs esfuerzo) y disparador concreto (spike al inicio de P2 con 3 PDF reales). Cumple el criterio de aceptación del ciclo 1 (nombrar enfoque + declarar como hueco de mayor prioridad). No exijo diseño; la declaración sirve como red.
- **S2-1 (FTS multilingüe):** DECLARADO ACEPTADO — sec. 9 pto 11 ([HUECO]), con opciones (config FTS por idioma / apoyarse en denso) y medición en P3. Correcto. (Ver S3-1 nuevo sobre la atribución.)
- **S2-2 (enrutador no producía telco/periodo_fiscal):** RESUELTO — esquema de 3.1 ampliado con `telco: string|null, periodo_fiscal: string|null`; coherente con 3.3, nodo X1 del Mermaid y 3.4. (Ver S2-1 nuevo: el fix abre un flanco.)
- **S2-3 (fuera_de_alcance sin salvaguarda):** DECLARADO ACEPTADO — sec. 9 pto 12 ([HUECO]), con vigilancia de falsos `fuera_de_alcance` como métrica propia. Coherente con el principio "coste de más, nunca respuesta de menos".
- **S3-1 / S3-2 (ciclo 1):** SIGUEN ABIERTOS por decisión de Javier → van a retro. No bloquean.

## Hallazgos nuevos

**S2-1 (nuevo): el fix del enrutador introduce extracción de slots `telco`/`periodo_fiscal` cuya fiabilidad no se mide y cuyo fallo "valor no-null pero erróneo" no está declarado.**
Dónde: sec. 3.1 (esquema con `telco`/`periodo_fiscal`), sec. 3.3 pto 1 (filtro de la ruta TEXTO), sec. 3.4 (último bullet) y sec. 9 pto 12 (solo cubre `fuera_de_alcance`).
Por qué importa: el diseño trata bien el caso `null` (sin telco/periodo → sin filtro o `n/d`), pero NO el caso en que Haiku extrae un valor equivocado (p. ej. `telco=Orange` para una pregunta de Vodafone, o un periodo erróneo). En la ruta TEXTO ese filtro silencioso recupera trozos de la telco/periodo equivocados: lo más probable es `n/d` ("respuesta de menos"), pero un filtro de periodo mal extraído dentro de la misma telco puede devolver trozos plausibles de otro año que pasen el umbral y produzcan una respuesta citada pero del año equivocado. El `confianza` del esquema es de la ruta, no por slot, así que una ruta con confianza alta y slot mal extraído no se captura. La "accuracy de ruta" medida no incluye accuracy de extracción de slots. Es exactamente el flanco que abre el fix S2-2 y la pregunta que un entrevistador hace de inmediato ("¿y si el router extrae mal la telco?").
Criterio de aceptación: declarar el fallo de extracción de slot (valor no-null erróneo) como riesgo y cómo se acota (p. ej. medir accuracy de extracción de `telco`/`periodo_fiscal` en el golden set como métrica propia, o en TEXTO degradar a sin-filtro ante baja confianza). Un párrafo de hueco al estilo de los pto 11/12; corrección acotada, no rediseño.

**S3-1 (nuevo): sobre-atribución en sec. 9 pto 11.** Dice que "el corpus multilingüe... degrada el componente léxico (ya señalado en stack.md §4.4)". §4.4 respalda que el FTS nativo tiene configuraciones por idioma y que pg_textsearch hace índice BM25 por idioma para tablas multilingüe; de ahí la degradación es una INFERENCIA sólida, pero §4.4 no la "señala" literalmente. No engaña sobre el fondo (la degradación es correcta y bien fundada), pero rompe la disciplina de etiquetado del propio documento ([HECHO]/[INFERENCIA]) al presentar una inferencia como señalada por la fuente. Criterio de aceptación: reformular como [INFERENCIA desde §4.4] o quitar "ya señalado". Menor; no sube a S2 porque la sustancia es correcta.

**S3-2 (nuevo): tensión entre el spike de extracción en P2 y "P2 no usa API / 0 $".** El spike del pto 10 se sitúa "al inicio de P2 con 3 PDF reales" midiendo fiabilidad; probar el enfoque (b) (Haiku mapea rótulo→KPI) implica llamadas a API en P2, lo que roza la restricción del brief ("P1-P2 no usan API") y la celda "P2 = 0 $" de la sec. 8 (y el tope de gasto se decide en P3). Coste real: céntimos. Criterio de aceptación: o situar la parte del spike que usa API al inicio de P3, o anotar la excepción (unas llamadas Haiku de céntimos) en sec. 8 y sec. 9 pto 10. Menor.

## Notas (no bloqueantes)
- El spike del pto 10 debe aparecer en el plan por fases (T8, `docs/plan-fases.md`) para que no se olvide en P2; la arquitectura lo ancla, pero el plan es quien lo ejecuta. Verificar al integrar T8.
- Rounding P5 en sec. 8 (0,6 $ en la fila, 0,56 en la suma): ya está declarado entre paréntesis (0,0375); trivial, no es hallazgo.

## Juicio global
Los dos hallazgos de diseño del ciclo 1 (S1 extracción, S2-2 enrutador) están resueltos o declarados con red suficiente; los otros dos S2 quedan honestamente como huecos aceptados. El cierre es sólido y el documento ya sirve como base de P2/P3. El único coste del fix S2-2 es un flanco nuevo (S2-1): la fiabilidad de la extracción de slots del enrutador, sin declarar su modo de fallo "valor erróneo". Es acotado —un párrafo de hueco y una métrica— y no justifica REVISE (0 S1, <3 S2). Procede APPROVE arrastrando ese S2 como limitación conocida, o una pasada acotada de builder para declararlo y re-verificar.

Ruta escrita: C:/Users/34680/Documents/Claude/Projects/analista-ia-telcos/loop/2026-11-01-p1-arquitectura/critique-2.md
