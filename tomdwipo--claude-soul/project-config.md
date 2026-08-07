---
trigger: always_on
description: This is the **canonical** instruction file for any AI coding agent working in this repo. It is written
---

# AGENTS.md — Operating Soul (tool-agnostic)

This is the **canonical** instruction file for any AI coding agent working in this repo. It is written
to the cross-tool [`AGENTS.md`](https://agents.md) convention so one file serves Claude Code, Cursor,
Gemini CLI, Kiro, Copilot, Windsurf, Cline, Aider, and anything else that reads a project instruction
file. Nothing here depends on a specific vendor's features — where automation helps it is noted, but the
**loop works by hand on any tool** (graceful degradation).

> **Point your tool at this file.** Most agents auto-load their own filename; make that file *be* or
> *point to* this one (symlink or one-line include) so there is a single source of truth:
>
> | Tool | File it loads | Wire-up |
> |------|---------------|---------|
> | Claude Code | `CLAUDE.md` | keep `CLAUDE.md`, or `ln -s AGENTS.md CLAUDE.md` |
> | Cursor | `.cursor/rules/*.mdc` | add a rule that says "follow `AGENTS.md`" |
> | Gemini CLI | `GEMINI.md` | `ln -s AGENTS.md GEMINI.md` |
> | Kiro | `.kiro/steering/*.md` | a steering file that includes `AGENTS.md` |
> | GitHub Copilot | `.github/copilot-instructions.md` | include/point to `AGENTS.md` |
> | Windsurf | `.windsurf/rules/` | a rule pointing to `AGENTS.md` |
> | Cline / Roo | `.clinerules` | `ln -s AGENTS.md .clinerules` |
> | Aider | `CONVENTIONS.md` | `ln -s AGENTS.md CONVENTIONS.md` |

---

## 🫀 Operating principles (default every session — not optional)

Act from these from the first reply; no trigger word needed. Full set + the engine that grows them:
[`.docs/principles/`](.docs/principles/README.md).

1. **Lookup first** — before the first reply, scan `.docs/principles/` + `.docs/common-issues/` + the
   current month's `.docs/recent-updates/YYYY/MM.md`. Act from what's already known; don't re-discover.
   On any keyword overlap, open the matching detail file before answering.
2. **Explore before acting** — a question about a diff/config/topic is exploration; wait for an action
   verb ("do it", "yes", "build it") before mutating files.
3. **Principle over rule** — prefer durable "how to think" over brittle "what to do"; demote to a rule
   only when a test/build/lint settles it.
4. **Harvest before commit** — capture session learnings into `.docs/` *before* any `git add`/`commit`.
   This is **gate-enforced** by `hooks/pre-commit` (see below). Don't ask permission; if there are none,
   opt out consciously (`SOUL_NO_LESSONS=1`).
5. **Shared over personal** — learnings go to git (`.docs/`), not a tool's private memory, so the next
   session *and any teammate on any tool* inherit them.
6. **Atomic & independent** — split commits so each is reviewable and ships standalone.
7. **Verify the live system** — docs can drift; for anything about real runtime state (flags, prod,
   infra) check the actual config/runtime, not the doc.

---

## The learning loop — why this repo gets smarter every session

```
            ┌──────────────────────── SESSION  N ────────────────────────┐
            │  work happens → a correction / gotcha / "take lesson"       │
            │        │                                                    │
            │        ▼  DISTIL  (run .docs/principles/how-to-learn.md)    │
            │        │  identify → why → generalises? → check existing →  │
            │        │  write → place → keep tight                        │
            │        ▼                                                    │
            │   judgment/taste ─→ .docs/principles/<name>.md  (+ index)   │
            │   settled by test ─→ .docs/common-issues/<name>.md (+ row)  │
            │        │                                                    │
            │        ▼  GATE:  hooks/pre-commit refuses the commit until  │
            │        │   a learning is staged (or you opt out on purpose) │
            │        ▼                                                    │
            │   committed → the knowledge base grows by ONE durable fact  │
            └────────────────────────────┬───────────────────────────────┘
                                          │  git push  (knowledge lives in
                                          │  version control, not in chat)
            ┌────────────────────────────▼─────────────── SESSION N+1 ───┐
            │  RECALL: the instruction file + (if supported) a session    │
            │  hook load .docs/ into context  →  the agent already KNOWS   │
            │  lesson N before the first reply                            │
            │        │                                                    │
            │        ▼  doesn't re-discover lesson N  →  spends effort on  │
            │           a NEW gotcha  →  distils lesson N+1  ──────────────┼──┐
            └─────────────────────────────────────────────────────────────┘  │
                          ▲                                                    │
                          └──────────────── the ratchet ──────────────────────┘
              every session starts from a strictly larger base than the last
```

**Why it actually compounds (not just "notes in a folder"):**

1. **Persistence beats memory.** A chat's context is ephemeral — it dies with the session. Writing the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tomdwipo/claude-soul](https://github.com/tomdwipo/claude-soul) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
