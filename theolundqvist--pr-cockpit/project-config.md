---
trigger: always_on
description: This repository is worked on by humans and coding agents alike. Leave it easier to operate than you found it.
---

# PR Cockpit contributor and agent guide

This repository is worked on by humans and coding agents alike. Leave it easier to operate than you found it.

## Layout

- `server/`: the Bun server. GitHub ingestion, SQLite cache, mutation queue, HTTP API.
- `ui/`: the Svelte 5 front end (runes: `$state`, `$derived`, `$props`, `$effect`).
- `shell/`: the Electron shell. It only provides the macOS window, global hotkeys, and `prcockpit://` links.
- `relay/`: the Cloudflare Worker that receives GitHub webhook events and hands them to a cockpit.
- `scripts/`: install, launch, update, and the `pr-cockpit` CLI.

## Working here

- `bun install` at the repository root, in `ui/`, and in `shell/`.
- `cd ui && bun run dev` for a hot-reloading UI. It proxies `/api` to `127.0.0.1:4820`, so it needs a server on that port.
- `bun test server/<file>.test.ts` for a targeted test. Prefer targeted runs over the whole suite while iterating.
- Screenshot harnesses render at `1600x1200`; keep their default viewport and PNG dimension checks aligned.
- PR detail headers and tabs use the standard width; Conversation centers its 816px primary column and adjacent sidebar beneath them, and Files alone expands to full width.
- Verify landing-page search queries against live GitHub results; captured PR titles can disappear from the search index while the fixture remains valid.
- Match the surrounding style. Comments are for a non-obvious constraint, not for narration.
- Pull-request actions follow GitHub semantics: merge green, update neutral, and destructive actions red.
- Workflow jobs use commit-check semantics: green check for success, red cross for failure, orange spinner while active, and gray minus when skipped. Logs preserve ANSI colors and open at the bottom. Commit status icons and failing-check rows open Cockpit’s Actions tab at the exact commit and job; its selector covers the full mirror range, not GraphQL’s last 100 commits.
- Operational telemetry belongs on a dedicated Usage page in Settings, not among general controls; it keeps three days of hourly context and forecasts the active GitHub quota window.
- File status lives in the colored leading icon; moved files use a distinct move glyph, never a text badge beside the name.
- Agents mutate existing PRs through `pr-cockpit`; use its `--body-file` commands for exact multiline text, never `gh` or direct GitHub APIs.
- Do not perform Vercel or `forge.scape.app` infrastructure work from this repository.
- For manual app recordings, launch `/Users/theo/dev/pr-cockpit`; this `pr-cockpit-v3` checkout is the legacy landing worktree.

## Pull requests

- Functionality, themes, and small UI-polish fixes are welcome.
- New functionality defaults off. Styling is opt-in unless it is minor polish that preserves the default appearance.
- Every pull request includes before-and-after screenshots showing its effect in the app.

## Restarting the local server

`scripts/cockpit` is the full launcher: it builds if needed, starts the server only when it is down, and opens the Electron shell. Do not use it when the request is specifically to restart only the server.

Server environment:

```sh
COCKPIT_PORT=4820                                  # HTTP port
COCKPIT_DATA_DIR="$HOME/.local/share/pr-cockpit"   # SQLite cache, images, queued actions
COCKPIT_REPOS="owner/repo,owner/other-repo"        # seeds tracked repos on first launch
COCKPIT_PROXY="scape-agent"                        # optional Cockpit replica source over SSH
COCKPIT_PROXY_PORT=4820                            # Cockpit port on that SSH host
```

`COCKPIT_MANAGED=1` is exported by `scripts/cockpit` and read by the Electron shell, not the server. It marks the installed instance so a dev launch stays isolated from it.

Replica mode keeps the Bun server, SQLite cache, UI, Electron shell, and image fetches on the Mac. `scripts/cockpit --use-as-proxy HOST` and `pr-cockpit --use-as-proxy HOST ...` tunnel the remote Cockpit API, replicate its inbox state, and disable local GitHub API access. A replica outage must surface as an outage, never fall back to local polling, and local install or quit paths must never shut down the remote server.
Before pushing replica or server-topology changes, verify the installed Electron app, image rendering, and CLI against the intended backend.

Runbook:

The installed backend runs from the `app.pr-cockpit.server` launch agent. Check with `launchctl list | grep cockpit`; the separate `app.pr-cockpit` entry is the renderer and must be left alone.

1. Launch-agent managed: `launchctl kickstart -k gui/$(id -u)/app.pr-cockpit.server`. This restarts it under launchd with the plist's own environment, which a hand-rolled `bun server/main.ts` will not reproduce. The kickstart call can take a minute, so run it in the background rather than assuming it hung.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [theolundqvist/pr-cockpit](https://github.com/theolundqvist/pr-cockpit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-30 -->
