---
trigger: always_on
description: > Renombrado de `pbi-docs` a `pbi-context` el 2026-08-03 — ver la entrada fechada al final de este
---

# CLAUDE.md — pbi-context: soporte PBIP/TMDL + mejoras de contexto para agentes

> Renombrado de `pbi-docs` a `pbi-context` el 2026-08-03 — ver la entrada fechada al final de este
> archivo para el porqué. Las entradas fechadas anteriores a esa se dejan tal cual (narran hechos
> de cuando el proyecto se llamaba `pbi-docs`), consistente con el criterio de no reescribir
> historia que este mismo archivo ya aplica en otras partes.

Este archivo es la fuente de verdad para las próximas fases de desarrollo de `pbi-context`.
Léelo completo antes de proponer un plan. Trabajar siempre en **Plan Mode** antes de tocar código:
presentar el plan, esperar aprobación explícita, y solo entonces implementar por fases.

---

## Estado actual (actualizado 2026-07-15)

Las secciones 1 y 2 de este documento describían trabajo planeado que **ya está implementado y
probado** (161 tests, 0 dependencias externas). Se dejan las secciones originales abajo sin
reescribir — el razonamiento de diseño sigue siendo válido y útil — pero marcadas como
completadas para que una sesión futura no las re-planifique desde cero.

- **Sección 1 (soporte PBIP/TMDL): ✅ COMPLETADO.** `pbi_extractor/pbip_extractor.py`. Validado
  contra un export `.pbip` real (no solo el fixture sintético) — ver
  `docs/pbip_validation_report.md` para los 5 bugs encontrados y corregidos en el proceso.
- **Sección 2 (salida indexada + TOON): ✅ COMPLETADO.** `pbi_extractor/indexed_output.py`,
  `pbi_extractor/toon_encoder.py`. Medido empíricamente, no solo implementado — ver
  `docs/token_optimization_report.md` (TOON no es un ahorro uniforme, gana solo en tablas
  grandes/uniformes; el ahorro dominante viene de `index.json` + carga selectiva, no del formato).
- **Trabajo adicional no anticipado en este documento original**, construido sobre lo anterior:
  `pbi_extractor/resolver.py` (capa de consulta estructurada, JSON/TOON transparente), servidor
  MCP de solo lectura hecho a mano (`pbi_extractor/mcp_server.py`, sin dependencia del SDK `mcp`
  oficial para preservar cero-dependencias), Skills invocables desde Claude Code y GitHub Copilot
  (`.claude/skills/analyze-pbi-model/`, `.github/prompts/analyze-pbi-model.prompt.md`), y un
  experimento de precisión automatizado (`docs/precision_validation_report.md`) que valida que el
  contexto acotado no sacrifica exactitud de respuesta.
- **Documento de planeación estratégica activo:**
  `docs/Analisis_Posicionamiento_Comparativa_Plan_Futuro.md` — reemplaza al checklist original de
  esta sesión de Plan Mode como fuente de verdad del roadmap (secciones 3-4 de ese documento).
  Secciones 3 y 4 de este archivo (Graphify, Grafo diferido) siguen vigentes sin cambios.

### Actualización 2026-07-16 — Horizonte 1 cerrado, Resolver extendido, validación humana preparada

Todos los ítems de Horizonte 1 que quedaban pendientes se completaron esta sesión (ver
`docs/Analisis_Posicionamiento_Comparativa_Plan_Futuro.md` y `CHANGELOG.md` para el detalle
completo), 187 tests en verde:

- **`--index-format auto`**: selección de TOON/JSON por tabla (no más un flag global), umbral
  real confirmado en 7 filas (columnas+measures), no el "~10" aproximado que tenía el roadmap.
- **`index.json` documentado como spec abierta** en `README.md`.
- **Resolver extendido con dependencias/impact-analysis**: `get_measure_dependencies()` /
  `find_measure_usages()`, con soporte transitivo (BFS multi-salto, cycle-safe) además del modo de
  un salto original — expuesto en MCP y `--query` (`--dependencies`, `--usages`, `--transitive`).
  Sigue sin ser un grafo persistido; es resolución bajo demanda sobre `formatted_expression` vía
  regex, consistente con el alcance que la sección 4 de este archivo ya permite.
- **`files_test/Sales Sample.pbip`**: sample oficial de Microsoft (MIT) agregado como segundo
  modelo de prueba (11 tablas/29 measures/5 relaciones) — reveló y permitió corregir un bug real
  del parser TMDL (expresiones DAX delimitadas por backtick-fence se leían como el literal
  `` ``` ``, ver `CHANGELOG.md`).
- **`docs/human_validation_protocol.md`**: protocolo ejecutable para el experimento de la sección
  6 del análisis de posicionamiento (el "próximo paso crítico"). Está en curso — logística de
  reclutamiento de participantes, fuera del control de este repo.
- **`scripts/count_tokens.py`**: conteo real de tokens vía API de Anthropic, dev-only (mismo
  tratamiento que Graphify, sección 3). No corrido en este entorno por falta de
  `ANTHROPIC_API_KEY` — documentado como bloqueado en `docs/token_optimization_report.md`, no
  simulado.

**Decisión reafirmada, no nueva — capa de escritura/modificación de PBIP sigue diferida
(Horizonte 4 sin cambios).** Se evaluó explícitamente en esta sesión si convenía empezarla ahora
y la respuesta fue no, por las mismas razones que ya constaban en la sección 4: escribir TMDL de
vuelta con seguridad (preservar formato/comentarios/lineage tags, resolver merges/conflictos) es
sustancialmente más riesgoso que leer, es redundante con el Modeling MCP oficial de Microsoft, y
distrae del posicionamiento de "compilador de contexto" antes de tener resultados de la
validación humana en curso. Si se reconsidera, que sea por un caso de uso puntual y acotado

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Osc2405/pbi-context](https://github.com/Osc2405/pbi-context) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
