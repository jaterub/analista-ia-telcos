# feedback.md — reglas aprendidas

Formato por regla: fecha · `[PROYECTO]|[LOOP]` · regla · porqué. Máx. ~30; fusiona las parecidas.

---

- **2-nov-2026 · [LOOP] · Criterio de parada del loop = "nada grave abierto y todo lo importante registrado", no "cero hallazgos".** Un APPROVE del critic con S2/S3 declarados como limitaciones conocidas y con mitigación anotada en plan/retro es un cierre válido. *Porqué:* el valor del critic es destapar y registrar riesgos, no eliminarlos todos antes de avanzar; perseguir cero hallazgos quema ciclos sin subir la calidad real. (Origen: T4 de P1, 4 ciclos de verify/critic.)

- **2-nov-2026 · [LOOP] · Al invocar un subagente, comprobar que tiene las herramientas que su tarea exige.** Si la tarea toca un archivo existente, el agente necesita `Edit`; sin ella reescribirá con `Write` reproduciendo el resto de memoria. En ese caso, o se le da `Edit`, o se le instruye a no reescribir lo demás y el orquestador valida con `git diff` antes de construir encima. *Porqué:* un `Write` de reemplazo sobre un archivo ya verificado puede corromperlo en silencio. (Origen: el researcher reescribió `stack.md` sin `Edit`; salió +33/−0 por suerte.)
