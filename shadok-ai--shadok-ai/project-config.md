---
trigger: always_on
description: Read this first. It exists so you can evolve shadok-ai **without re-scanning
---

# CLAUDE.md — shadok-ai

Read this first. It exists so you can evolve shadok-ai **without re-scanning
the whole codebase** and **without repeating the mistakes already made**.
Keep it up to date when you change architecture, invariants, or the protocol.

## What shadok-ai is

A **web cockpit that drives multiple real Claude Code TUI sessions in
parallel**. Each "channel" in the UI is one `claude` process. It runs on the
user's **Claude subscription** (not the API), via a pseudo-terminal or tmux —
so it's Claude Code piloting Claude Code, with a browser chat on top.

It largely **built itself** (agents in git worktrees). See `docs/architecture.md`
for the deep dive; `docs/superpowers/specs/*` for per-feature design specs.

## Build / run / restart (do it exactly this way)

```bash
npm run build          # tsc → dist/  (ALWAYS build before restarting)
```

The server runs under a **detached supervisor**, `node dist/main.js`, launched
from the repo. The supervisor doesn't run your working tree: it runs the
**npm-installed** copy (`~/.shadok-ai/app/node_modules/shadok-ai/dist/server.js`)
and auto-updates it. Every merge to main publishes a new version
(`.github/workflows/publish.yml`, version = `major.minor.<commits since that minor>`), so a
running instance picks up merged work on its own within minutes.

UI: **http://localhost:3789**. Logs: `~/.shadok-ai/local-supervisor.log`.
Health: `curl -s -o /dev/null -w '%{http_code}' localhost:3789/`.

- `package.json` stays at `0.1.0` locally — the CI computes the published
  version. A local version "behind" npm is normal, not a symptom.
- `CLAUDE_CODE_OAUTH_TOKEN` is only for the `/usage` (pace) endpoint; `claude`
  itself authenticates via the keychain.
- Never `cat` the token or print it. Extract it in the shell, pass via env.

### Running YOUR build (to check a fix before merging)

**Run it side by side on a free port. Never take over 3789.** Stopping the
running instance kills every sibling `sk-*` session mid-work — including, when an
agent does it, the very session driving the change (invariant 8,
`context/pilot-prompt.md`). Don't edit `~/.shadok-ai/config.json` either: the
running instance reads it.

```bash
npm run build
PORT=3899 SHADOK_VERSION_CHECK_MIN=0 node dist/server.js
```

Three properties make that safe, and all three are load-bearing — change one and
you're back to the old failure:

- `PORT` sets where the port walk *starts* (`START_PORT`, server.ts:102). It does
  not remove the walk: `MAX_PORT_TRIES = 20` still applies, so a busy 3899 climbs
  to 3900 — never down toward 3789.
- `SHADOK_VERSION_CHECK_MIN=0` closes the gate on the version poll
  (`if (VERSION_CHECK_MIN > 0)`, server.ts:878). That poll is what would install
  the npm release and exit for the supervisor to respawn — i.e. your build
  vanishing without a word. Note `triggerUpdate` has a *second* caller, the
  `POST /autoupdate` handler (server.ts:893); it needs a deliberate request, so
  just don't tick the GUI checkbox. `autoUpdate: false` in the config reaches the
  same end, but by mutating state the running instance shares — prefer the env var.
- The Telegram token is keyed by **launch dir** (`cfg.tokens?.[cwd]`,
  config.ts:98), so an instance started from a worktree has no token and starts no
  bridge. That's what lets the two coexist: only one process can long-poll the bot.
  A `TELEGRAM_BOT_TOKEN` in the env overrides the per-dir lookup and *would* steal
  the bot — so confirm your startup output has no `telegram:` line.

Verify you're really on your build: `curl -s localhost:3899/version` must report
the local `current` (0.1.0), not the published one. Check
`curl -s -o /dev/null -w '%{http_code}' localhost:3789/` still answers `200`
before and after. Stop your instance when done; nothing to restore.

No interactive browser? Borrow Playwright from `~/projects/aibrowser`
(`node_modules/playwright`, required by absolute path — don't install into the
worktree), screenshot, and **read the screenshots back**. Capture the console too:
a CSP violation or a failed module import is how invariants 10 and 12 show up, and
both are silent in the DOM.

## Architecture map (file → responsibility)

| File | Responsibility |
|---|---|
| `src/server.ts` | HTTP + WebSocket server. Session registry (`sessions` Map), the `Live` object, the WS message handlers, all endpoints. The hub. |
| `src/session.ts` | `PtyPilot` — drives `claude` in a **node-pty** PTY + `@xterm/headless`. Dies with the server. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shadok-ai/shadok-ai](https://github.com/shadok-ai/shadok-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
