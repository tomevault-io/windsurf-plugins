---
trigger: always_on
description: Agent instructions for AI assistants working on the Mux codebase
---


**Prime directive:** keep edits minimal and token-efficient—say only what conveys actionable signal.

## Project Snapshot

- `mux`: Electron + React desktop app for parallel agent workflows; UX must be fast, responsive, predictable.
- Minor breaking changes are expected, but critical flows must allow upgrade↔downgrade without friction; skip migrations when breakage is tightly scoped.
- **Before creating or updating any PR, commit, or public issue**, you **MUST** read the `pull-requests` skill (`agent_skill_read`) for attribution footer requirements and workflow conventions. Do not skip this step.

## External Submissions

- **Do not submit updates to the Terminal-Bench leaderboard repo directly.** Only provide the user with commands they can run themselves.

## Repo Reference

- Core files: `src/main.ts`, `src/preload.ts`, `src/App.tsx`, `src/config.ts`.
- Up-to-date model names: see `src/common/knownModels.ts` for current provider model IDs.
- Persistent data: `~/.mux/config.json`, `~/.mux/src/<project>/<branch>` (worktrees), `~/.mux/sessions/<workspace>/chat.jsonl`.

## Documentation Rules

- No free-floating Markdown. User docs live in `docs/` (read `docs/README.md`, add pages to `docs.json` navigation, use standard Markdown + mermaid). Developer notes belong inline as comments.
  - Exception: the `rfc` folder contains human-written RFCs for implementation planning.
- For planning artifacts, use the `propose_plan` tool or inline comments instead of ad-hoc docs.
- Do not add new root-level docs without explicit request; during feature work rely on code + tests + inline comments.
- External API docs already live inside `/tmp/ai-sdk-docs/**.mdx`; never browse `https://sdk.vercel.ai/docs/ai-sdk-core` directly.

### Code Comments

- When delivering a user's request, leave their rationale in the code as comments.
- Generally, prefer code comments that explain the "why" behind a change.
- Still explain the "what" if the code is opaque, surprising, confusing, etc.

## Key Features & Performance

- Core UX: projects sidebar (left panel), workspace management (local git worktrees or SSH clones), config stored in `~/.mux/config.json`.
- Fetch bulk data in one IPC call—no O(n) frontend→backend loops.
- **React Compiler enabled** — auto-memoization handles components/hooks; do not add manual `React.memo()`, `useMemo`, or `useCallback` for memoization purposes. Focus instead on fixing unstable object references that the compiler cannot optimize (e.g., `new Set()` in state setters, inline object literals as props).
- **useEffect** — Before adding effects, consult the `react-effects` skill. Most effects for derived state, prop resets, or event-triggered logic are anti-patterns.

## Tooling & Commands

- Package manager: bun only. Use `bun install`, `bun add`, `bun run` (which proxies to Make when relevant). Run `bun install` if modules/types go missing.
- Makefile is source of truth (new commands land there, not `package.json`).
- Primary targets: `make dev|start|build|lint|lint-fix|fmt|fmt-check|typecheck|test|test-integration|clean|help`.
- Full `static-check` includes docs link checking via `mintlify broken-links`.
- `.mux/tool_env` is sourced before every `bash` tool call. Use `run_and_report <step_name> <command...>` when running multiple validation steps in one call.
- Do not pipe/redirect/wrap `run_and_report` output; keep helper markers intact so Mux can show clean step status.
- `./scripts/wait_pr_ready.sh <pr_number>` is the preferred tail-end helper after local validation and after you've exhausted useful local work.
- `./scripts/wait_pr_checks.sh <pr_number>` is the checks watcher; `wait_pr_ready.sh` must execute `wait_pr_checks.sh --once` on each loop iteration.
- `./scripts/wait_pr_codex.sh <pr_number>` is the Codex gate used by `wait_pr_ready.sh`.

## Browser Automation

Use `agent-browser` for web automation. Run `agent-browser --help` for all commands.

Core workflow:

1. `agent-browser open <url>` - Navigate to page
2. `agent-browser snapshot -i` - Get interactive elements with refs (@e1, @e2)
3. `agent-browser click @e1` / `fill @e2 "text"` - Interact using refs
4. Re-snapshot after page changes

## PR Workflow (Codex)

- If a PR has Codex review comments, address + resolve them, then re-request review by commenting `@codex review` on the PR.
- Prefer `gh` CLI for GitHub interactions over manual web/curl flows.

- User preference: when work is already on an open PR, push branch updates at the end of each completed change set so the PR stays current.
- **PR creation gate:** Do **not** open/create a pull request unless the user explicitly asks (e.g., "open a PR", "create PR", "submit this"). By default, complete local validation, commit/push branch updates as requested, and let the user review before deciding whether to open a PR.

> PR readiness is mandatory. You MUST keep iterating until the PR is fully ready.
> A PR is fully ready only when: (1) Codex confirms approval (thumbs-up reaction on the PR description or an approval comment like "Didn't find any major issues"), (2) all Codex review threads are resolved, and (3) all required CI checks pass.
> You MUST NOT report success or stop the loop before these conditions are met.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [coder/mux](https://github.com/coder/mux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
