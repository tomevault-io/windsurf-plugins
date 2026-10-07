---
trigger: always_on
description: Neovim plugin for live markdown preview in the browser. Pure Lua, no npm: the plugin is Lua alone, and its browser test is a bun package under `tests/browser/`. This file is the index for any coding agent and the source of truth for how to work here; CLAUDE.md includes it. The README is the user's contract and CHANGELOG.md the record of what shipped.
---

# mdkite.nvim: agent operating file

Neovim plugin for live markdown preview in the browser. Pure Lua, no npm: the plugin is Lua alone, and its browser test is a bun package under `tests/browser/`. This file is the index for any coding agent and the source of truth for how to work here; CLAUDE.md includes it. The README is the user's contract and CHANGELOG.md the record of what shipped.

## Structure

- `lua/mdkite/init.lua`: setup and config, server lifecycle, refresh, scroll sync, mermaid pre-rendering
- `lua/mdkite/floor.lua`: the Neovim requirement and its message, the one source the plugin file and the module read
- `lua/mdkite/util.lua`: fs helpers, workspace resolution, asset resolution, browser open
- `lua/mdkite/ts.lua`: Tree-sitter mermaid extractor plus a Lua-pattern fallback
- `lua/mdkite/lock.lua`: the takeover-mode lock file (port, workspace, pid, token and the address the primary's server bound; mode 0600) under `stdpath("cache")/mdkite`, removed only by the instance that wrote it; a lock counts only while its pid runs and its port's server takes its token, and an older release's lock under `stdpath("cache")/markdown-preview` is read, never written
- `lua/mdkite/remote.lua`: HTTP event injection for secondary instances (scroll sync) and the lock check's request
- `plugin/mdkite.lua`: the floor check and the user command `:MdKite` (`start`, `stop`, `refresh`, `toggle`), and through 2.x the commands before 2.0.0 (`:MarkdownPreview`, `:MarkdownPreviewRefresh`, `:MarkdownPreviewStop`), each running its subcommand after one warning a session
- `lua/markdown_preview.lua`: the module's name before 2.0.0, through 2.x: it hands back `mdkite`'s table and warns once a session; `vim.g.loaded_markdown_preview` opts out beside `vim.g.loaded_mdkite`
- `assets/index.html`: the browser preview app (CSS plus JS, one file)
- `lazy.lua`: the spec lazy.nvim reads from this plugin, listing kitehost.nvim alone; it stays in step with the README's lazy.nvim snippet
- `tests/`: the headless suites, `helpers.lua` (the harness), `run.sh` (the runner) and `floor_smoke.sh` (the below-floor smoke)
- `tests/browser/`: the browser smoke test (`smoke.test.ts`), a bun package that pins Playwright exactly (`package.json`, `bun.lock`)
- `.githooks/commit-msg`: the hook `make hooks` copies into the clone with a copy of `.githooks/message-policy`, the one message policy the CI `commits` job runs too; the hook runs that copy, never the working tree's (a merge runs the hook with the merged tree checked out), so `make hooks` runs again after a policy change; `tests/message_policy_test.sh` measures both

## Sibling dependency

- kitehost.nvim (`selimacerbas/kitehost.nvim`, named live-server.nvim before its 2.0.0, cloned beside this repo as `../kitehost.nvim`, or still as `../live-server.nvim`) is the pure Lua HTTP server with SSE this plugin drives; one maintainer edits both, and commits stay per repo.
- The plugin requires `kitehost.server` and `kitehost.util`, new in kitehost v2.0.0, so requiring them is the version check: without them a start refuses with one error naming the floor, and an error kitehost raises while loading goes on as raised. The lookup runs at load and again at every start until it finds them, so a kitehost put on the runtimepath after the plugin loaded is used.
- The kitehost floor is v2.0.0 in four places that move together: `H.kitehost_floor` in `tests/helpers.lua`, `KITEHOST_FLOOR` in `lua/mdkite/init.lua`, and `KITEHOST_FLOOR` and `KITEHOST_FLOOR_SHA` in `.github/workflows/ci.yml`; the `kitehost floor is the pinned tag` step of the local action `.github/actions/kitehost-floor`, which the five jobs on the floor share, reds when the helper and the workflow disagree, `start_failure_test` reds when the helper and the module do, and the gating test jobs run on that commit.
- kitehost exports `require("kitehost.server").features` (`token_auth`, `host_binding`, `asset_route`, `host_check`, `cors_list`, `start_raises`); a start requires `host_check` and `start_raises` (`REQUIRED_FEATURES` in `lua/mdkite/init.lua`) and refuses a kitehost without either with one error naming the floor and the missing flag, before any state is made; the asset route is older than both.
- APIs used: `server.start(cfg)` (an instance with `.port`), `server.stop(inst)`, `server.reload(inst, path)`, `server.send_event(inst, event, data)`, `server.update_target(inst, root, index)`, `server.connected_client_count(inst)`, `server.wildcard_loopback(ip)` when present, `server.features`, and `util.random_token(16)` from `kitehost.util` (the session token).
- Endpoints used: `GET /__live/inject?event=<type>&data=<json>&t=<token>` (remote.lua), the same route with no event to check a lock's holder (`?t=<token>`, or a bare `?`), and `GET /__live/events?t=<token>` (the event stream) and `GET /__live/asset?p=<relpath>&t=<token>` (the preview page).

## Architecture


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [selimacerbas/mdkite.nvim](https://github.com/selimacerbas/mdkite.nvim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
