---
trigger: always_on
description: This repo ships the pi extensions that power its features in `extensions/`:
---

# Agent guidelines for oyster

## Bundled pi extensions

This repo ships the pi extensions that power its features in `extensions/`:

| File | Tool / command | What it does |
|---|---|---|
| `extensions/file-explorer.ts` | `/files` command + `ctrl+o` shortcut | Browse the workspace from the TUI, then edit or download any file. |
| `extensions/hublot.ts` | `hublot` tool | Open/close/list public Cloudflare tunnels to caller-provided local ports. |
| `extensions/loop.ts` | `/loop` command + `loop` tool | Execute Markdown checklist items sequentially in isolated subagents, advancing only after an executable validation script passes. |
| `extensions/pinned-widget.ts` | `pinned_widget` tool | Pin/list/group private files, media, Markdown, directories, and HTTPS links in the right sidebar. |
| `extensions/routine.ts` | `routine` tool | Create/start/stop/teardown session-bound scripts with live progress reporting. |
| `extensions/sudo.ts` | `bash` permission gate | Prompt through Oyster for a masked sudo password before a permissioned command executes. |

pi loads extensions from `~/.pi/agent/extensions/`. To make these bundled files
available (and keep them in sync with the repo), symlink or copy them:

```sh
mkdir -p ~/.pi/agent/extensions
ln -sf "$(pwd)"/extensions/*.ts ~/.pi/agent/extensions/   # symlink — edits here apply immediately
# or:
# cp extensions/*.ts ~/.pi/agent/extensions/              # copy — stable snapshot
```

Restart pi afterwards. When developing an extension through Oyster, stop and restart every affected session runner after changing the extension; existing pi RPC sessions keep their originally loaded extension code. Sending `/reload` from Oyster does not reload an RPC session because `/reload` is handled only by pi's interactive mode, and refreshing the browser is also insufficient.

## Built-in MCP endpoint (Claude Code and other MCP harnesses)

pi has no MCP client, so the extensions above stay the pi integration. For MCP-capable harnesses, the Oyster server itself serves the same tools over MCP (Streamable HTTP, stateless) at `POST /mcp` — `hublot`, `pinned_widget`, `group_pinned_widgets`, `routine` — plus a `bash` tool whose `sudo=true` option mirrors `extensions/sudo.ts`: the complete command runs as root after Oyster's masked password dialog, brokered through `POST /runner/ui-request` to the runner's browser. The implementation lives in `server/http/routes/mcpRoutes.mjs`; its tools call the regular route handlers in-process through `server/http/internalDispatch.mjs`, so there is no second copy of the widget, tunnel, or routine logic and no loopback HTTP.

Every request carries its caller: `POST /mcp?runner=<id>&session=<id>&workdir=<abs path>` with the usual bearer token. The Claude Code driver builds that URL per launch and passes it with `--mcp-config` as an `http` server, using the header `Authorization: Bearer ${OYSTER_TOKEN}`, which Claude Code expands from the inherited environment so the token never appears on a command line. It also allows every `mcp__oyster` tool. Nothing has to be registered in Claude Code's settings, and other MCP clients can use the same URL.

Driver modules such as `server/runner-drivers/claude-code.mjs` are reached only through static imports, which the hot reloader does not cache-bust, so restart the Oyster service (not just the runner) after changing them. When changing a tool, update both the pi extension and the MCP endpoint. Verify with `node --test tests/mcp-routes.test.mjs`.

Pinned files remain private and open through authenticated native Markdown, image, and video displays; use a hublot only for a public live interface. Opening a hublot requires a local `port` (1–65535), with an optional `description` label. Provision the service separately: hublot starts only cloudflared and persists its SQLite entry; closing the tunnel leaves the local service running. The `hublot` and `routine` tools discover the UI server
from `OYSTER_URL` (default `http://127.0.0.1:8080`) and authenticate with
`OYSTER_TOKEN` or the project-root `.ui-token` file.

## Installation

Oyster and its bundled pi submodule use separate lockfiles. Install and build both explicitly; do not run an unscoped workspace install across them.

### Prerequisites

- **Node.js ≥ 22.19** — check with `node --version`. This is required universally because the stable server uses the built-in `node:sqlite` application store, even when pi sessions use JSONL.
- **`pi` source submodule** — initialize `pi/` and build its coding-agent CLI. `PI_BIN` can still select another compatible executable.
- **`cloudflared`** (optional) — only needed for the tunnels feature. Install from [pkg.cloudflare.com](https://pkg.cloudflare.com) if you plan to use tunnel functionality.
- **FFmpeg** (optional outside containers) — converts pinned AVI, MOV, MKV, and M4V videos to cached browser-compatible MP4 playback. Container images and `scripts/install.sh` include it.

### Quick start

```bash
git clone --recurse-submodules <repo-url> oyster && cd oyster
npm ci
npm ci --prefix pi --ignore-scripts
npm run build:pi
npm run build
node server/server.mjs
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SergeiKireevDev/oyster](https://github.com/SergeiKireevDev/oyster) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-29 -->
