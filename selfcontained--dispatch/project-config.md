---
trigger: always_on
description: <!-- Keep behavioral rules in sync with CLAUDE.md (used by Claude Code agents). -->
---

# AGENTS Instructions

<!-- Keep behavioral rules in sync with CLAUDE.md (used by Claude Code agents). -->

## CRITICAL: Stay in Your Worktree

If you were started in a git worktree (check: does your working directory contain `.dispatch/worktrees/`?), **every file path you read, write, edit, or delete MUST use the worktree path**. Never `cd` to or reference the parent repo at the main checkout path directly — that is the main working tree and other agents or the user may be working there. Edits to the wrong tree silently land in the wrong place, cause merge conflicts, and get lost.

- Your working directory is your worktree root. Use relative paths or the full worktree-prefixed absolute path.
- Run `git` commands from your worktree — they automatically operate on the correct branch.
- Run `pnpm`, `vitest`, and other tools from the worktree root so they pick up your changes.
- If you need to verify which tree you are in: `git rev-parse --show-toplevel`.

## CRITICAL: Dispatch Status Events (Mandatory)

- You MUST call the `dispatch_event` MCP tool throughout every task turn. These events drive the agent status indicator in the Dispatch UI — the more frequently and accurately you report, the more useful the dashboard becomes.
- **Event types and when to use them:**
  - `working` — You are actively making progress: reading files, writing code, running commands, researching. Use a short message describing the current activity (e.g., "Reading agent-sidebar.tsx", "Running E2E tests", "Refactoring auth middleware").
  - `blocked` — You are stuck and unable to make progress without help or a change in approach. Do not use blocked for errors you are actively investigating or fixing — stay in `working` for those. Message should describe why you are stuck (e.g., "Cannot resolve missing API key", "Repeated test failure after 3 different approaches").
  - `waiting_user` — You need a decision, clarification, or approval before continuing. Message should describe what you need (e.g., "Should I delete the legacy endpoint?", "Need confirmation on color palette").
  - `done` — The task is complete and all checks pass. Message should summarize what was accomplished.
  - `idle` — No meaningful action was taken this turn (e.g., an informational question was answered).
- **Required checkpoints (minimum):**
  1. **Start of turn**: `working` with what you are about to do.
  2. **Phase transitions**: Call `working` again with an updated message whenever your activity shifts to a distinct phase (e.g., moving from research → implementation → testing → validation). This keeps the UI status current.
  3. **When truly stuck**: Switch to `blocked` only when you cannot make further progress on your own.
  4. **Before final response**: Emit a terminal event — `done`, `idle`, `waiting_user`, or `blocked`.
- **Hard requirements:**
  - Do not send a final response unless `done`, `waiting_user`, `blocked`, or `idle` has been emitted in the same turn.
  - If `dispatch_event` fails, report that failure explicitly in the response.
  - Keep messages short (under ~80 chars) — they are displayed in a narrow sidebar.

## UI Validation

- For any UI/layout/style/feature change, validate behavior in Playwright before marking the task complete.
- Include at least one Playwright interaction that covers the changed UI path (for example: open/close panes, modal flow, or action button state changes).
- Capture at least one screenshot per validation flow and publish it with the `dispatch_share` MCP tool. Never leave screenshots local-only.
- For pages with SSE/WebSocket activity, do not use Playwright `waitUntil: "networkidle"` for readiness checks.
- Use `waitUntil: "domcontentloaded"` (or `"load"`) and wait for concrete UI-ready signals (visible control/text/state) instead.
- **Browser cleanup**: When you are done with Playwright validation, call `browser_close` to shut down the browser. Do this before your final `dispatch_event` call. Leaving browsers open wastes resources on headless VMs.

## Component Preference

- Prefer shadcn/ui components over hand-rolled UI when an equivalent shadcn option exists.
- Only hand-roll when there is no suitable shadcn primitive or composition path.

## Frontend State Ownership

- Default to colocating state with the smallest component that fully owns the UI and behavior.
- Do not lift state to a route, layout, or app root unless multiple siblings must coordinate through a shared owner.
- Treat route/layout-level state as a fallback, not a default. Those layers should not become new dumping grounds.
- Treat global client state (including Jotai atoms) as rare and justified only for truly cross-cutting concerns.
- If a piece of state only drives one feature subtree, keep it in that subtree even if persistence is needed. Prefer a small local persistence helper over promoting the whole state domain to global state.
- Prefer URL state over in-memory UI state for shareable/navigation state such as selected entity, active detail pane, or tab when that state should survive reloads, deep links, or back/forward navigation.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [selfcontained/dispatch](https://github.com/selfcontained/dispatch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
