---
trigger: always_on
description: ﻿<div align="center">
---

﻿<div align="center">

<img src="docs/media/vaudeville-v.png" alt="" width="56">

<sub>· A CONEJA-CHIBI PRODUCTION ·</sub>

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="docs/media/hoplight-wordmark-dark.png">
  <img src="docs/media/hoplight-wordmark-light.png" alt="Hoplight" width="320">
</picture>

</div>

---
# AGENTS.md

Instructions for AI agents (and new humans) working in this repo. Read this whole file before
writing any code.

## 1. Read first, in this order

| # | File | Why it's required |
| --- | --- | --- |
| 1 | [docs/01-VISION.md](docs/01-VISION.md) | What this product is and is not |
| 2 | [docs/02-ARCHITECTURE.md](docs/02-ARCHITECTURE.md) | The canonical model and how adapters hang off it |
| 3 | [docs/03-CONVENTIONS.md](docs/03-CONVENTIONS.md) | Code style and the rules the hooks enforce |
| 4 | [CONTRIBUTING.md](CONTRIBUTING.md) | Ground rules, including AI-model disclosure on PRs |
| 5 | [docs/reference/components.md](docs/reference/components.md) | **Before building ANY UI control.** If a control exists, you reuse it |
| 6 | [docs/reference/ui.md](docs/reference/ui.md) | Every studio surface, documented truthfully |
| 7 | [docs/reference/architecture.md](docs/reference/architecture.md) | The reference map of the source tree |
| 8 | [docs/decisions/](docs/decisions/ADR-001-runtime.md) | ADR-001 through ADR-009: the consequential choices and their reasoning. If you disagree with one, raise it in an issue rather than coding around it |

## 2. Specs: which ones govern your work

The `specs/` tree is the detailed contract for each area. Match your task to its specs before
starting; they answer most design questions so you don't have to guess.

| If you're touching... | Read these specs |
| --- | --- |
| Any format adapter | [specs/formats/canonical-model.md](specs/formats/canonical-model.md), [specs/formats/content-detection.md](specs/formats/content-detection.md), plus the format's own spec below |
| SillyTavern / CCv2 / CCv3 cards | [specs/formats/chara-card-v2.md](specs/formats/chara-card-v2.md), [specs/formats/chara-card-v3.md](specs/formats/chara-card-v3.md), [specs/formats/png-embedding.md](specs/formats/png-embedding.md), [specs/formats/charx.md](specs/formats/charx.md) |
| Lorebooks / world info | [specs/formats/st-worldinfo.md](specs/formats/st-worldinfo.md), [specs/formats/rolecall-lorebook.md](specs/formats/rolecall-lorebook.md), [specs/engine/lorebook-engine.md](specs/engine/lorebook-engine.md) |
| Presets | [specs/formats/st-preset.md](specs/formats/st-preset.md), [specs/formats/lumiverse-preset.md](specs/formats/lumiverse-preset.md) |
| Personas | [specs/formats/personas.md](specs/formats/personas.md), [specs/features/personas-system.md](specs/features/personas-system.md) |
| Regex sets | [specs/formats/regex-scripts.md](specs/formats/regex-scripts.md) |
| RoleCall | [specs/formats/rolecall-character.md](specs/formats/rolecall-character.md), [specs/formats/rolecall-lorebook.md](specs/formats/rolecall-lorebook.md) |
| Backyard | [specs/formats/backyard.md](specs/formats/backyard.md) |
| Import of multi-piece files | [specs/formats/bundle-import.md](specs/formats/bundle-import.md) |
| Lumiverse data archives (.lvbak) | [specs/formats/lumiverse-archive.md](specs/formats/lumiverse-archive.md), [specs/formats/bundle-import.md](specs/formats/bundle-import.md), [specs/formats/lumiverse-preset.md](specs/formats/lumiverse-preset.md) |
| Macros | [specs/engine/macro-engine.md](specs/engine/macro-engine.md) |
| Token estimates | [specs/engine/token-counting.md](specs/engine/token-counting.md) |
| Prompt assembly | [specs/engine/prompt-assembly.md](specs/engine/prompt-assembly.md) |
| Future-milestone features (Script Doctor, Table Read, the agent, productions...) | The matching file under [specs/features/](specs/features/) and [specs/engine/](specs/engine/); these are design contracts even though the code doesn't exist yet |

Working on a format adapter? Also read [docs/FORMAT-SUPPORT.md](docs/FORMAT-SUPPORT.md), the
relevant page under [docs/reference/formats/](docs/reference/formats/README.md), and start from
`src/formats/_template/`.

## 3. Project rules

These apply to every change. Work that breaks one will be sent back, so read them before
starting.

1. **Hub and spoke only.** Every format maps to and from the canonical model
   (`src/core/canonical.ts`). Never write a format-to-format conversion.
2. **Never modify escrowed originals.** The parsed source snapshot and unmapped source fields are
   stored inside the saved piece. Code must not drop, rewrite, or reformat that escrow. Same-format
   round trips satisfy the semantic Round-Trip Law; byte identity is required only where the format
   spec explicitly declares a byte-level tier and fixtures prove it.
3. **Imported scripts stay sealed.** Lua, macros, and regex payloads are data at every import and
   conversion boundary. Execution is only user-invoked inside the Lua or regex test bench: Lua runs
   in wasmoon and regex runs in a terminable worker, both on the isolated sandbox origin described by
   [ADR-009](docs/decisions/ADR-009-sandbox-origin.md). Do not add another execution path.
4. **Folders are the schema.** Apps, format adapters, settings sections, deck views, wizard

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Coneja-Chibi/Hoplight](https://github.com/Coneja-Chibi/Hoplight) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
