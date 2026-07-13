---
trigger: always_on
description: This file is read by AI coding agents. Keep it current when the app architecture, tool contracts, prompt pipeline, or UI workflows change.
---

# Ally — AGENTS.md

This file is read by AI coding agents. Keep it current when the app architecture, tool contracts, prompt pipeline, or UI workflows change.

---

## Build & Run Commands

| Command | Description |
|---------|-------------|
| `wails dev` | Run the desktop app in development mode with hot reload |
| `wails build` | Build a distributable desktop binary |
| `go build ./...` | Compile the Go backend |
| `go test ./...` | Run Go tests |
| `cd frontend && npm install` | Install frontend dependencies |
| `cd frontend && npx vite build` | Build the Vue frontend |
| `cd frontend && npm test` | Run frontend unit tests when present |
| `wails generate module` | Regenerate Go-to-JS bindings after adding or changing exported Go methods |

## Git Convention

When developing Ally itself, push with the author set to `ally agent`:

```powershell
git add -A
git -c user.name="ally agent" -c user.email="ally@agent.dev" commit -m "..."
git push origin main
```

This uses `-c` to override the commit author per-command only, leaving the developer's global Git config unchanged.

---

## Release Process

Git tags and GitHub Releases are the source of truth for Ally versions. Release tags use `vMAJOR.MINOR.PATCH`; `.github/workflows/build.yml` injects the published tag through `ALLY_BUILD_VERSION`. Do not treat `frontend/package.json`'s `0.0.0` as the app release version and do not change it only for a release.

1. Synchronize and identify the current release:
   - Require a clean worktree on `main` and make sure it matches `origin/main`.
   - Run `git fetch origin --tags --prune`.
   - Inspect `git tag --sort=-v:refname` and the latest GitHub Release before choosing the next semantic version.
2. Choose the next version and prepare the release notes:
   - Use a patch bump for compatible fixes or maintenance, a minor bump for backward-compatible features, and a major bump for breaking changes.
   - Summarize user-visible changes from `git log <previous-tag>..HEAD`; do not claim changes that are not present in that range.
   - End the notes with `**Full Changelog**: https://github.com/Bronya0/ally-agent/compare/<previous-tag>...<new-tag>`.
3. Verify the exact commit that will be released:
   - `npm --prefix frontend ci`
   - `npm --prefix frontend run build`
   - `go test ./...`
   - `go build ./...`
   - `wails build -clean -s -skipbindings`
4. Commit the release-related repository changes and push `main` to `origin`. Recheck that the worktree is clean and local `HEAD` equals `origin/main`.
5. Publish a non-draft GitHub Release targeting `main`, with tag `<new-tag>`, title `Ally <new-tag>`, and the prepared notes. Authentication must come from GitHub CLI login or a `GITHUB_TOKEN` environment variable with repository contents write permission. Never place a token value in repository files, release notes, scripts, or copied command text.

Publishing the Release triggers `.github/workflows/build.yml`, which builds and attaches the Windows x64, Linux x64, and macOS universal packages.

---

## Repository Layout

```
├── app.go                    # Main Wails-bound backend: config, chat loop, tools, sessions, skills, context accounting
├── model_provider.go         # Model provider adapter layer: OpenAI Chat, OpenAI Responses, Anthropic Messages
├── mcp.go                    # MCP manager: config loading, process lifecycle, tool discovery, tool dispatch
├── scheduler.go              # Persistent scheduled Agent tasks powered by robfig/cron
├── edit_helpers.go           # Read range, changed-line, and diff helpers for text edits
├── main.go                   # Wails app entry point, window options, app binding
├── procattr_windows.go       # Windows process attributes for hidden shell windows
├── procattr_other.go         # Non-Windows process attribute shim
├── *_test.go                 # Go tests for provider, tools, MCP, prompt/config behavior
├── wails.json                # Wails project/build config
├── frontend/
│   ├── src/
│   │   ├── App.vue           # Main Vue app: layout, settings, sessions, command handling, runtime events
│   │   ├── style.css         # Global dark theme styles, settings, tool cards, chat, modals
│   │   ├── app.css           # App entry CSS
│   │   ├── main.js           # Vue mount
│   │   ├── components/
│   │   │   ├── AllyWordmark.vue
│   │   │   ├── AskToolCard.vue
│   │   │   ├── CodeView.vue
│   │   │   ├── ComposerInfoBar.vue
│   │   │   ├── ContextUsageInline.vue
│   │   │   ├── DiffView.vue
│   │   │   ├── GitDiffModal.vue
│   │   │   ├── MessageAttachments.vue
│   │   │   ├── ReadGroupCard.vue
│   │   │   ├── RenderBoundary.vue
│   │   │   ├── ScheduledTasksPanel.vue
│   │   │   ├── SplashScreen.vue
│   │   │   ├── SubagentInlineCard.vue
│   │   │   ├── ToolCallCard.vue
│   │   │   └── WelcomeMessage.vue
│   │   └── utils/
│   │       ├── ascii.js
│   │       ├── config.mjs
│   │       ├── diff.js
│   │       └── toolPreview.mjs
│   ├── wailsjs/              # Generated Wails JS/TS bindings
│   ├── package.json
│   └── vite.config.js
└── build/                    # Build assets and platform metadata
```

---

## High-Level Architecture

Ally is a Wails v2 desktop AI coding agent.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Bronya0/ally-agent](https://github.com/Bronya0/ally-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-13 -->
