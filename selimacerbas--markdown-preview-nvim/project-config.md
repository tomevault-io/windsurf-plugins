---
trigger: always_on
description: Neovim plugin for live markdown preview in the browser. Pure Lua, no npm: the plugin is Lua alone, and its browser test is a bun package under `tests/browser/`. This file is the index for any coding agent and the source of truth for how to work here; CLAUDE.md includes it. The README is the user's contract and CHANGELOG.md the record of what shipped.
---

# markdown-preview.nvim: agent operating file

Neovim plugin for live markdown preview in the browser. Pure Lua, no npm: the plugin is Lua alone, and its browser test is a bun package under `tests/browser/`. This file is the index for any coding agent and the source of truth for how to work here; CLAUDE.md includes it. The README is the user's contract and CHANGELOG.md the record of what shipped.

## Structure

- `lua/markdown_preview/init.lua`: setup and config, server lifecycle, refresh, scroll sync, mermaid pre-rendering
- `lua/markdown_preview/floor.lua`: the Neovim requirement and its message, the one source the plugin file and the module read
- `lua/markdown_preview/util.lua`: fs helpers, workspace resolution, asset resolution, browser open
- `lua/markdown_preview/ts.lua`: Tree-sitter mermaid extractor plus a Lua-pattern fallback
- `lua/markdown_preview/lock.lua`: the takeover-mode lock file (port, workspace, pid, token; mode 0600)
- `lua/markdown_preview/remote.lua`: HTTP event injection for secondary instances (scroll sync)
- `plugin/markdown-preview.lua`: the floor check and the user commands (`:MarkdownPreview`, `:MarkdownPreviewRefresh`, `:MarkdownPreviewStop`)
- `assets/index.html`: the browser preview app (CSS plus JS, one file)
- `lazy.lua`: the spec lazy.nvim reads from this plugin, listing live-server.nvim alone; it stays in step with the README's lazy.nvim snippet
- `tests/`: the headless suites, `helpers.lua` (the harness), `run.sh` (the runner) and `floor_smoke.sh` (the below-floor smoke)
- `tests/browser/`: the browser smoke test (`smoke.test.ts`), a bun package that pins Playwright exactly (`package.json`, `bun.lock`)
- `.githooks/commit-msg`: the hook `make hooks` copies into the clone with a copy of `.githooks/message-policy`, the one message policy the CI `commits` job runs too; the hook runs that copy, never the working tree's (a merge runs the hook with the merged tree checked out), so `make hooks` runs again after a policy change; `tests/message_policy_test.sh` measures both

## Sibling dependency

- live-server.nvim (`selimacerbas/live-server.nvim`, cloned beside this repo as `../live-server.nvim`) is the pure Lua HTTP server with SSE this plugin drives; one maintainer edits both, and commits stay per repo.
- The live-server floor is v1.5.0 in three places that move together: `H.live_server_floor` in `tests/helpers.lua`, and `LIVE_SERVER_FLOOR` and `LIVE_SERVER_FLOOR_SHA` in `.github/workflows/ci.yml`; the `live-server floor is the pinned tag` step of the local action `.github/actions/live-server-floor`, which the five jobs on the floor share, reds when they disagree, and the gating test jobs run on that commit.
- live-server exports `require("live_server.server").features` (`token_auth`, `host_binding`, `asset_route`); the plugin reads `asset_route` and warns once when it is missing.
- APIs used: `server.start(cfg)` (an instance with `.port`), `server.stop(inst)`, `server.reload(inst, path)`, `server.send_event(inst, event, data)`, `server.update_target(inst, root, index)`, `server.connected_client_count(inst)`, `server.features`, and `util.random_token(16)` from `live_server.util` (the session token).
- Endpoints used: `GET /__live/inject?event=<type>&data=<json>&t=<token>` (remote.lua), and `GET /__live/events?t=<token>` (the event stream) and `GET /__live/asset?p=<relpath>&t=<token>` (the preview page).

## Architecture

- Neovim writes the buffer to `content.md` in a workspace directory under `stdpath("cache")/markdown-preview/` (takeover always; multi unless `workspace_dir` is set); live-server serves it and pushes SSE events (`reload` on change, `scroll` with the cursor line).
- The browser renders with markdown-it, highlight.js, KaTeX and mermaid (loaded from CDNs) and diffs the DOM with morphdom.
- Auth: a per-session token gates five surfaces, `content.md`, the `asset_root` sidecar, the SSE stream, the inject endpoint and the asset route; on the loopback default the index page is not gated and carries the token (`data-live-token`), and on a non-loopback `host` the index page is gated too and carries no token, which the browser takes from the `?t=` URL.
- Instance modes: `takeover` (the default; one shared workspace, port 8421 under the default `port = 0`, a lock file elects the primary) and `multi` (a per-buffer workspace, or `workspace_dir` when set, and a server per instance on an OS-assigned port under the default `port = 0`).
- `mermaid_renderer = "rust"` pre-renders mermaid fences through the `mmdr` CLI; the default renders them in the browser.

## Conventions

- Neovim 0.10 or newer: `lua/markdown_preview/floor.lua` states the requirement and the message once, below it the plugin file and the module stop with that message, and CI proves the refusal on a real Neovim 0.9.5 with `tests/floor_smoke.sh`.
- `vim.uv` for async I/O; Lua patterns, never regex quantifiers.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [selimacerbas/markdown-preview.nvim](https://github.com/selimacerbas/markdown-preview.nvim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
