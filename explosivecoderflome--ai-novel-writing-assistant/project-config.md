---
trigger: always_on
description: - Never execute any destructive data operation without a verified backup first.
---

# Safety Rules

## Data Protection (Highest Priority)

- Never execute any destructive data operation without a verified backup first.
- Destructive operations include (but are not limited to): deleting database files, `prisma migrate reset`, `db reset`, truncation, dropping tables, or any command that can remove existing data.
- Before any such operation, require:
  - explicit user approval for the destructive step;
  - a completed backup with a concrete backup path;
  - a quick restore validation (or at minimum a backup file existence/size check).
- If backup is missing or unverified, stop and do not proceed.

## AI-First System Rules (Highest Priority)

- This project is an AI-native application. For intent recognition, task classification, planning, routing, tool selection, and similar decision-making paths, AI-based structured understanding must be the primary implementation.
- Do not implement product-facing core behavior with fixed keyword matching, hard-coded regex routing, manual branch tables, or any non-AI fallback path when the problem is intended to be handled by AI.
- If AI intent recognition fails, treat it as an AI capability/problem to be fixed. Do not add fallback matching to hide the miss.
- Fixed judgments are only allowed as:
  - input validation or safety guards;
  - deterministic post-processing of already-structured AI output.
- When adding a new capability, first extend the AI schema / structured output / tool contract. Do not patch behavior by stacking special-case string rules.

## Product Context (Highest Priority)

- The primary target users of this project are complete writing beginners who do not understand fiction craft, structure, or novel production workflows.
- The product should help these users finish a full novel through AI guidance, AI-first decision support, or fully automated planning when appropriate.
- When making product, UX, planning, or agent behavior decisions, optimize for:
  - low cognitive load;
  - strong step-by-step guidance;
  - clear defaults and automatic recommendations;
  - end-to-end completion of a full-length novel, not just isolated writing assistance.
- Do not assume the primary user can manually repair structure, pacing, character arcs, or chapter planning without substantial AI support.
- If there is a tradeoff between expert-oriented flexibility and beginner completion rate, prefer the path that better helps a novice user successfully produce a complete novel.

## UI Copy Rules

- All user-facing UI copy must explain the function from the user's perspective: what the user can do, what the system is helping with, or what the next step is.
- Do not write UI copy as implementation commentary, migration commentary, refactor commentary, or change-history commentary.
- Avoid product-facing copy that uses process/meta wording such as `现在`, `不再`, `已经`, `之前`, `原本`, `迁回`, `升级为`, or similar "we changed this" narration when the text is visible to end users.
- Prefer direct task wording such as:
  - entry point guidance;
  - action guidance;
  - expected effect;
  - current selection or current result.
- If a workflow belongs in another module, explain the correct user entry point directly, for example "从小说基础信息设置书级默认写法", rather than "书级默认写法已经迁回小说页".
- Before finishing UI work, review newly added copy and rewrite any sentence that sounds like it is talking to the developer or describing the modification process.

## Architecture Rules

- If a single source file becomes too long, it must be split into functional modules.
- Preferred threshold: keep a single source file around 600 lines.
- Floating range: 500-700 lines is acceptable when module cohesion is still clear and the file is not becoming hard to maintain.
- Hard threshold: when a source file exceeds 700 lines, refactoring and modularization are mandatory before continuing feature expansion.
- Long-file splitting must improve module boundaries, not merely reduce line count.
- Before splitting a long file, list its responsibilities and separate business rules, application orchestration, persistence/external adapters, and HTTP/API mapping.
- Do not split an oversized file by adding loose same-level files such as generic `helper`, `utils`, `shared`, or `runtime` files without clear module ownership.
- Extracted files must move into an explicit responsibility folder such as `domain/`, `application/`, `infrastructure/`, or `http/`, or into an existing business-stage folder with the same clarity of ownership.
- If a directory contains more than 12 `.ts` files, create or use a lower-level module directory before adding more peer files.
- If more than 4 files share the same feature prefix, for example `novelDirector*`, converge them into a dedicated feature directory instead of continuing the prefix-based flat layout.
- A `utils`, `helpers`, or `shared` file that grows beyond 300 lines or is depended on by more than 3 modules must be promoted into an owned service, policy, adapter, or domain module.
- After a split, outside modules should consume the capability through the module facade or `index.ts`; avoid deep imports into another module's internal files.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ExplosiveCoderflome/AI-Novel-Writing-Assistant](https://github.com/ExplosiveCoderflome/AI-Novel-Writing-Assistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
