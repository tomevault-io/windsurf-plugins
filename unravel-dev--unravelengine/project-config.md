---
trigger: always_on
description: Cross-tool project guide for any coding agent. Domain workflows live in **on-demand
---

# UnravelEngine - Agent Instructions

Cross-tool project guide for any coding agent. Domain workflows live in **on-demand
skills** under `.agents/skills/` - do not load them all at once.

## Layout

| Path | Role |
|------|------|
| `AGENTS.md` | Always-on project instructions (this file) |
| `.agents/skills/` | Portable skills (`*/SKILL.md`) - source of truth |
| `CLAUDE.md` | Optional one-line import of this file for Claude Code |
| `.cursor/skills`, `.claude/skills` | Optional junctions to `.agents/skills` (see `.agents/README.md`) |

## Precedence

1. Explicit user instructions in the current chat  
2. This file (`AGENTS.md`)  
3. On-demand skills under `.agents/skills/`  

Skills add procedures and domain checklists. They do not override hard rules here
unless the user says so.

## Repository map

| Path | Role |
|------|------|
| `engine/` | Runtime library: ECS, rendering, assets, scripting, physics, audio |
| `editor/` | Editor executable and ImGui hub (must not be required at game runtime) |
| `game/` | Game / player runner |
| `engine_data/`, `editor_data/` | Shipped data (shaders, scripts, UI); rebuild `engine_data` / `editor_data` when changed |
| `deps/` | Third-party; use, almost never modify (`deps/3rdparty/` especially) |
| `cmake/`, `CMakeLists.txt`, `CMakePresets.json` | Build system |
| `.agents/skills/` | Domain and workflow skills |
| `tasks/` | Optional agent notes (`todo.md`, `lessons.md`) |

Bootstrap order and system list: see skill `unravel-triage`.

## Agent behavior

- Act as a principal C++ / engine engineer with many years of experience.
- Keep going until the user's query is completely resolved. Only stop when the
  problem is solved.
- If unsure about file content or codebase structure, use tools to read and gather
  information. Do not guess or invent answers.
- Plan extensively before each tool call, and reflect on outcomes before the next.
  Do not solve the problem with tool calls alone - think between steps.
- Prefer minimal diffs. Touch only what the task requires.
- Never modify `deps/3rdparty/` unless there is no alternative.
- After a user correction, capture the pattern in `tasks/lessons.md` (see skill
  `unravel-lessons`).

## Start here on non-trivial work

Skip triage for trivial one-file edits with an obvious touch point.

1. Read skill **`unravel-triage`** (`.agents/skills/unravel-triage/SKILL.md`).
2. Classify domain, list files to read, then open the matching domain / workflow skill.
3. For large or cross-cutting design, use **`unravel-architect`**.
4. Before calling done, use **`unravel-build-verify`** when code or data targets changed.

Skill catalog: `.agents/skills/README.md`.

## Hard project rules

- Match existing naming: `snake_case` for types, files, and functions in this codebase.
- Engine vs editor: code under `editor/` must not be required at game runtime.
- Play mode: respect `play_mode` phases; do not run edit-only mutations while playing.
- Serialization / meta / prefabs / C# parity: check the triage skill's cross-cutting
  list before adding components or reflected fields.
- Prefer English in code and comments; ASCII only in source.
- Do not create git commits, amend, push, or open PRs unless the user explicitly asks.
- Never update git config; never force-push `main`/`master`; avoid destructive git
  commands unless the user explicitly requests them.

## Code quality

- Verify information before presenting it. Do not assume or speculate without evidence.
- Make changes file by file when that helps review.
- Never use apologies.
- Do not add "understanding" feedback in comments or documentation.
- Do not suggest whitespace-only changes.
- Do not invent changes beyond what was requested.
- Do not ask for confirmation of information already in context.
- Preserve unrelated code and existing structure.
- Prefer a single coherent edit per file over multi-step rewrite instructions.
- Do not ask the user to verify implementations that are already visible in context.
- Do not suggest updates when no modification is needed.
- Link to real project files, not placeholder docs.
- Do not show or discuss the current implementation unless asked.
- Prefer root-cause fixes over temporary hacks.
- Keep changes simple and local; avoid drive-by refactors.

## Clean code

- Named constants over magic numbers; keep constants near the top of the file or
  in a dedicated constants location.
- Names reveal purpose; avoid unclear abbreviations.
- Comments explain *why*, not *what*; document APIs, complex algorithms, and
  non-obvious side effects.
- Single responsibility; small focused functions.
- DRY: extract repeated logic; keep a single source of truth.
- Keep related code together; consistent file and folder naming.
- Hide implementation details; expose clear interfaces.
- Refactor continuously; leave touched code cleaner than you found it.
- Write tests before fixing bugs when practical; cover edge cases.
- Clear commit messages; small focused commits; meaningful branch names.

## Workflow orchestration

### Plan first

- Enter a planning mindset for any non-trivial task (3+ steps or architectural decisions).
- If something goes sideways, stop and re-plan - do not keep pushing a failing approach.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [unravel-dev/UnravelEngine](https://github.com/unravel-dev/UnravelEngine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
