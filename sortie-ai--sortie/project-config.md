---
trigger: always_on
description: **Don't assume. Don't hide confusion. Surface tradeoffs.**
---

# Project Conventions for AI Agents

## Working Agreement

### 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask. This applies even when the confusion seems mild ("a bit confusing") or when you can imagine a reasonable resolution. Ambiguity that the agent silently resolves is a class of bug; ambiguity that the user resolves is not.

### 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

### 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

When the working tree has changes you didn't make:
- You are not the only one working in this repo. The user or a parallel agent session may edit files while you work, so `git status` and `git diff` show their uncommitted changes mixed with yours.
- Changes you cannot trace to your own task are not yours to revert. Don't assume unfamiliar edits are accidental or stray - they may be deliberate work from another session.
- Never discard, revert, reset, stash, or reformat files outside your task's scope (`git checkout --`, `git restore`, `git reset --hard`, `git stash`, `git clean`). Stage your own paths by name; never `git add -A` or `git add .`.
- If changes you didn't make seem to collide with your task, stop and ask. Never resolve it by throwing them away.

The test: Every changed line should trace directly to the user's request.

### 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

## Guidelines

### Commands

Read the project Makefile to discover available targets before running any Go toolchain commands directly.

### Gotchas

- Go is managed by asdf. The `go` binary resolves through `~/.asdf/shims/go`.
- **Architecture doc is the spec.** `docs/architecture.md` is a curated index; the spec lives in one file per section under `docs/architecture/`. Open the index, find the section your task touches, and read that section file before implementing anything. Drift from the spec is a bug. Section files are authoritative when they disagree with the index.
- **Symphony is prior art, not a template.** Sortie derives from OpenAI Symphony but diverges intentionally (Go instead of Elixir, SQLite persistence, adapter interfaces). Do NOT copy Symphony patterns or Elixir idioms.
- **Workspace safety invariants are security boundaries.** Path containment under workspace root, sanitized workspace keys (`[A-Za-z0-9._-]` only), and cwd validation before agent launch are mandatory — not suggestions. See [architecture Section 9.6](docs/architecture/09-workspace-management-and-safety.md#96-safety-invariants).
- **Generic naming in core code.** Use `agent_*`, `tracker_*`, `session_*` in orchestrator core. Never `jira_*`, `claude_*`, `codex_*` outside their adapter packages.
- **Integration tests are env-gated.** Each adapter has its own `SORTIE_<ADAPTER>_TEST=1` gate (e.g. `SORTIE_JIRA_TEST`, `SORTIE_LINEAR_TEST`, `SORTIE_CODEX_TEST`). GitHub end-to-end orchestrator tests also gate on `SORTIE_GITHUB_E2E=1` and require `SORTIE_GITHUB_TOKEN` and `SORTIE_GITHUB_PROJECT`. Without the gate set, integration tests must skip cleanly — never fail.
- **SQLite library is `modernc.org/sqlite` only.** Never `mattn/go-sqlite3` — CGo breaks the single-binary zero-dependency deployment model.
- **`internal/scm/` is an adapter family.** Apply the same boundary rules as `internal/tracker/*/`: no cross-adapter imports, no orchestrator imports, normalize external responses to domain types at the boundary. The coder agent's layer constraints enumerate trackers and agents but omit SCM — treat that gap as a drafting bug, not permission.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sortie-ai/sortie](https://github.com/sortie-ai/sortie) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
