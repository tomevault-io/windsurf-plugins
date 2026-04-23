---
trigger: always_on
description: A single-binary Go CLI tool that opens a browser-based UI for reviewing code changes and markdown files with GitHub PR-style inline commenting. Supports multi-file review with git diff rendering and structured review file output for AI coding agents.
---

# Crit — Development Guide

## What This Is

A single-binary Go CLI tool that opens a browser-based UI for reviewing code changes and markdown files with GitHub PR-style inline commenting. Supports multi-file review with git diff rendering and structured review file output for AI coding agents.

## Project Structure

```
crit/
├── main.go              # Entry point: subcommand dispatcher + individual runX() functions
├── server.go            # HTTP handlers: REST API (session, file, comments CRUD, finish, share, config)
├── session.go           # Core state: multi-file session, comment storage, review file persistence, SSE
├── watch.go             # File/git watching, round-complete handlers, comment carry-forward
├── git.go               # Git integration: branch detection, changed files, diff parsing
├── github.go            # GitHub PR sync: fetch/post PR comments, crit comment CLI, review file I/O
├── config.go            # Config file loading: ~/.crit.config.json + .crit.config.json merge, ignore patterns
├── diff.go              # LCS-based line diff for inter-round markdown comparison
├── status.go            # Terminal status output formatting
├── daemon.go            # Daemon lifecycle: spawn, connect, stop, session registry
├── share.go             # Share/unpublish to crit-web, share CLI subcommand
├── plans.go             # Plan file detection and handling
├── integrations.go      # Integration config installation (crit install <agent>)
├── gen_integration_hashes.go     # Script to regenerate integration content hashes
├── integration_hashes_gen.go     # Generated integration content hashes
├── main_test.go         # Subcommand argument parsing tests
├── testutil_test.go     # Shared test helpers (initTestRepo, runGit, writeFile, flushWrites)
├── *_test.go            # Tests for all Go files above
├── frontend/
│   ├── index.html       # HTML shell — references style.css, theme.css, and app.js
│   ├── app.js           # All JS (multi-file state, rendering, comments, SSE, keyboard shortcuts)
│   ├── style.css        # Layout, diff rendering, file sections, components
│   ├── theme.css        # Color themes (light/dark/system CSS variables)
│   ├── markdown-it.min.js    # Markdown parser (provides source line mappings via token.map)
│   ├── highlight.min.js      # Syntax highlighter core (languages bundled)
│   └── mermaid.min.js        # Mermaid diagram renderer
├── integrations/        # Drop-in config files for AI coding tools (claude-code, cursor, aider, etc.)
├── e2e/                 # Playwright E2E tests for the frontend
│   ├── playwright.config.ts         # Five projects: git-mode, file-mode, single-file, no-git, multi-file
│   ├── setup-fixtures.sh            # Git repo with feature branch
│   ├── setup-fixtures-filemode.sh   # Plain files without git
│   ├── setup-fixtures-singlefile.sh # Single markdown file
│   ├── setup-fixtures-multifile.sh  # Multiple code + markdown files
│   ├── setup-fixtures-nogit.sh      # File mode without git (no-git)
│   └── tests/           # Test specs (naming convention determines which project runs them)
├── go.mod
├── Makefile             # build / build-all (cross-compile) / update-deps / clean / e2e
├── package.json         # Frontend dependency management (markdown-it, highlight.js, mermaid)
├── copy-deps.js         # Copies npm deps to frontend/ for embedding
├── LICENSE              # MIT
└── README.md
```

## Key Architecture Decisions

1. **All frontend assets embedded** via Go's `embed.FS` — produces a true single binary
2. **No frontend build step** — vanilla JS, no npm/webpack/framework. npm is only for fetching vendor libs.
3. **Multi-file sessions** — `crit` (no args) auto-detects git changes; `crit file1 file2` reviews specific files
4. **Two modes**: "git" mode (auto-detect from git) and "files" mode (explicit file arguments)
5. **markdown-it for parsing** — chosen because it provides `token.map` (source line mappings per block)
6. **Block-level splitting** — lists, code blocks, tables, blockquotes are split into per-item/per-line/per-row blocks so each source line is independently commentable
7. **Diff hunk rendering** — code files show git diffs with dual gutters (old/new line numbers)
8. **Comments reference source line numbers** — stored in the review file (`~/.crit/reviews/<key>.json`) with per-file sections
9. **Real-time output** — review file written on every comment change (200ms debounce)
10. **GitHub-style gutter interaction** — click-and-drag on line numbers to select ranges
11. **File watching** — git mode polls `git status --porcelain`; files mode polls mtimes; reloads via SSE
12. **Localhost only** — server binds to `127.0.0.1`, no CORS headers needed
13. **Two-level config** — `~/.crit.config.json` (global) merged with `.crit.config.json` (project), CLI flags override both. Exception: `agent_cmd` is global-only and cannot be set by project config (prevents malicious repos from hijacking the agent command)
14. **GitHub PR sync** — `crit pull` / `crit push` bridge between the review file and GitHub PR review comments via `gh` CLI

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tomasz-tomczyk/crit](https://github.com/tomasz-tomczyk/crit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
