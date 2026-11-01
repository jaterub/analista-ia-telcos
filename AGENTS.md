# AGENTS.md — reglas para agentes externos (Codex) en este repo

Trabajas como ejecutor de tareas acotadas que te encarga el orquestador (Claude Code). Las reglas del proyecto están en `CONTEXTO.md`; léelo antes de empezar.

## Codificación (Windows)
- Lee siempre en UTF-8: `Get-Content -Encoding UTF8 <archivo>`.
- Escribe siempre en UTF-8 sin BOM: `[System.IO.File]::WriteAllText("<ruta>", $texto, (New-Object System.Text.UTF8Encoding $false))`.
- Comprueba al terminar que las tildes y eñes del archivo escrito se leen bien.

## Alcance
- Escribe solo en la ruta de salida que indique el encargo.
- No modifiques `CLAUDE.md`, `CONTEXTO.md`, `AGENTS.md`, `.claude/` ni `memory/`.
- Sin commits ni push: eso lo hace el orquestador.

## Datos y fuentes
- Solo información pública; nunca datos corporativos.
- Cada dato con su fuente (URL y, si es un PDF, página del PDF). Sin fuente: `n/d`.
- No descartes en silencio: lo que no cumpla va con etiqueta y motivo.

## Estilo
- ponytail: lo mínimo que resuelve la tarea, sin pasos ni texto de más.
