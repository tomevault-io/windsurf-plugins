---
trigger: always_on
description: Read this before changing anything. It records what exists, where it lives, how it ships, and the owner's rules. Last updated 2026-09-27, at version 0.2.2 on PyPI and Homebrew, with unreleased changes in the working tree.
---

# Tokenmeter: context for agents

Read this before changing anything. It records what exists, where it lives, how it ships, and the owner's rules. Last updated 2026-09-27, at version 0.2.2 on PyPI and Homebrew, with unreleased changes in the working tree.

## What it is

A local web dashboard showing token usage, cost, cache misses and rate-limit windows for every prompt in Claude Code, Codex and GitHub Copilot CLI. It reads the logs those tools already write to disk. No hooks, no instrumentation, nothing sent anywhere. Python 3.9+ standard library only, plus one HTML file. The owner, Serkan Korkut, wants it to become a publishable, pitchable product.

## Repositories

| Path | Remote | Visibility | Role |
|---|---|---|---|
| `~/repo/tokenmeter` | `serkankorkut/tokenmeter` | public | the app, this repo |
| `~/repo/homebrew-tap` | `serkankorkut/homebrew-tap` | public | Homebrew formula, public README, demo GIF, release tarballs |
| `~/repo/tokenmeter-site` | `serkankorkut/tokenmeter-site` | public | marketing and docs site, live at tokenmeter.fyi |
| `~/repo/serkan.fyi` | private | | owner's personal site; tokenmeter-site copies its build setup and style |

## Layout of this repo

- `tokenmeter/server.py` — everything server side: parsers, store, HTTP handler, budget and export threads, CLI. About 500 lines.
- `tokenmeter/index.html` — the whole dashboard, vanilla JS and inline CSS. About 600 lines.
- `tokenmeter/pricing.json` — bundled list prices per model prefix plus `_plans`, `_budget`, `_context_windows`. `_plans` ships as zeros on purpose.
- `tokenmeter/__init__.py`, `__main__.py` — package glue; `__version__` lives in `server.py`.
- `tokenmeter start` / `stop` (in `control()` in server.py) run the dashboard in the background and open the browser. Tokenmeter manages its own login service: a launchd agent `fyi.tokenmeter` in `~/Library/LaunchAgents` on macOS (KeepAlive on the program path, so it stops retrying after uninstall), a systemd user unit `~/.config/systemd/user/tokenmeter.service` on Linux, and a detached process where neither works (Windows, SSH, CI). For Homebrew installs the service points at `$(brew --prefix)/opt/tokenmeter/bin/tokenmeter`, which survives upgrades. `start` removes the old Homebrew agents (`sh.brew.tokenmeter`, `homebrew.mxcl.tokenmeter`), restarts a running copy whose version differs, and prints the URL last. The formula has no `service do` block since 0.2.6 so the install output ends with our message; running `brew` inside `post_install` is refused by Homebrew (tested 2026-09-28). The Tier 2 notice some users see comes from Homebrew about their machine (for example outdated Command Line Tools) and cannot be removed by the formula.
- Ports: `ports_to_try()` returns only the chosen port when `--port`, `TOKENMETER_PORT` or `_port` in the user config is set; otherwise 7788 through 7798. A port counts as free only if nothing accepts a connection on it and the bind succeeds, because on macOS `SO_REUSEADDR` lets 127.0.0.1 bind over another app's 0.0.0.0 listener and would silently hijack it. `start --port N` saves `_port` only after confirming the port is usable. `start`, `stop`, the skill and the menu bar script find Tokenmeter by probing `/api/health` across the range.
- `test_server.py` — the single assert-based self-check. Run `python3 test_server.py`, expect `ok`.
- `SKILL.md`, `.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json`, `install.sh` — Claude Code plugin and Codex skill packaging. The owner asked not to promote the skill publicly for now.
- `menubar/tokenmeter.1m.sh` — SwiftBar/xbar plugin reading `/api/summary`. Not included in the PyPI package.
- `release/brew-formula.sh`, `release/brew-stats.sh` — Homebrew release and download stats.
- `.github/workflows/publish.yml` — publishes to PyPI on any `v*` tag via trusted publishing, environment `pypi`.
- `docs/demo.gif` — also copied to the tap repo, which is where the public README reads it from.
- `release/demo_data.py` — writes a synthetic 60-day dataset for Claude Code, Codex and Copilot. All marketing images come from a Tokenmeter instance running on it. Never capture screenshots or GIFs from the owner's real data: it contains employer repo names, PR links, ticket keys and colleagues' names. A GIF made from real data was public for a week and had to be replaced on 2026-09-27; it still exists in the tap repo's git history.

## How the data flows

1. `Store.sources()` yields Claude JSONL under `~/.claude/projects/**`, Codex JSONL under `~/.codex/sessions/**`, the Copilot SQLite at `~/.copilot/session-store.db`, and team files under `~/.tokenmeter/team/*.json`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [serkankorkut/tokenmeter](https://github.com/serkankorkut/tokenmeter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
