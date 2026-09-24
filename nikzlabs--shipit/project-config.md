---
trigger: always_on
description: ShipIt is a browser-based AI editor — describe what you want in chat, the agent writes the code, and you see results live. The agent runs as a CLI inside a session container; Claude Code CLI is the default backend, Codex CLI is also supported, and the architecture is agent-agnostic so additional backends can be added later. Authentication normally uses the user's existing subscription with the chosen provider; a metered API key exists only as a fallback ranked *below* connected accounts.
---

# CLAUDE.md

ShipIt is a browser-based AI editor — describe what you want in chat, the agent writes the code, and you see results live. The agent runs as a CLI inside a session container; Claude Code CLI is the default backend, Codex CLI is also supported, and the architecture is agent-agnostic so additional backends can be added later. Authentication normally uses the user's existing subscription with the chosen provider; a metered API key exists only as a fallback ranked *below* connected accounts.

## Product principles

These govern what ShipIt is. They override convenience, "what other tools do," and "this is how the underlying platform works." A proposal that conflicts with one is wrong. Design docs cite them by number — **§1–§5 are stable identifiers; compress the prose, never renumber.**

### 1. ShipIt is the surface. The user does not leave it.

You build, review, ship, and debug inside one chat-shaped IDE. PRs, CI status, deploy status, diffs, commit history, conversation history, terminal output, preview, and merge conflicts all surface inline — no GitHub tab, hosting dashboard, CI tab, or local terminal required. Sending the user elsewhere is a failure of the product, not a feature.

### 2. Inline beats link-out. Always.

If the upstream system has the data, ShipIt fetches and renders it. Links to GitHub or the cloud provider are **escape hatches** in overflow menus, never the happy path — "View on GitHub" sits beside the PR card, and we never bounce anyone to GitHub's diff viewer. The reason is compounding: once the user is reading a PR on github.com, the next comment, re-request, and fixup happen there too. The cycle has to start somewhere; we keep it inside.

### 3. External tabs are reserved for things ShipIt does not own.

The whole list: **OAuth / auth flows** (Anthropic, GitHub own their login screens), **account and billing pages** (provider billing, GitHub repo creation and settings), and **external documentation** the user explicitly clicks through to. "The PR was created so let's open it" is not on it.

### 4. If we don't render it inline yet, that's a backlog item, not a license to link out.

The link-out acknowledges we haven't built the inline view. It isn't the design.

### 5. Chat is the input surface. The agent is the actor.

The user describes intent; the agent runs the commands, edits the files, reads the logs, runs the tests. We deliberately do **not** give the user shell-shaped affordances — quick-action button rows, command palettes that execute shell, hotkey-bound task runners, "click to run npm test" buttons. Those belong to terminal-shaped IDEs; here they're a category mistake that nudges the product back toward the CLI wrapper it's trying to replace. The legitimate needs already have primitives:

| Need | Primitive |
|---|---|
| Recurring user-driven task ("run the tests", "regenerate types") | Ask the agent in chat. |
| Long-running services (dev server, Prisma Studio, log tailer) | Declare in `docker-compose.yml` with `x-shipit-preview: auto`. |
| One-time setup on a new session (`npm install`, codegen) | `agent.install` in `shipit.yaml`. |
| Ad-hoc shell access for debugging or exploration | The existing terminal panel. |

**Corollary: "saves an LLM round-trip" is not a feature.** Spending a turn on a routine command is the intended cost of chat-shaped UX — it keeps the agent in the loop and the chat history complete. The user still navigates, reviews, instructs, accepts, rolls back, branches, merges; they just don't *operate* the box. That's what they hired the agent for.

### Corollary: how to evaluate proposals

1. Does it need a tab outside ShipIt, or assume GitHub is open elsewhere? Redesign, unless it's a §3 exception.
2. Is the link-out the primary affordance rather than an overflow escape hatch? Redesign.
3. Does it give the user a shell-shaped affordance for a command the agent could run? It solves a problem ShipIt doesn't have — §5.

## Runtime

ShipIt runs inside Docker containers: the orchestrator runs in a container and spawns session worker containers. The one exception is the dogfood inner instance (`RUNTIME_MODE=local`), which skips Docker entirely — see [Dogfooding ShipIt in ShipIt](#dogfooding-shipit-in-shipit).

## Setup

```bash
npm install
```

**Important:** If any npm command fails with missing `node_modules` (e.g., `Cannot find package`), run `npm install` first.

## Commands

- **`npm run test:dev`** — **dev default.** Only tests affected by uncommitted, staged, and untracked (newly created) changes + smoke tests (`-- --list` to dry-run). Use while iterating.
- `npm run test:smoke` — smoke tests only (core connectivity, HTTP bootstrap, git, one client component).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nikzlabs/shipit](https://github.com/nikzlabs/shipit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
