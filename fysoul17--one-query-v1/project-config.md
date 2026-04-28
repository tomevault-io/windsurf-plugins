---
trigger: always_on
description: - Proactively use subagents and skills where needed
---

# Project Instructions

## General

- Proactively use subagents and skills where needed
- Follow commit conventions in `.claude/commit-conventions.md`
- Follow design system in `docs/design-system.md` for UI/UX work if exist

## Error Handling Philosophy

**No silent fallbacks.** Handle errors explicitly and show the user what happened.

- **Default behavior**: When something fails, display a clear error state in the UI (error message, retry option, or actionable guidance). Do NOT silently fall back to default/placeholder data.
- **Fallbacks are the exception, not the rule.** Only use fallbacks when it is a widely accepted best practice (e.g., fallback fonts in CSS, CDN failover, graceful image loading with placeholder). If unsure, handle the error explicitly instead.
- **Never hide failures.** The user should always know when something went wrong. A visible error with a retry button is better UX than silently showing stale/default data.
- **Pattern**: `try { doThing() } catch (error) { showErrorUI(error) }` — NOT `try { doThing() } catch { return fallbackValue }`

## Investigation Workflow

When investigating bugs, analyzing features, or exploring code:

1. **Define exit criteria upfront** - Ask "What does 'done' look like?" before starting
2. **Checkpoint progress** - Use TodoWrite every 5-10 minutes to save findings
3. **Output intermediate summaries** - Provide "Current Understanding" snapshots so work isn't lost if interrupted
4. **Always deliver findings** - Never end mid-analysis; at minimum output:
   - Files examined
   - Key findings
   - Remaining unknowns
   - Recommended next steps

For complex investigations, use `/devlyn:team-resolve` to assemble a multi-perspective investigation team, or spawn parallel Task agents to explore different areas simultaneously.

## UI/UX Workflow

The full design-to-implementation pipeline:

1. `/devlyn:design-ui` → Generate 5 style explorations
2. `/devlyn:design-system [N]` → Extract tokens from chosen style
3. `/devlyn:implement-ui` → Team implements or improves UI from design system
4. `/devlyn:team-resolve [feature]` → Add features on top

## Feature Development

1. **Plan first** - Always output a concrete implementation plan with specific file changes before writing code
2. **Track progress** - Use TodoWrite to checkpoint each phase
3. **Test validation** - Write tests alongside implementation; iterate until green
4. **Small commits** - Commit working increments rather than large changesets

For complex features, use the Plan agent to design the approach before implementation.

## Automated Pipeline (Recommended Starting Point)

For hands-free build-evaluate-polish cycles — works for bugs, features, refactors, and chores:

```
/devlyn:auto-resolve [task description]
```

This runs the full pipeline automatically: **Build → Build Gate → Browser Validate → Evaluate → Fix Loop → Simplify → Review → Challenge → Security Review → Clean → Docs**. Each phase runs as a separate subagent with its own context. Communication between phases happens via files (`.devlyn/done-criteria.md`, `.devlyn/BUILD-GATE.md`, `.devlyn/EVAL-FINDINGS.md`, `.devlyn/BROWSER-RESULTS.md`, `.devlyn/CHALLENGE-FINDINGS.md`).

The **Build Gate** (Phase 1.4) runs real compilers, typecheckers, and linters — the same commands CI/Docker/production will run. It auto-detects project types (Next.js, Rust, Go, Solidity, Expo, Swift, etc.) and Dockerfiles. This is the primary defense against "tests pass locally, breaks in CI/Docker" class of bugs (type errors in un-tested files, cross-package drift, Dockerfile copy mismatches).

The **Challenge** phase (Phase 4.5) is a fresh skeptical review with no checklist — a subagent reads the entire diff cold with zero context from prior phases and asks "would I ship this to production with my name on it?" This catches the subtle issues that structured checklist-driven reviews miss: wrong-but-working approaches, unstated assumptions, non-idiomatic patterns, and integration gaps.

For web projects, the Browser Validate phase starts the dev server and tests the implemented feature in a real browser — clicking buttons, filling forms, verifying results. If the feature doesn't work, findings feed back into the fix loop.

Optional flags:
- `--max-rounds 6` — increase max evaluate-fix iterations (default: 4)
- `--skip-browser` — skip browser validation phase (auto-skipped for non-web changes)
- `--skip-build-gate` — skip the deterministic build gate (not recommended)
- `--build-gate strict` — treat warnings as errors; `--build-gate no-docker` — skip Docker builds for speed
- `--skip-review` — skip team-review phase
- `--skip-clean` — skip clean phase
- `--skip-docs` — skip update-docs phase
- `--engine auto|codex|claude` — intelligent model routing. `auto` (default) routes each phase and team role to the optimal model (Claude or Codex GPT-5.4) based on benchmark data. `codex` forces Codex for implementation, Claude for evaluation. `claude` uses Claude for everything. Requires codex-mcp-server for `auto` and `codex` modes.
- `--with-codex [evaluate|review|both]` — (legacy, superseded by `--engine`) use OpenAI Codex as cross-model evaluator/reviewer (requires codex-mcp-server)

## Preflight Check (Post-Roadmap Verification)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fysoul17) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-15 -->
