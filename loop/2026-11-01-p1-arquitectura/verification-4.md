# Verification 4 — drafts/arquitectura.md (re-verificación tras cierre de critique-1)
Resultado: **PASS**

Estado verificado: lectura del draft actual (287 líneas) y de stack.md el 2-nov-2026. El draft está en carpeta ignorada por git, así que no hay diff; la regresión (punto 5) se comprueba por contenido contra las descripciones de critique-1.md y recalculando.

## Comprobaciones
| # | Qué | Método | Esperado | Obtenido | OK |
|---|---|---|---|---|---|
| 1a | S1 declarado como HUECO en sec. 9 | lectura l.267 | punto con "HUECO, riesgo prioritario", 3 enfoques sin elegir, spike P2 con 3 PDF | l.267 (punto 10): "(a) plantilla determinista... (b) modelo barato (Haiku)... (c) carga manual", "sin elegir", "spike al inicio de P2 con 3 PDF reales" | OK |
| 1b | Sec. 3.2 ya no lo da por hecho | l.89 | marca hueco y remite a sec. 9 | "[HUECO, riesgo prioritario] El cómo ... no está diseñado; ver sec. 9, punto 10" | OK |
| 1c | Nodo T del Mermaid | l.43 | marca hueco | "HUECO: extraccion sin disenar, spike P2" | OK |
| 1d | No se diseña el mecanismo (DoD = declarar) | lectura | decisión diferida, sin elegir | cumple; sin diseño impuesto | OK |
| 2 | Esquema del enrutador con telco y periodo_fiscal, coherente | l.75 vs l.57, l.98, l.113 | `{ruta, confianza, telco, periodo_fiscal}` nullable; X1 y 3.3/3.4 consistentes | l.75 incluye `telco: string\|null, periodo_fiscal: string\|null` y dice que son fuente de filtros de 3.3 y 3.4 y que X1 los recibe; l.57 "filtro telco / periodo"; l.98 "cuando el enrutador los extrae"; l.113 sin telco/periodo -> TEXTO sin filtro. Sin contradicción residual | OK |
| 3a | S2-1 en Huecos, no resuelto | l.268 (punto 11) | FTS multilingüe ES/DE/EN/FR, cita stack.md §4.4, opciones sin elegir | presente, "[HUECO]", "Opciones sin elegir", "Se decide midiendo recall@k en P3" | OK |
| 3b | Respaldo de la cita §4.4 | stack.md l.112-113 | §4.4 trata idiomas del FTS | §4.4: FTS nativo "Configuraciones de idioma predefinidas"; pg_textsearch "un índice parcial por idioma en tablas multilingüe". Respalda la premisa (config por idioma). La frase "lo que degrada el componente léxico" es consecuencia inferida, no literal en la fuente | NO VERIFICABLE solo ese matiz; el hueco está declarado y la cita existe |
| 3c | S2-3 en Huecos, no resuelto | l.269 (punto 12) | `fuera_de_alcance -> n/d` sin medir, vigilado con golden set | "[HUECO] ... sin salvaguarda ... No está medido; se vigilará con las preguntas del golden set (accuracy de ruta, ... falsos fuera_de_alcance)" | OK |
| 4a | Mermaid parsea | extraje el bloque (32 líneas) y lo pasé por el parser oficial `mermaid@10` `mermaid.parse()` con jsdom | sin error | `PARSE OK true`. Control negativo (`A[x --> B`) da error de parseo, así que el parser detecta fallos. (Además revisión manual) | OK |
| 4b | Nodo T con comas/dos puntos y `->` en `[]` | parser + lectura l.43 | válido | parsea; `<br/>` todos cerrados (l.41-43, 49, 52, 57) | OK |
| 4c | Nodos destino definidos | lectura | ND0 (l.50), ND1 (l.53), ND2 (l.65, definido tras usarse en l.64; válido), KV (l.46), PV (l.45), G (l.61), ANS, LOG, V, X1, D1-D3, T, R | todos definidos; sin nodos huérfanos | OK |
| 5a | 10 piezas y citas a stack.md intactas | lectura sec. 2 (l.19-30) | 10 filas con §N | 10 filas, todas con cita a stack.md; coherentes con lo descrito en critique-1 | OK |
| 5b | Sec. 8 coste sin cambios | recálculo a mano de la tabla | 0,0375 / 0,75 / 1,50 / 3,75 / 0,28125 / 0,10; P3 7,5/15; P4 4,59/8,34; P5 0,5625; total 13,4/25,4 | coinciden con lo escrito en l.228-245 (suma 7,5+4,59+0,56+0,75=13,4; 15+8,34+0,56+1,5=25,4) | OK |
| 5c | S3 no tocado | lectura | V = "Verificación de cita" solo comprueba existencia (S3-1); `documentos`+`version`/`vigente`+hash (S3-2) | nodo V l.63 y sec. 5.1 l.158 sin cambios; `documentos`, `vigente` y reindexado por hash siguen en l.87-88 y l.172 | OK |

## Evidencia
- Parser: `node p.mjs` sobre el bloque mermaid extraído -> `PARSE OK true`. Control negativo -> "Expecting 'SQE', ... got '1'".
- Sección "Cambios ciclo 4" (l.283-286) del draft coincide con lo comprobado: S1 en 3.2 pto 2, nodo T y sec. 9 pto 10; S2-2 en 3.1; S2-1/S2-3 en sec. 9 ptos 11-12; S3 sin tocar.
- Límite del método: no hay diff porque `drafts/` está fuera de git; la comprobación de "no tocado" se basa en contenido, no en historial.

## Fallos
Ninguno.
