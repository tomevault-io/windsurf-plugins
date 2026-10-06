---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Claude Code Hub — a unified launcher that combines multiple Claude Code tools (Marketplace + Kanban + Cost + Memory Diagnoser) into a single chromeless PWA via iframes and git submodules.

## Commands

```bash
npm start                # Start hub + all sub-apps (http://localhost:3540)
npm run dev              # Start with auto-open browser
npm run test:e2e         # Hub keys in Chrome (needs Chrome); by hand only, not in npm test, the hook or CI
```

`test/fixtures/key-matrix.json` is the list of key presses and what each one must do. `npm test` runs it against the hub page and the SDK; `npm run test:e2e` presses it in Chrome against a scratch hub with fixture apps. A row with `todo` is a known bug. Add a row with each key change.

The hub is usually already running on :3540. Check it with `curl -s -o /dev/null -w '%{http_code}' http://localhost:3540/api/config` (401 without the token still means it is up; add `?token=$(cat ~/.claude-hub/token)` to read it). Use it only for read-only checks. To test a fix, start a second hub on other ports (`--port`, `--marketplace-port`, `--kanban-port`, `--cost-port`, `--memory-port`) so the running one is not disturbed.

CLI flags: `--port <n>`, `--<id>-port <n>` (`kanban`, `marketplace`, `cost`, `memory`), `--hub-dir <path>`, `--pool-size <n>`, `--open`, `--detached` (do not stop when stdin closes), `--install`/`--uninstall` (with `--dir <config dir>`), `--tray`/`--autostart`/`--no-autostart`/`--tray-status` and `--app-id <id>` (Windows and macOS)

**Tray (Windows).** `lib/tray.js` copies `tray/` to `<hub-dir>/tray` with a `config.json` (port, `app`, version, `server.js` and `node` paths, Run value; the tray derives the hub dir from its own folder), then starts `launch.vbs` → hidden PowerShell `hub-tray.ps1`. `--autostart` also writes `HKCU\...\Run\ClaudeCodeHub-<hash of hub dir>`; the same name keys the tray's single-instance mutex, so a scratch `--hub-dir` gets its own tray and Run value. Stop calls `POST /api/shutdown` and falls back to `taskkill /f /t` after 15 s. The tray writes `<hub-dir>/tray.pid` and deletes it on Quit; `--tray-status` checks it against the image name, because a killed tray leaves it behind. The tray starts the hub through `cmd /c … >> <hub-dir>/hub.log` when no live hub is there, and does not restart a hub that dies. Open Hub starts the installed app through its Start Menu shortcut (`chrome_proxy.exe --app-id=…` plus `--app-launch-url-for-shortcuts-menu-item=<url with token>`) when the hub runs on `app.port`, else a browser tab. `config.app` is `{id, port: --port}` with `--app-id`, else `{name: 'Claude Code Hub', port: 3540}`.

**Tray (macOS, experimental).** `tray/mac/hub-tray.js` is the same tray as a JXA script run by `osascript`. `--autostart` writes a launch agent to `~/Library/LaunchAgents`. The tray starts the hub with the `PATH` that `--tray` saved in `config.json` (launchd gives a bare one) and `--detached`, which turns off the hub's stop on stdin close. It cannot run on Windows: test it with the dispatch-only `.github/workflows/tray-macos.yml`, which runs `hub-tray.js --self-test` and uploads screenshots and logs.

**Hub record.** Once it listens, the hub writes `<hub-dir>/hub.json` (`pid`, `port`, `lib/hub-record.js`) and removes it on exit when the pid is its own. Before it listens, a hub whose record names a hub that accepts the dir's token on that port prints that hub's URL and exits, so one hub runs per hub dir. The proxies, the hub listener and the stdin watch start only after that check: a closed stdin would end the check halfway. A record left by a killed hub fails the token check. The tray's `Get-Hub` checks the pid only (node, not exited), because it runs every 3 s. Nothing on disk maps an app id to its origin, and Chrome loads any URL into the app, so the port is the only check.

**Hub plugin.** `plugin/` is a Claude Code marketplace with one plugin, `claude-code-hub`, which holds the `hub-builder` skill. `--install` (`lib/install.js`) copies it to `<hub-dir>/plugin` and registers that copy in one config dir per run: `--dir`, else `CLAUDE_CONFIG_DIR`, else `~/.claude`. Any change under `plugin/` bumps the version in `plugin/plugins/claude-code-hub/.claude-plugin/plugin.json`, because `claude plugin update` only loads a new version. Test an install with scratch `--dir` and `--hub-dir`.

**Hub dir.** `--hub-dir`, else `CLAUDE_HUB_DIR`, else `~/.claude-hub`. It holds `config.json` and `token`. A test hub can use its own dir instead of overriding `USERPROFILE`.

**App ports.** `--<id>-port`, else `port` in the app's `apps` entry in `config.json`, else the manifest's `run.defaultPort`. A busy port falls back to a random one for that run. Nothing saves it: the port is the app's origin, so the app has empty `localStorage` for that run.

## Performance


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NikiforovAll/claude-code-hub](https://github.com/NikiforovAll/claude-code-hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
