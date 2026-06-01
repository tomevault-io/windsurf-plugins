---
trigger: always_on
description: This file is auto-loaded by **seek** at startup (and by other agent tools that follow the `AGENTS.md` convention). The canonical agent instructions for this repo live here. [`CLAUDE.md`](CLAUDE.md) mirrors structural content for Claude Code; the two files are related but allowed to diverge where each agent's tooling differs (see [AGENTS.md vs CLAUDE.md](#agentsmd-vs-claudemd--related-but-not-identical)).
---

# seek — project guide for AI assistants

This file is auto-loaded by **seek** at startup (and by other agent tools that follow the `AGENTS.md` convention). The canonical agent instructions for this repo live here. [`CLAUDE.md`](CLAUDE.md) mirrors structural content for Claude Code; the two files are related but allowed to diverge where each agent's tooling differs (see [AGENTS.md vs CLAUDE.md](#agentsmd-vs-claudemd--related-but-not-identical)).

Treat the instructions below as mandatory project conventions.

## Pitfall recording (load-bearing — read this)

When you fix a non-obvious bug, discover an undocumented API constraint, or find a surprising piece of behaviour:

1. **Append an entry to [`docs/pitfalls.md`](docs/pitfalls.md)** using the template at the top of that file. Categorise it (TUI / DeepSeek API / Go / Tooling / etc.); keep it terse — symptom, root cause, fix, lesson, refs.
2. **Add a `Pitfall: <one-line summary>` trailer** to the commit message. Multiple pitfalls in one commit = multiple trailers.
3. The trailer is what `scripts/extract-pitfalls.sh` reads — it's the redundant signal so `docs/pitfalls.md` can be regenerated if it ever falls out of sync.

What counts as "non-obvious":
- A behaviour that surprised you (or the user) for more than a minute.
- An API contract that isn't documented in the obvious spot.
- A workaround for a tooling / classifier / sandbox limitation.
- A design constraint that, if forgotten, would be re-discovered painfully.

What does **not** count:
- Routine typos, missing imports, version bumps, refactors.
- Anything obvious from reading the code or running `go vet`.

If you're not sure: log it. Cheap to add, expensive to recover from memory months later.

## Architecture (non-negotiable)

- **DeepSeek-first, two-tier providers**. `pkg/deepseek` is a first-class client with DeepSeek-specific fields (cache metadata, FIM endpoint, reasoner content). `pkg/llm` is a thin generic interface for second-tier providers (Anthropic / OpenAI / Gemini — landing in M6).
- **`pkg/deepseek` must not import `pkg/llm`** (CI lint enforces this — see `.github/workflows/ci.yml`). The whole point of the split is that DeepSeek-specific optimisations don't get lowered into a generic interface.
- **Skill subsystem (v2)**: read-only loader in `internal/skill` (Anthropic Agent Skills layout: `<dir>/SKILL.md` + frontmatter); install/uninstall/update in `internal/skillmgr`; call-stats JSONL in `internal/skillstats`; shared CLI/TUI dispatcher in `internal/skillcli`. Loader is the only path that runs at startup; everything else is on-demand. **Never** add filesystem writes under `~/.seek/skills/` outside `internal/skillmgr` and `internal/skillstats` — those are the only two packages allowed to mutate user-level skill state.
- **Plan-mode subsystem (v2 + v2.x)**: confirmation-gated workflow `analyze → propose → execute → adjust → report`. Driven by the `propose` tool (`internal/tools/propose/`) and progress-tracked by the `plan` tool (`internal/tools/plan/`). Substate state machine lives in `permission.Policy` (Mode / preApproved flag) + agent mode reminder + TUI status bar; transcript event-sourcing reconstructs state on `seek -resume` (see `plan/reconstruct.go`). Plan-approval artifacts (write-once markdown snapshots) land in `~/.seek/projects/<id>/plans/` via `plan/artifact.go`. Full design + status table in [`docs/prd/feature-plan-mode.md`](docs/prd/feature-plan-mode.md).
- See [`docs/prd/`](docs/prd/) for the full PRD series: v0 initial, v1 Memory, v2 Skill lifecycle, plus standalone feature PRDs (`feature-plan-mode.md`, `feature-webfetch.md`, `feature-permission-refactor.md`, `feature-active-memory.md`, `feature-mcp-client.md`, …).

## Tool usage workflow (load-bearing)

When exploring code, follow this order — skipping steps costs tokens and breaks prefix cache:

1. **grep first** — find exact file + line before reading anything.
2. **read(offset=N) second** — read only the relevant window, not the whole file.
3. `read` accepts an optional `limit` parameter (default 50, max 50 — values above 50 error). Use `offset=N` to page through larger files.
4. `grep` caps at 20 matches by default (`max_matches` can be raised, but rarely should be).

Never read a whole file to answer a question you could answer with grep. The prefix cache survives only when old messages are byte-identical; lazy whole-file reads balloon prompt tokens and degrade cache hit rate.

5. **read before edit** — before calling `edit`, first `read(offset=N, path=...)` on the target lines to capture the **exact whitespace** of the `old_string`. Do not guess tab depth from memory; the read output preserves it byte-for-byte. A single read call costs less than the error-fix loop from a mismatched `old_string`.

## Tool descriptions: the highest-leverage behavioural lever


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [whyiyhw/seek](https://github.com/whyiyhw/seek) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
