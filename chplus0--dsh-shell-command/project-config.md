---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A [DeepSeek Harness (DSH)](https://github.com/deepseek-ai/deepseek-harness) plugin. It adds:

1. A `/!` command (run one shell command, analyze output, inspired by Claude Code's `!` gesture)
2. A `/terminal` command (interactive PTY popup; transcripts persist automatically when the session received input, and are referenced into the model on demand from a history tab — closing never itself sends anything to the model)

**Note**: DSH's input trigger system only supports `/` and `@` as trigger characters (hardcoded in `dsh-client-ui-input-trigger/lib/types/types.d.ts`). We cannot use a bare `!` prefix like Claude Code does, so we use `/!` instead.

Not a git repository (no `.git`) — this is a working checkout, not something to run `git log`/`git diff` against for history.

## Commands

```sh
node test/unit.mjs           # pure parse/format tests, no dependencies
node test/smoke.mjs          # full apply() + handler against a mock context
node test/smoke-terminal.mjs # terminal registry + hasInput tracking + history-reference RPC (mock)
```

There is no build step (`lib/client.js` is hand-authored, no bundler) and no lint script configured in `package.json`.

The smoke tests import the real `@deepseek-ai/*` peer packages. Outside of a DSH profile install, link them once:

```sh
mkdir -p node_modules/@deepseek-ai
ln -s "$HOME/.dsh/profiles/node_modules/@deepseek-ai/"* node_modules/@deepseek-ai/
```

To exercise the plugin live, install it into a profile (`dsh plugin --profile web add "link:/path/to/dsh-shell-command"`) and restart `dsh web`.

## Architecture

### Two independent execution paths

1. **Single-command (`/!`)** — Client-side input trigger (lib/client.js) registers under `trigger: "/"`, name-matched to `!` (candidates/matchSpace/matchEnter all key off the literal `/!` token). Handler flow: user types `/! ls` → client calls RPC `/shell-bang-rpc` → host (lib/index.js) does `parse.js` → `exec.js` → `format.js` → `message.js` → `agent.followup`. This path requires `connection` and `agents` services. It checks `agent.session.requestHeader()` — if the session has no model request context yet, it refuses with an error message (mirroring dsh-btw behavior).

2. **Multi-command terminal (`/terminal`)** — assembled by `setupTerminalMode` in `lib/rpc.js`, which lazily imports `node-pty` and `ws`. If those (or the `connection`/`agents`/`webServer` services) are missing, terminal mode disables itself with a warning and the `/!` path stays fully functional. Never let a terminal-mode failure take down the whole plugin — this degrade-gracefully contract is intentional, preserve it when editing `rpc.js`.

`/terminal` and `/!` are both registered under `trigger: "/"` with distinct `name`s — DSH's input trigger system only supports `/` and `@` as physical trigger characters (see `TriggerChar` in `dsh-client-ui-input-trigger`), so a bare `!` prefix is not possible.

### Mode parsing (`lib/parse.js`)

Simplified: only extracts the command text from the input. No mode flags anymore — the `/!` trigger always analyzes output (that's its purpose).

### Client-side input triggers (`lib/client.js`)

Two input trigger sources registered via `inputTriggers.registerSource`:

1. **`/!` command trigger** (`createBangSource`): `trigger: "/"`, `name: "shell-bang"`, implements `candidates`/`onPick`/`matchSpace`/`matchEnter`. Candidates filter: only shows when `request.query` starts with `!`. On submit, calls RPC `/shell-bang-rpc` with the command text. The RPC result determines success/error shown to the user.

2. **`/terminal` trigger** (`createTerminalSource`): `trigger: "/"`, `name: "terminal"`, name-matched to `/terminal`. Implements the full contract, `matchEnter` returns a Promise. Opens the terminal popup via `TerminalController.open()`.

### Execution backends (`lib/exec.js`)

`runCommand` has two backends behind one normalized outcome shape:
- `ctx.subprocess` (harness-managed seam, preferred when present): scrubbed environment, tree-scoped `SIGTERM → grace → SIGKILL`, bounded tail-retaining stream collection.
- `node:child_process` fallback (used when the seam is absent): scrubbed env via `scrubbedParentEnv()` from `@deepseek-ai/dsh-subprocess`, in-memory bounded buffers, process-level (not tree-level) kill.

Timeout vs. cancellation is always classified from the caller-owned `AbortSignal`s (`timeoutSignal.aborted` / `signal.aborted`), never inferred from exit code/signal — keep that ordering if you touch the outcome-normalization tail of `runCommand`.

### RPC handling (`lib/index.js`)

The `/!` trigger's RPC handler (`/shell-bang-rpc`) is registered via `connection.rpc.handle`. It:
1. Resolves the agent from the session ID.
2. Checks `agent.session.requestHeader()` — if undefined, rejects with "No model request exists yet. Send a message before using /! commands."
3. Parses the command (via `parse.js`), runs it (via `exec.js`), formats the output (via `format.js`), and injects the analysis message (via `message.js` and `agent.followup`).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CHplus0/dsh-shell-command](https://github.com/CHplus0/dsh-shell-command) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
