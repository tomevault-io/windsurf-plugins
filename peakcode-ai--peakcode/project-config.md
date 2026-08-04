---
trigger: always_on
description: - Do not run `bun fmt`, `bun lint`, or `bun typecheck` unless the user explicitly asks for them in the current conversation.
---

# AGENTS.md

## Task Completion Requirements

- Do not run `bun fmt`, `bun lint`, or `bun typecheck` unless the user explicitly asks for them in the current conversation.
- All of `bun fmt`, `bun lint`, and `bun typecheck` must pass before considering tasks completed.
- Treat `bun fmt`, `bun lint`, and `bun typecheck` as heavyweight workspace checks: bundle them into one final verification pass per task whenever possible, and avoid rerunning the full set repeatedly during iteration.
- If a user asks for a small follow-up right after a recent full verification pass, prefer no rerun or the smallest reasonable re-check unless the user explicitly asks for full validation again.
- If the user asks to focus on code only, do not run `bun fmt`, `bun lint`, or `bun typecheck` automatically. In that mode, make the code changes first and only run verification if the user explicitly asks for it.
- NEVER run `bun test`. Always use `bun run test` (runs Vitest).

## PR Review

Before reviewing or merging any pull request, run the project's PR review script first:

```bash
bash scripts/pr-review.sh [PR_NUMBER]      # review a specific PR
bash scripts/pr-review.sh                  # auto-detect PR from current branch
bash scripts/pr-review.sh --all            # review all open PRs
```

The script checks a PR against `CONTRIBUTING.md` and `AGENTS.md` conventions across 16 categories: description completeness, diff size (<200 lines preferred), mergeability, changed-file hygiene, `bun.lock` drift, debug artifacts, `bun fmt:check`, `bun typecheck`, `bun lint`, affected tests, checklist compliance, scope purity, and AGENTS.md compliance.

Exit codes: `0` = approved (possibly with caveats), `1` = changes requested (errors must be fixed), `2` = rejected (do not merge).

Use the script's output to guide review comments and merge decisions. Do not merge a PR that returns exit code `1` or `2` without resolving the reported issues.

## Project Snapshot

Peak Code is a minimal web GUI for using coding agents like Codex and Claude.

This repository is a VERY EARLY WIP. Proposing sweeping changes that improve long-term maintainability is encouraged.

## Core Priorities

1. Performance first.
2. Reliability first.
3. Keep behavior predictable under load and during failures (session restarts, reconnects, partial streams).

If a tradeoff is required, choose correctness and robustness over short-term convenience.

## Transcript Performance Guardrails

- Treat transcript auto-scroll as a live-output feature, not a generic "working" feature. Buffering, reconnecting, pending approvals, and tool-only activity must not be wired as if assistant text is actively streaming.
- When wiring scroll-follow logic, count real transcript messages only. Tool/work rows must not retrigger the same "new content arrived" auto-stick path.
- Prefer the simpler fork-style transcript path for the common case. Small and medium transcripts should avoid virtualization churn unless there is a clear measured need.
- If virtualization is used, never couple `rowVirtualizer.measure()` directly to another bottom-stick or height-follow cycle. Height-follow for live output should stay one-way to avoid measure/scroll feedback loops.
- Preserve these behaviors with focused transcript tests when changing chat scrolling, timeline measurement, or sidebar-driven transcript updates.

## Maintainability

Long term maintainability is a core priority. If you add new functionality, first check if there is shared logic that can be extracted to a separate module. Duplicate logic across multiple files is a code smell and should be avoided. Don't be afraid to change existing code. Don't take shortcuts by just adding local logic to solve a problem.

## Package Roles

- `apps/server`: Node.js WebSocket server. Wraps Codex app-server (JSON-RPC over stdio), serves the React web app, and manages provider sessions.
- `apps/web`: React/Vite UI. Owns session UX, conversation/event rendering, and client-side state. Connects to the server via WebSocket.
- `packages/contracts`: Shared effect/Schema schemas and TypeScript contracts for provider events, WebSocket protocol, and model/session types. Keep this package schema-only — no runtime logic.
- `packages/shared`: Shared runtime utilities consumed by both server and web. Uses explicit subpath exports (e.g. `@t3tools/shared/git`) — no barrel index.

## Local Dev Instance Isolation

- Never start the default `bun run dev` while another Peak Code instance is running unless the user explicitly wants shared ports/state.
- Use an isolated home dir and non-default ports when running alongside the user's own Peak Code instance, for example: `env -u PEAKCODE_AUTH_TOKEN PEAKCODE_PORT_OFFSET=3158 PEAKCODE_NO_BROWSER=1 bun run dev -- --home-dir ./.peakcode-pr84 --port 58090`.
- Always dry-run first when avoiding conflicts: `env -u PEAKCODE_AUTH_TOKEN PEAKCODE_PORT_OFFSET=3158 bun run dev -- --home-dir ./.peakcode-pr84 --port 58090 --dry-run`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PeakCode-AI/PeakCode](https://github.com/PeakCode-AI/PeakCode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
