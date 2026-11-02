# Retro — P1 (borrador, se cierra con la entrega de todo P1)

> Borrador vivo. Hasta ahora cubre solo T4 (arquitectura). Se completa cuando T5-T8 estén hechos y se integre el entregable final (paso 6).

## Qué funcionó
- **El orden verifier→critic ahorró ciclos:** los fallos objetivos (DoD incompleto, contradicciones, citas mal atribuidas) los cazó el verifier barato antes de gastar al critic. El critic solo entró sobre un documento ya coherente y aportó el juicio que un test no da (el hueco funcional de la extracción de KPIs).
- **Declarar un hueco como red de seguridad** (3 enfoques + criterio + spike) resolvió un S1 sin diseñarlo: el loop no exige construir lo que una fase posterior decidirá, solo no esconderlo.
- **Verificar la mecánica con herramienta real:** el Mermaid se validó con el parser oficial `mermaid@10` + control negativo, no "a ojo".

## Qué falló
- **Un subagente sin la herramienta esperada reescribió un archivo entero.** El researcher no tenía `Edit` y reescribió `stack.md` con `Write`, reproduciendo de memoria las secciones previas. Salió bien (`git diff`: +33, −0), pero fue suerte: pudo haber corrompido contenido verificado. → regla [LOOP] abajo.
- **El loop necesitó 4 ciclos** (3 + 1 excepción autorizada). La causa raíz no fue mala construcción, sino que el primer draft dio por hechas piezas críticas (extracción de KPIs, slots del enrutador) sin declararlas; el critic las destapó tarde.

## Limitaciones conocidas (aceptadas, no son fallos abiertos)
- **S2 — Fiabilidad de la extracción de slots del enrutador.** El enrutador (Haiku) extrae `telco` y `periodo_fiscal`; el diseño cubre el caso `null` pero NO el **valor no-null pero erróneo** = fallo silencioso: telco mal extraída → filtro TEXTO vacío → `n/d` espurio; o `periodo_fiscal` mal extraído → respuesta citada del **año equivocado**. Mitigación registrada: el golden set (T6) incluirá preguntas que midan accuracy de extracción de ambos slots. Se construye en P4; queda anotado para entonces.
- **S3-1 — Sobre-atribución a stack.md §4.4.** El hueco de FTS multilingüe (sec. 9) dice que §4.4 "ya señaló" la degradación del brazo léxico; §4.4 respalda el FTS-por-idioma, no esa frase literal. La inferencia es sólida pero debe re-etiquetarse `[INFERENCIA]` cuando se integre a `docs/` en el paso 6.
- **S3-2 — Tensión spike-P2 vs "P2 = 0 $ / sin API".** Si el spike de extracción de KPIs prueba el enfoque Haiku, P2 deja de ser estrictamente 0 $ (céntimos). Aclarar en `plan-fases.md` (T8) que ese gasto mínimo del spike es la excepción.

## Reglas nuevas
- **[LOOP] Criterio de parada del loop = "nada grave abierto y todo lo importante registrado", no "cero hallazgos".** Un APPROVE con S2/S3 declarados como limitaciones conocidas y con mitigación anotada en el plan/retro es un cierre válido; perseguir cero hallazgos quema ciclos sin aumentar la calidad real. Porqué: el valor del critic es destapar y registrar riesgos, no eliminarlos todos antes de avanzar.
- **[LOOP] Al invocar un subagente, comprobar que tiene las herramientas que su tarea exige** (p. ej. `Edit` para tocar un archivo existente sin reescribirlo); si no, o se le dan, o se le instruye explícitamente a no reescribir el resto y el orquestador valida con `git diff` antes de construir encima. Porqué: un `Write` de reemplazo sobre un archivo ya verificado puede corromperlo en silencio.
- **[PROYECTO] La extracción de KPIs de tablas PDF es el riesgo prioritario de la ruta DATOS** y se decide en un spike al inicio de P2 con 3 PDF reales (3 enfoques: plantilla determinista / Haiku / manual), antes de construir nada de P2.

## Para subir al OO
- Las dos reglas `[LOOP]` → `memory/feedback.md` (hecho) → PR al OO cuando se recojan las lecciones de P1.
