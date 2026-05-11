---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Test Commands

```bash
make build              # CGO_ENABLED=0, pure Go (no DuckDB)
make build-full         # CGO_ENABLED=1 -tags duckdb (requires C compiler)
make build-all          # Cross-compile all release targets (linux/darwin/windows)
make test               # go test ./...
make test-race          # go test -race ./...
make vet                # go vet ./...
make fmt                # gofmt + goimports
make lint               # golangci-lint run ./...
make tidy               # go mod tidy
make run ARGS="--adapter sqlite --file demo.db"
```

Run a single test:
```bash
go test ./internal/completion/ -run TestFuzzyMatch
```

PostgreSQL integration tests require a running instance and `gotermsql_test` database:
```bash
go test ./internal/adapter/postgres/ -run TestIntegration -v
GOTERMSQL_PG_DSN="postgres://user:pass@host/db" go test ./internal/adapter/postgres/ -run TestIntegration
```
Integration tests auto-skip if PostgreSQL is unavailable.

Version info is injected via LDFLAGS from git tags/commit/date. Releases use `make build-all` + `gh release create` (targets: linux/darwin amd64+arm64, windows amd64). Homebrew tap at `sadopc/homebrew-tap`. Release archives follow naming `gotermsql_X.Y.Z_{os}_{arch}.tar.gz`.

## Architecture

Bubble Tea (Elm Architecture) TUI. Root model in `internal/app/app.go`.

**Message routing priority in `Update()`:**
1. `tea.WindowSizeMsg` → recalculate layout
2. `tea.KeyMsg` → connection manager (if visible, blocks all input) → help overlay (if visible, blocks all input) → autocomplete (if visible) → global keys → focused pane handler
3. Application messages: Connect, SchemaLoaded, QueryResult, NewTab, etc.

**Focus system:** Three panes (`PaneSidebar`, `PaneEditor`, `PaneResults`). Tab/Shift+Tab cycles. Alt+1/2/3 jumps directly. Each pane's component has `Focus()`/`Blur()` methods that control whether it processes input.

**Multi-tab state:** `tabStates map[int]*TabState` — each tab owns its own `editor.Model` and `results.Model`. Always nil-check `activeTabState()`. `results.New(tabID)` takes the tab ID for message routing.

**Layout system:** Tab bar (top) + status bar (bottom) + main area. Main area splits into sidebar (left, fixed width) + editor/results (right, percentage-based vertical split). Resizable via Ctrl+Arrow keys. Sidebar: 15–50% width. Editor height: 20–80%.

**Border width accounting:** lipgloss `.Width(w)` sets *content* width; borders add 2 chars on top. All components must use `.Width(m.width - 2)` to fit within their allocated space.

**Message re-export layer:** All message types live in `internal/msg/msg.go`. The file `internal/app/messages.go` re-exports them as type aliases for convenience within the app package. When adding new messages, update both files.

## Async Message Flow & Staleness Guards

Query execution and schema loading are async (goroutines returning `tea.Cmd`). Three generation/ID counters prevent stale results:

**`TabState.RunID uint64`** — per-tab query execution counter. Incremented in `executeQuery()` before dispatching. `QueryStartedMsg`, `QueryResultMsg`, and `QueryErrMsg` all carry `RunID`. Handlers discard messages where `msg.RunID != ts.RunID`.

**`Model.connGen uint64`** — connection generation counter. Incremented in `ConnectMsg` handler. All async messages carry `ConnGen`: `SchemaLoadedMsg`, `SchemaErrMsg`, `QueryStartedMsg`, `QueryResultMsg`, `QueryErrMsg`. Handlers discard messages where `msg.ConnGen != m.connGen`.

**`Model.executingTabID int`** — tracks which tab has the in-flight query. When a tab is closed while executing (`CloseTabMsg`), the query is cancelled and `m.executing` is cleared. When `QueryResultMsg`/`QueryErrMsg` arrives for a closed tab (`ts == nil`), `m.executing` is still cleared if `msg.TabID == m.executingTabID`.

**`QueryStreamingMsg`** also carries `ConnGen` and `RunID`. Its handler must close the iterator on stale/mismatched messages to avoid resource leaks.

**When adding new async messages:** Always capture the relevant generation counter at dispatch time and check it in the handler. The closure in `tea.Cmd` functions must capture the counter value, not a pointer to the model.

## Connection Lifecycle

- **Connect:** `connect()` returns a `tea.Cmd` that opens connection + pings. On success, sends `ConnectMsg`.
- **Reconnect:** `ConnectMsg` handler closes old `m.conn`, cancels in-flight schema load (`m.schemaCancel()`), assigns new connection, increments `connGen`.
- **Shutdown:** `main.go` calls `m.Connection()` on the final model and closes it. History DB is closed via `defer hist.Close()` (panic-safe).
- **Query cancellation:** `executeQuery()` creates a cancellable context and stores cancel in `m.cancelFunc`. For streaming SELECTs, the context has no timeout (iterator may be browsed for hours); for non-streaming queries, a 5-minute timeout is applied. Ctrl+C calls both `m.cancelFunc()` (cancels context) and `m.conn.Cancel()` (database-level cancellation).
- **Schema loading:** `loadSchema()` uses `context.WithTimeout(30s)`. Cancel func stored in `m.schemaCancel`; previous load cancelled on reconnect or quit.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sadopc/gotermsql](https://github.com/sadopc/gotermsql) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
