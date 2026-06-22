---
trigger: always_on
description: Drop-in custom instructions for GitHub Copilot. Sibling to `rules/CLAUDE.md` (Claude Code), `rules/CURSOR.md` (Cursor), `rules/CLINE.md` (Cline). Copilot reads `.github/copilot-instructions.md` for repo-wide rules and a per-workspace custom-instructions field for IDE-side rules — both apply on every completion request, so this file is intentionally tight.
---

# GitHub Copilot Rules — Perception.cx Scripting

Drop-in custom instructions for GitHub Copilot. Sibling to `rules/CLAUDE.md` (Claude Code), `rules/CURSOR.md` (Cursor), `rules/CLINE.md` (Cline). Copilot reads `.github/copilot-instructions.md` for repo-wide rules and a per-workspace custom-instructions field for IDE-side rules — both apply on every completion request, so this file is intentionally tight.

## How to Use

Either:

1. Save as `.github/copilot-instructions.md` at the repo root — Copilot picks it up automatically for any developer working in the repo, no per-workspace config needed.
2. Paste into your IDE's Copilot custom-instructions field (VS Code: `Settings → GitHub Copilot → Chat: Custom Instructions`) — per-workspace, applies only to your editor.

Most projects want option 1; option 2 is for adding personal preferences on top of the shared repo rules.

---

## Project Context

- **Languages.** Enma (`.em`), AngelScript (`.as`), Lua (`.lua`), C++ (host SDK).
- **Platform.** Perception.cx scripting runtime.
- **API docs.** Live under `docs/`:
  - `docs/enma/` — Enma language + standard library + SDK
  - `docs/perception/` — Enma APIs
  - `docs/perception/angelscript/` — AngelScript APIs
  - `docs/perception/lua/` — Lua APIs
- **Read before writing API calls.** Copilot does not have these APIs in pretraining; it will hallucinate confident-looking names. Always grep / link the relevant doc file when an API is needed.

## Coding Standards

- Addresses are `uint64`. No `int64`, no `int` for memory addresses.
- Float literals get the `f` suffix for `float32` (`1.5f`); bare `1.5` is `float64`.
- After `ref_process` check `.alive()`; after any `ru64` / `find_code_pattern` / `get_module_base` check for `0`.
- Update routines do reads + logic; render routines do drawing only. Never mix.
- Sigs (`find_code_pattern`) over hardcoded offsets; resolve RIP-relative displacements correctly.
- One feature per file (`esp.em`, `aim.em`, `radar.em`); shared state in `globals.em`.
- Construct `color`, `vec2`, `vec3` per frame.
- Mark unverified offsets with `// UNVERIFIED` and cite `// E-NNN` evidence entry.

## The 12 Guidelines (one-line form)

Long form: `.claude/skills/game-cheat-guidelines/SKILL.md`.

1. Ground every offset — cite sig + source.
2. `uint64` for all addresses.
3. Null-check every pointer chain link.
4. Update routine reads; render routine draws.
5. Sigs over hardcoded offsets.
6. One feature, one file.
7. Construct render primitives per frame.
8. `f` suffix on float32 literals.
9. Minimize memory writes.
10. World-to-screen once, correctly (check `w > 0.001`).
11. GUI for every tunable, no magic constants.
12. Verify against the live binary.

## File Structure

Recommended layout (`templates/full-project/`):

```
project/
├── globals.em       # process handle, base address, entity cache, config state
├── offsets.em       # all sigs + resolved addresses (with E-NNN cross-refs)
├── esp.em           # one feature per file
├── aim.em
├── menu.em          # GUI sidebar — every tunable bound here
├── main.em          # entry: attach, resolve, register routines
└── evidence/        # per-binary evidence log
```

## What Copilot Is Good At Here

- Inline completions: routine boilerplate, GUI sections with N widgets, common loop shapes.
- Doc-comment generation: turn a one-line description into a documented function header.
- Pattern-following: once you've written one feature module the right way, Copilot mimics its shape well in the next module.
- Test scaffolding for the host C++ SDK side.
- Rote refactors confined to a single file (renames within scope, extracting a helper).

## What Copilot Is Bad At Here

- **Multi-file refactors.** Use Cursor / Cline / Aider for cross-file changes; Copilot's window is too narrow.
- **API names from less-common APIs.** It will hallucinate `proc.read_mat3x4_fl32` because the more common APIs follow that shape. Verify every API name against `docs/`.
- **Enforcing the 12 guidelines unprompted.** It does not scan its own completions for `int64`-on-an-address or missing `f` suffixes. You must.
- **Choosing between Enma / AngelScript / Lua.** See `knowledge/pcx-cross-language-bridge.md` for the decision; Copilot will produce whichever language the file extension suggests, which is sometimes the wrong choice.
- **MCP-aware work.** Copilot does not speak MCP. Binary-level RE (find_pattern, disassemble, struct_dump) happens in another tool; Copilot handles only the script-side editing.

## Workflow Notes

- **Inline vs Chat.** Use Copilot Chat (sidebar) for explanation and review; use inline completions for typing-out. Different surfaces, different strengths.
- **Steer with comments.** When Copilot hallucinates an API, write `// from: docs/perception/render-api.md` as a comment above the cursor — Copilot reads it as context for the next completion.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [VoidChecksum/pcx-ai-toolkit](https://github.com/VoidChecksum/pcx-ai-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-22 -->
