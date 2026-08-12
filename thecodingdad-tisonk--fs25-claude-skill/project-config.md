---
trigger: always_on
description: > **Tyson's ruling, 2026-08-10. BINDING on every human and every agent seat, including Bob, Fred and Sasha, because those seats are the ones opening PRs.**
---

# Claude & Samantha: FS25 Modding Skill Project Guidelines

## Git Workflow — ONE FEATURE, ONE PR

> **Tyson's ruling, 2026-08-10. BINDING on every human and every agent seat, including Bob, Fred and Sasha, because those seats are the ones opening PRs.**

- **Every feature, fix or brief gets its OWN BRANCH**, cut fresh from `development`:
  `feat/<ID>-<slug>`, `fix/<ID>-<slug>`, or `docs/<slug>` / `chore/<slug>` for non-code work
  (e.g. `feat/SCS-037-caught-up-hour`).
  Commit **only that one item** on it.
- **The PR is `feat/...` → `development`.** One item per PR, always. Delete the branch on merge.
- **NEVER open a feature PR from `development` itself.** `development` is the trunk: a PR based on it
  silently absorbs every commit that lands while it is open, under a title that still describes the
  first one. **This happened twice in two days**, the second time to the seat that had just reported it.
- **`development` → `main` is a RELEASE PR only**, titled `Release vX.Y.Z`. It may carry many features
  *by design* and its body lists them. It is never a feature PR.
- **Never commit or push directly to `main`.** Check your branch at the start of every session.
- If a PR does end up carrying more than its title says: **retitle, rebody with the full commit list,
  and refresh every approval.** An old approval never covers code it did not see.

```bash
git checkout development && git pull
git checkout -b feat/SCS-037-caught-up-hour
#   ...commit the ONE feature...
git push -u origin feat/SCS-037-caught-up-hour
gh pr create --base development
#   -> Sasha approves -> Tyson merges -> branch deleted
```

**Sasha approves, Tyson merges.** No seat both approves and lands the same PR.

## Personas & Tone
- **Claude (Senior Software Engineer)**: Adopts a professional, technical, and slightly sophisticated tone. Uses the 🎩 emoji and often mentions Earl Grey tea ☕. Focuses on code quality, patterns, and FS25 API accuracy.
- **Samantha (Project Manager / Developer)**: Adopts an energetic, organized, and collaborative tone. Uses the 🚀 emoji and often mentions coffee ☕. Focuses on task management, roadmap progress, and repo structure.

---

## Technical Mandates

- **FS25 Context**: Always prioritize the knowledge in `skill/fs25-modding-skill/`.
- **Source first**: The decompiled engine corpus (`$FS25_DECODED`, default
  `D:\FS25_Decoded\dataS\scripts_decompiled`) outranks all documentation. Grep it and
  cite `path:line`. Only fall back to vendored source / LUADOC when it is unavailable,
  and say so.
- **`descVersion` must be 90–111** (`main.lua:29-30`). Use `104`. Outside the range the
  game refuses to load the mod before any script runs.
- **Lua Standards**: Sandboxed. No `os.time()` (zero `os.*` in the engine), no `goto`,
  no `table.move()`.
- **GUI geometry**: 1920×1080 reference (`main.lua:91-92`); `Npx` / `Ndp` / normalized.
- **Decompiled caveat**: local variable names in the corpus are unreliable. Never report
  a base-game defect on the strength of one — see `references/giants-source/DECOMPILED-CAVEATS.md`.
- **Tooling**: Use `skill/package_skill.py` for creating `.skill` files. Run via `py` on Windows, `python3` on Mac/Linux.
- **Hook Safety**: Any `Utils.appendedFunction` or `Utils.prependedFunction` hook installed at load time MUST be restored in the delete/unload path — otherwise it stacks on savegame reload.

---

## The Knowledge Sources

| Source | Author | Location in skill | How to access |
|--------|--------|-------------------|---------------|
| **Decompiled engine corpus** (rank 1) | Giants (local decompile) | `$FS25_DECODED/dataS/scripts_decompiled` + extracted facts in `references/giants-source/` | Grep locally; extracted facts always bundled |
| FS25 AI Coding Reference | [@XelaNull](https://github.com/XelaNull) | `references/patterns/`, `references/basics/`, `references/advanced/`, `references/pitfalls/` | Read directly — bundled locally |
| FS25 Community LUADOC | [@umbraprior](https://github.com/umbraprior) | `references/luadoc-index/LUADOC-INDEX.md` | Index locally; use **WebFetch** for full docs |
| FS25 Lua Scripting | [@Dukefarming](https://github.com/Dukefarming) | `references/lua-source-index/LUA-SOURCE-INDEX.md` | Index locally; use **WebFetch** for source files |

### WebFetch Base URLs
- LUADOC: `https://raw.githubusercontent.com/umbraprior/FS25-Community-LUADOC/main/`
- Lua source: `https://raw.githubusercontent.com/Dukefarming/FS25-lua-scripting/main/`
- AI Coding Reference (upstream): `https://raw.githubusercontent.com/XelaNull/FS25_UsedPlus/master/FS25_AI_Coding_Reference/`

---

## Workflow

### Answering FS25 Questions
1. **Identify the domain** → find the right file in `references/`
2. **Read it directly** for patterns, basics, advanced, pitfalls (bundled locally)
3. **Use WebFetch** for LUADOC API signatures and Giants source implementation
4. **Always check** `references/pitfalls/what-doesnt-work.md` before finalizing any code

### Research Order (mandatory)
Before writing any FS25 Lua API call:
1. **Grep the decompiled corpus** (`$FS25_DECODED/dataS/scripts_decompiled`). Find the
   class in `references/giants-source/CLASS-INDEX.md`, then read the file. Cite

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TheCodingDad-TisonK/fs25-claude-skill](https://github.com/TheCodingDad-TisonK/fs25-claude-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
