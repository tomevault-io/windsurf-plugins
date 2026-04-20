---
trigger: always_on
description: `mo` is a CLI tool that opens Markdown files in a browser with live-reload. It runs a Go HTTP server that embeds a React SPA as a single binary. The Go module is `github.com/k1LoW/mo`.
---

# Copilot Instructions for mo (Markdown Opener)

## What is mo

`mo` is a CLI tool that opens Markdown files in a browser with live-reload. It runs a Go HTTP server that embeds a React SPA as a single binary. The Go module is `github.com/k1LoW/mo`.

## Build & Run

Requires Go and [pnpm](https://pnpm.io/). Node.js version is managed via `pnpm.executionEnv.nodeVersion` in `internal/frontend/package.json`.

```bash
# Full build (frontend + Go binary, with ldflags)
make build

# Dev: build frontend then run with args (uses port 16275, foreground mode)
make dev ARGS="testdata/basic.md"

# Dev with tab groups (-t can only specify one group per invocation)
make dev ARGS="-t design testdata/basic.md"

# Frontend code generation only (called by make build/dev via go generate)
make generate

# Run all tests (frontend + Go)
make test

# Run a single frontend test (vitest)
cd internal/frontend && pnpm test src/utils/buildTree.test.ts

# Run Go tests only
go test ./...

# Run a single Go test
go test ./internal/server/ -run TestHandleFiles

# Run linters (oxlint for frontend, golangci-lint + gostyle for Go)
make lint

# CI target (install dev deps + generate + test)
make ci
```

### CLI Flags

- `--port` / `-p` — Server port (default: 6275)
- `--target` / `-t` — Tab group name (default: `"default"`)
- `--open` — Always open browser
- `--no-open` — Never open browser
- `--watch` / `-w` — Glob pattern to watch for matching files (repeatable)
- `--unwatch` — Remove a watched glob pattern (repeatable)
- `--close` — Close files instead of opening them
- `--clear` — Clear saved session for the specified port
- `--status` — Show status of all running mo servers
- `--shutdown` — Shut down the running mo server
- `--restart` — Restart the running mo server
- `--foreground` — Run mo server in foreground (do not background)
- `--json` — Output structured data as JSON to stdout
- `--dangerously-allow-remote-access` — Allow remote access without authentication (trusted networks only)

## Architecture

**Go backend + embedded React SPA**, single binary.

- `cmd/root.go` — CLI entry point (Cobra). Handles single-instance detection: if a server is already running on the port, adds files via HTTP API instead of starting a new one.
- `internal/server/server.go` — HTTP server, state management (mutex-guarded), SSE for live-reload, file watcher (fsnotify). All API routes use `/_/` prefix to avoid collision with SPA route paths (group names).
- `internal/static/static.go` — `go:generate` runs the frontend build, then `go:embed` embeds the output from `internal/static/dist/`.
- `internal/frontend/` — Vite + React 19 + TypeScript + Tailwind CSS v4 SPA. Build output goes to `internal/static/dist/` (configured in `vite.config.ts`).
- `version/version.go` — Version info, updated by tagpr on release. Build embeds revision via ldflags.
- `testdata/` — Sample Markdown files (GFM, mermaid, math, alerts, etc.) and fixture projects for tests and dev. Reuse these for new test cases.

## API Conventions

All internal API endpoints are under `/_/api/` and SSE under `/_/events`. The `/_/` prefix is intentional to avoid collisions with user-facing group name routes (e.g., `/mygroup`).

Key endpoints:
- `GET /_/api/groups` — List all groups with files
- `POST /_/api/files` — Add file
- `DELETE /_/api/files/{id}` — Remove file
- `GET /_/api/files/{id}/content` — File content (markdown)
- `PUT /_/api/files/{id}/group` — Move file to another group
- `PUT /_/api/reorder` — Reorder files in a group (group name in body)
- `POST /_/api/files/open` — Open relative file link
- `POST /_/api/patterns` — Add glob watch pattern
- `DELETE /_/api/patterns` — Remove glob watch pattern
- `GET /_/api/status` — Server status (version, pid, groups with patterns)
- `GET /_/events` — SSE (event types: `update`, `file-changed`, `restart`)

## Frontend

- Located in `internal/frontend/`, uses **pnpm** as the package manager.
- React 19, TypeScript, Tailwind CSS v4.
- Markdown rendering: `react-markdown` + `remark-gfm` + `rehype-raw` + `rehype-slug` (heading IDs) + `rehype-sanitize` + `@shikijs/rehype` (syntax highlighting) + `mermaid` (diagram rendering) + `remark-math` + `rehype-katex` (math/LaTeX) + `rehype-github-alerts` (GitHub-style alerts) + `react-zoom-pan-pinch` (image zoom).
- SPA routing via `window.location.pathname` (no router library).
- Key components: `App.tsx` (routing/state), `Sidebar.tsx` (file list with flat/tree view, resizable, drag-and-drop reorder), `TreeView.tsx` (tree view with collapsible directories), `MarkdownViewer.tsx` (rendering + raw view toggle), `TocPanel.tsx` (table of contents, resizable), `GroupDropdown.tsx` (group switcher), `FileContextMenu.tsx` (shared kebab menu for file operations), `WidthToggle.tsx` (wide/narrow content width toggle).
- Custom hooks: `useSSE.ts` (SSE subscription with auto-reconnect), `useApi.ts` (typed API fetch wrappers), `useActiveHeading.ts` (scroll-based active heading tracking via IntersectionObserver).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [k1LoW/mo](https://github.com/k1LoW/mo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
