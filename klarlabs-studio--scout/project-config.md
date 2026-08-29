---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository

GitHub: `klarlabs-studio/scout`. Use `gh` CLI with this repo for issues, PRs, releases (e.g. `gh issue list -R klarlabs-studio/scout`).

## What This Is

scout is a Gin-like browser automation library for Go using a pure Chrome DevTools Protocol (CDP) implementation over WebSocket. No rod, no chromedp. It has two API layers: a core `browse` package (Engine/Context/Group/HandlerFunc for developers) and an `agent` package (Session-based, structured-output API for AI agents). Includes an MCP server binary at `cmd/scout` and a conversational browser UI at `cmd/scout ui serve`.

## Commands

```bash
# Build & verify
go build ./...
go vet ./...
golangci-lint run --timeout 5m ./...             # examples/ excluded via .golangci.yml

# Tests (Chrome required for integration tests; integration suites are
# behind the `integration` build tag, so the default run is unit-only)
go test ./...                                                  # unit tests only, no Chrome
go test -tags integration ./...                               # all tests (unit + integration)
go test -tags integration -run TestIntegrationClick ./...     # single test
go test -tags integration -run TestIntegration -timeout 600s ./...  # all integration tests
go test -tags integration -v -race -timeout 600s ./agent/...  # agent package with race detector

# Coverage
make cover-check   # runs tests + coverctl policy enforcement

# Pre-commit hook (gofmt, vet, lint, unit tests, coverctl, nox baseline)
make hooks         # install
bash scripts/pre-commit.sh  # run manually

# AG-UI server (conversational browser UI)
scout ui serve --provider=ollama --model=mistral   # local LLM
scout ui serve --provider=claude                    # Claude (needs ANTHROPIC_API_KEY)
scout ui serve --provider=openai                    # OpenAI (needs OPENAI_API_KEY)
cd ui && npm install && npm run dev                 # Vue frontend at :3000
```

## Architecture

**Three interfaces, one CDP engine:**

The root `browse` package follows Gin's patterns — `Engine` manages browser lifecycle, `Context` carries page state through a `HandlerFunc` middleware chain, `Group` organizes tasks with shared middleware, `Selection`/`SelectionAll` wrap DOM elements. The `agent` package wraps all of this into a single `Session` type with structured JSON-serializable responses, auto-wait, content distillation, and mutex-protected concurrency safety.

The `internal/agui` package adds a third interface: an AG-UI protocol HTTP server (`scout ui serve`) that streams SSE events to a Vue frontend. An LLM (Claude, OpenAI, or Ollama) interprets user messages and calls scout tools via an agentic loop. Browser state (URL, title, screenshot) streams to the frontend as JSON Patch deltas.

**CDP data flow:**

`Page.call(method, params)` → `Conn.CallSessionCtx(ctx, sessionID, method, params)` — every CDP command is scoped to a session ID and carries a `context.Context` for cancellation. Events flow back through `Conn.dispatchEvent` which filters by `sessionID` before invoking handlers. `Page.Close()` cancels its context, removes all session-scoped event handlers, and closes the CDP target.

**Key internal contracts:**

- `Page.getRootNodeID()` caches the DOM document root node ID. It is invalidated when `Navigate()` is called (sets `rootNodeID = 0`). This halves CDP round-trips for `QuerySelector`/`QuerySelectorAll`.
- `Page.Navigate()` validates URLs via `URLValidator` — blocks non-http(s) schemes and private IPs by default. Tests must use `WithAllowPrivateIPs(true)`.
- Resilience middleware (Retry, Timeout, CircuitBreaker, Bulkhead) uses `c.SaveIndex()`/`c.RestoreIndex()` to replay the downstream handler chain. `RestoreIndex` clears `errors` and `aborted` but preserves `keys` — data set by prior handlers survives retries.
- `agent.Session` holds a `sync.Mutex` and locks on every public method. Internal helpers (`ensurePage`, `observeInternal`, `pageResult`, `discoverFormInternal`) are called with the lock held — they must not re-lock.
- The `internal/wait` package provides the polling implementation. `Page.WaitLoad()` and `Page.WaitForSelector()` delegate to `wait.ForLoad()` and `wait.ForSelector()`.
- MCP eval tool is gated behind `SCOUT_ENABLE_EVAL=1` env var due to arbitrary code execution risk.
- MCP server uses lazy session creation — browser starts on first tool use, not at startup. `configure` tool changes settings without restart.
- Playwright-style selectors (`:text('...')`, `:has-text('...')`) are translated to JS text-content lookup via `agent/selector.go`.
- `annotated_screenshot` returns element list only by default (no base64 image). Set `include_image: true` for the image.
- Action replay: `recordAction()` is called inside Navigate/Click/Type when `s.recording != nil`. Playbooks validate expected outcomes.
- Multi-tab: `tabManager` tracks named pages. Default page becomes "default" tab when `OpenTab` is first called.
- DOM diff classification: `classifyDiff()` categorizes mutations as modal_appeared, form_error, notification, loading_complete, etc.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [klarlabs-studio/scout](https://github.com/klarlabs-studio/scout) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
