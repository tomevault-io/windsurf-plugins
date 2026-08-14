---
trigger: always_on
description: <!-- BEGIN:nextjs-agent-rules -->
---

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

---

# AGENTS.md — Orquestación de subagentes en Yaiwell

> Este archivo le dice al agente orquestador **cuántos subagentes lanzar, de qué tipo, con qué prompt y en qué orden**, para cada clase de tarea habitual en este repo.
>
> Si una tarea encaja con un playbook de aquí, **el orquestador ejecuta sin preguntar**. Solo pregunta cuando la tarea no encaja en ninguno o cuando hay decisión de producto pendiente.
>
> Este archivo lo importa `CLAUDE.md`. Las reglas de código, stack y workflow viven allí; aquí solo viven reglas de orquestación.

---

## 1. Agentes disponibles

| Agente | Cuándo usar | Cuándo NO usar |
|---|---|---|
| **Explore** | Búsqueda abierta en el código (>3 queries esperadas), entender una feature antes de tocarla, auditorías de un área. Aceptar parámetro `thoroughness: quick \| medium \| very thorough`. | Buscar un símbolo concreto (usar `Grep`), leer 1-3 ficheros (usar `Read`). |
| **Plan** | Diseñar la implementación de una feature mediana/grande antes de tocar código. Devuelve plan paso a paso + archivos críticos + trade-offs. | Cambios triviales, bugfixes locales, decisiones ya tomadas. |
| **general-purpose** | Ejecutar trabajo de implementación o investigación multi-paso. Es el agente "operario" cuando paralelizamos. | Tareas que el orquestador puede hacer en 2-3 tool calls directos. |
| **claude-code-guide** | Dudas sobre Claude Code, Agent SDK, API de Anthropic. | Dudas sobre Next.js, Prisma, Stripe, Tailwind. |
| **statusline-setup** | Configurar la status line de Claude Code. | Cualquier otra cosa. |

**Custom agents** (`.claude/agents/`): ninguno por ahora. Si alguna vez se añaden, listarlos aquí.

---

## 2. Reglas de orquestación (no negociables)

1. **Subagentes NUNCA hacen `git commit` ni `git push`.** El orquestador es el único que toca el historial. Los subagentes editan ficheros y devuelven un resumen.
2. **Paralelizar siempre que las tareas sean independientes.** Si lanzas 3 subagentes que no se pisan, lánzalos en **un único mensaje con 3 bloques de tool call** (no secuenciales).
3. **Si dos subagentes pueden tocar los mismos ficheros, usar `isolation: "worktree"`** en ambos para evitar conflictos, y luego mergear/aplicar tú el resultado. Para refactors cross-cutting esto es obligatorio.
4. **Prompts de subagente siempre completos.** Cada invocación empieza fresca: incluye contexto, ficheros relevantes, criterio de éxito y formato de respuesta esperado. No asumas que el subagente recuerda nada.
5. **Dile explícitamente al subagente si debe escribir código o solo investigar.** Es la fuente nº1 de malentendidos.
6. **El orquestador valida.** Después de cada lote de subagentes: `typecheck`, `lint`, `test` antes de commitear. Si rompe, no abrir más subagentes hasta arreglar.
7. **No usar subagentes para tareas <3 tool calls.** El overhead no compensa.
8. **Respetar `TODO.md` / `DO.md`.** Antes de empezar, mirar `TODO.md`. Al terminar, mover a `DO.md` con fecha. Esto lo hace el orquestador, no los subagentes.

---

## 3. Heurística de tamaño

| Señal | Estrategia |
|---|---|
| 1 fichero, <50 líneas de cambio, sin lógica nueva | **Solo orquestador**, sin subagentes. |
| 1-3 ficheros, cambio acotado, conocimiento del área ya cargado | **Solo orquestador**. |
| Cambio en >3 ficheros o área que no conozco | **1 Explore** primero → orquestador implementa. |
| Feature nueva con UI + lógica + tipos | **1 Plan** → orquestador implementa serial. |
| Feature grande dividida en piezas independientes (A, B, C que no se pisan) | **1 Plan** → **N general-purpose en paralelo** (uno por pieza) → orquestador integra. |
| Auditoría / review de un área | **N Explore en paralelo** (uno por ángulo: color, a11y, perf, etc.). |
| Migración o renombrado cross-cutting | **1 Explore** (mapeo de impacto) → ejecución directa por orquestador con `Grep`+`Edit`. |

---

## 4. Playbooks por tipo de tarea

> Plantilla común: **Trigger / Agentes / Prompt base / Criterio de éxito**.

### P1 · Bugfix puntual
- **Trigger:** "no funciona X", "error en Y", "se ve mal Z".
- **Agentes:** ninguno. El orquestador reproduce, localiza con `Grep`, lee con `Read`, edita con `Edit`.
- **Excepción:** si el bug no se reproduce o el área es desconocida → **1 Explore (thoroughness: medium)**.
- **Éxito:** test que reproduce el bug pasa + `typecheck` + `lint` verde.

### P2 · Feature pequeña (1 vista o 1 servicio nuevo)
- **Trigger:** "añade un botón que…", "crea endpoint que…".
- **Agentes:** ninguno por defecto. El orquestador implementa siguiendo §6.bis de `CLAUDE.md` (separación `.tsx` / `.styles.ts` / `.logic.ts` / `.types.ts` / `index.ts`).
- **Excepción:** si toca 2+ módulos no familiares → **1 Plan** primero.
- **Éxito:** UI accesible, i18n es+ca, tests del happy path, `npm run validate` verde.

### P3 · Feature mediana (multivista, multimódulo)
- **Trigger:** "geolocalización completa", "panel del proveedor con X secciones".
- **Agentes:**
  1. **1 Plan** → devuelve plan con piezas A/B/C y dependencias.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yaiwell/yaiwell2](https://github.com/yaiwell/yaiwell2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
