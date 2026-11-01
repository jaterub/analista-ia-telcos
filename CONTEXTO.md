# CONTEXTO — analista-ia-telcos (proyecto cliente del OO · A8-L1)

## Objetivo
- Aprender el stack de AI Agent Engineer construyendo un sistema real: RAG + embeddings + BBDD vectorial · agentes (LangGraph) · evals y LLMOps · FastAPI + Docker.
- Datos reales y públicos (CNMV / SEC / webs de inversores), nunca solo sintéticos.
- Cada fase termina con una propuesta aplicable en Telefónica y, si es real, 1 línea en el CV de Profesión.
- Origen (1-nov, del OO): huecos del CV V1 frente a las ofertas V1 del pipeline = RAG, embeddings/BBDD vectoriales, Kubernetes, LangGraph/LangChain, Gemini/Vertex, JS/TS, MLOps/LLMOps, Copilot Studio, FastAPI. Regla: cada uno entra en el CV solo cuando haya proyecto real; cada proyecto termina con una propuesta aplicable en Telefónica (Track 4 de Profesión). CV V1 texto en `profesion/cv/` (master_cv.md + cv_v1_limpio.md).
- Meta (1-nov): salir preparado para producción real, en la situación más realista posible, y para preguntas técnicas de entrevista.
- Calidad de código en la era IA (1-nov): el código lo escribe la IA y Javier lo audita. Criterio del critic: ponytail.

## Reglas propias
- Nunca datos corporativos de Telefónica en este repo: solo información pública.
- Toda respuesta del analista cita documento y página; si no hay fuente, `n/d`.
- Niveles: lo propio aquí · lo que sirva a más proyectos → `[LOOP]` en `memory/feedback.md` → PR al OO.
- Cierre estándar de cada fase: examen de revisión de código (bugs y fallos de seguridad sembrados que Javier encuentra) + 5 preguntas de entrevista con respuesta + propuesta aplicable en Telefónica.
- Cada fase se arranca con el mini-bucle.

## Palabras propias
- **ponytail**: código mínimo como lo haría un senior (lo justo que resuelve, sin capas, variables ni abstracciones de más). Candidata a subir al OO.
- **mini-bucle**: al arrancar cada fase, compilar (objetivo en 1 frase + 3-5 hijos) → Musk (podar hijos: ¿existe ya?, ¿sobra?, ¿se fusiona?) → consolidar (al cerrar un hijo, lo útil sube al padre, lo irrelevante a ⏸️). Candidata a subir al OO.

## Check
* ✅ P0. Arranque: repo privado + OO instalado (1-nov)
* ⏳ P1. Brief, plan y arquitectura (1er encargo, nivel 3 con el loop)
  1. ✅ Intención compilada: producción real + entrevistas técnicas (1-nov)
  2. ☐ Arquitectura y stack
  3. ☐ KPIs: glosario de equivalencias + golden set
  4. ☐ Calidad de código: ponytail + diseño del examen de revisión
* ☐ P2. Datos: informes públicos de 4 telcos (carpeta según P1)
* ☐ P3. RAG + embeddings + BBDD vectorial (A8-L1)
  1. ☐ Montaje de calidad: tests, mutation testing, escáneres de seguridad y CI
* ☐ P4. Agentes con LangGraph (A8-L2)
* ☐ P5. Evals, observabilidad y costes (A8-L3)
* ☐ P6. Despliegue FastAPI + Docker (A8-L4)
