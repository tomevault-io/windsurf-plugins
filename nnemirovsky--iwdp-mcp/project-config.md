---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Dev Commands

```bash
go build ./...                    # Compile (quick check)
make build                        # Compile to bin/iwdp-mcp + bin/iwdp-cli
make install                      # Install both binaries via go install
make test                         # Run unit + smoke tests (go test ./... -v -count=1)
make test-e2e                     # Run e2e tests (boots iOS Sim + iwdp, tests ALL tools)
make test-coverage                # Run tests with coverage → coverage.html
make sim-setup                    # Boot iOS Simulator + iwdp (prints IWDP_SIM_WS_URL)
make sim-teardown                 # Shut down simulator + iwdp
make fmt                          # gofumpt -w . (format all Go files)
make lint                         # golangci-lint run ./...
make tidy                         # go mod tidy
make clean                        # Remove bin/ and coverage files
```

## Pre-commit

Before committing, always run `make fmt` and `make lint` and fix any issues.

## How ios-webkit-debug-proxy Works

- **Port 9221** — listing port: `http://localhost:9221/json` returns connected devices
- **Port 9222+** — each device gets an incremented port starting at 9222
  - `http://localhost:9222/json` → pages for first device
  - `http://localhost:9223/json` → pages for second device, etc.
- Each page has a `webSocketDebuggerUrl` for WebSocket connection
- The WebSocket speaks **WebKit Inspector Protocol**

### Target-Based Message Routing

iwdp uses **Target routing** — you cannot send domain commands directly on the WebSocket. On connect, iwdp sends a `Target.targetCreated` event with a `targetId`. All subsequent commands must be wrapped in `Target.sendMessageToTarget`:

```
Client → Target.sendMessageToTarget { targetId, message: JSON-stringified inner command }
iwdp → WebKit
WebKit → Target.dispatchMessageFromTarget { targetId, message: JSON-stringified response }
iwdp → Client
```

- The outer `sendMessageToTarget` gets an empty ack `{"result":{},"id":N}` — this is ignored
- The inner command has its own ID used for response matching (dual-ID routing)
- `webkit.Client` handles this transparently: it waits up to 100ms for `Target.targetCreated` on connect; if received, all `Send()` calls auto-wrap in Target routing
- Mock test servers don't send `Target.targetCreated`, so the client falls back to direct mode (no wrapping)

### Domain Enable/Disable Through Target Routing

Some `<Domain>.enable` methods work through iwdp Target routing (e.g., `Debugger.enable`, `Canvas.enable`, `Worker.enable`, `Animation.enable`). Others like `CSS.enable` hang without a response. Most actual domain methods (DOM.getDocument, CSS.getMatchedStylesForNode, CSS.getComputedStyleForNode, CSS.setStyleText, Runtime.evaluate) work without explicit enabling.

### Known Limitations

- `CSS.enable`, `CSS.getAllStyleSheets`, `CSS.getStyleSheetText` hang through iwdp Target routing. The command is sent but no response comes back, and the hang corrupts the connection pipeline (all subsequent commands on the same connection will also hang). The tool implementations detect Target routing and return an error immediately instead.
- `Page.snapshotRect` requires explicit pixel dimensions — compute them first via `Runtime.evaluate` (see `TakeScreenshot` in page.go).
- Only **one WebSocket debugger connection per page** — simulator tests use a `sync.Once` shared connection pattern.
- iwdp sends error `data` as a JSON array `[{"code":...,"message":...}]` — `ErrorData.Data` is `json.RawMessage` to handle this.

## Architecture

Two binaries, one shared `internal/` package tree:

- `cmd/iwdp-mcp/` — MCP server binary (stdio transport, 100+ tools registered via `mcp.AddTool`)
- `cmd/iwdp-cli/` — CLI binary (devices, pages, eval, navigate, screenshot, cookies, dom, console, network)

### Package Layout

- `internal/webkit/` — WebKit Inspector Protocol client
  - `client.go` — WebSocket connection with Target-based message routing for iwdp, concurrent-safe writes (`writeMu`), dual-ID request/response routing, event dispatch
  - `types.go` — Protocol type definitions for all domains (DOM, CSS, Network, Runtime, Debugger, etc.)
  - `domains.go` — Domain enable/disable helpers
  - `testutil/mock.go` — Mock WebSocket server for unit tests
- `internal/tools/` — Tool implementations shared by MCP server and CLI
  - One file per domain: page.go, runtime.go, dom.go, css.go, network.go, storage.go, console.go, debugger.go, timeline.go, memory.go, profiler.go, animation.go, canvas.go, layertree.go, worker.go, audit.go, security.go, interaction.go
- `internal/proxy/` — iwdp process detection and management
  - `IsRunning()` — checks port 9221 (listing port). Never use a device port for detection.
  - `ListDevices()` — HTTP GET on port 9221
  - `ListPages(port)` — HTTP GET on a device-specific port
  - `ListAllPages()` — discovers all devices, queries each for pages
  - `DevicePort(entry)` — extracts port from device entry URL (e.g., `localhost:9222` → 9222)

### Key Patterns

- All WebKit protocol communication goes through `webkit.Client`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nnemirovsky/iwdp-mcp](https://github.com/nnemirovsky/iwdp-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
