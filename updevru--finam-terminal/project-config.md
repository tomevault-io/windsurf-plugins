---
trigger: always_on
description: **finam-terminal** is a Go-based Terminal User Interface (TUI) application designed to interact with the Finam Trade API. It demonstrates how to authenticate, retrieve account information, and fetch market data (quotes, positions) using gRPC.
---

# Finam Terminal Project

## Project Overview

**finam-terminal** is a Go-based Terminal User Interface (TUI) application designed to interact with the Finam Trade API. It demonstrates how to authenticate, retrieve account information, and fetch market data (quotes, positions) using gRPC.

### Key Technologies
*   **Language:** Go (v1.26)
*   **API Protocol:** gRPC
*   **TUI Library:** `github.com/rivo/tview`
*   **Configuration:** `github.com/joho/godotenv`
*   **SDK:** `github.com/FinamWeb/finam-trade-api/go`
*   **Testing:** `google.golang.org/grpc/test/bufconn` (in-process gRPC for integration tests)

## Architecture

The project follows a clean modular structure:

*   **`main.go`**: The entry point. Handles configuration loading, API client initialization, and starting the UI loop.
*   **`api/`**: Contains the `Client` struct and methods for interacting with the Finam gRPC services. Encapsulates the complexity of the raw API calls.
    *   `client.go`: Core client — `NewClient` creates a TLS connection, `newClientFromConn` initializes service clients, authenticates, starts token refresh, and loads the asset cache. `newClientFromConn` is also used by integration tests to create clients via `bufconn` without TLS.
*   **`api/testserver/`**: In-process mock gRPC server for integration testing (see [Testing](#testing) section).
*   **`ui/`**: Manages the Terminal User Interface.
    *   `app.go`: Main `App` struct, state management, tabbed view (Positions/History/Orders), and lifecycle (Run/Stop).
    *   `render.go` / `components.go`: Responsible for drawing UI elements (tables, lists, headers).
    *   `data.go`: Data fetching logic for trades history and active orders.
    *   `search.go`: Dedicated search window for finding securities.
    *   `profile.go`: Full-screen instrument profile overlay with asset details, trading parameters, and chart. Renders instrument-type-specific fields (futures: expiration + contract size; options: + strike; bonds: face value + currency) and open interest in the Quote section for derivatives.
    *   `chart.go`: Unicode candlestick chart renderer with smart time labels.
    *   `input.go`: Keyboard input handlers for all views (navigation, shortcuts, order actions).
    *   `modal.go`: Order placement modal with dynamic fields for Market/Limit/Stop/TP/SL+TP order types.
    *   `utils.go`: UI utility functions (number formatting, account ID masking).
*   **`config/`**: Handles loading environment variables from `.env` or system environment.
*   **`models/`**: Shared data structures used across the application to represent accounts, quotes, positions, trades, and orders. Key fields include `LotSize` and `Name` for instrument metadata. `AccountInfo.LoadError` is set when an account fails to load from the broker. `AccountInfo.DailyPnL` holds the daily P&L value. `Order` includes extended fields for stop/limit prices, conditions, validity, and SL/TP quantities.
*   **`version/`**: Build-time version metadata. Exposes `Version`, `Commit`, and `BuildDate` as **package-level vars** (not consts — the linker can only override vars via `-ldflags -X`). `String()` returns the display string used by the UI header: a release tag verbatim (`v1.2.3`), or a dev build with VCS info (`dev (a1b2c3d)` or `dev (a1b2c3d, dirty)`), falling back through `runtime/debug.ReadBuildInfo()` when no commit is injected. `Info()` returns the raw tuple for diagnostics.

## Getting Started

### Prerequisites
*   Go 1.26 or higher
*   Finam Trade API Token (obtain from Finam Developer Portal)

### Installation

1.  Clone the repository.
2.  Install dependencies:
    ```bash
    go mod tidy
    ```

### Configuration

The application requires an API token.

1.  Copy the example configuration:
    ```bash
    cp .env.example .env
    ```
2.  Edit `.env` and add your token:
    ```env
    FINAM_API_TOKEN=your_actual_token_here
    ```

### Building and Running

**Run directly:**
```bash
go run main.go
```

**Run with specific account (by index):**
```bash
go run main.go -account 0
```

**Build executable:**
```bash
go build -o finam-trade.exe main.go
./finam-trade.exe
```

**Build with version metadata (recommended for local distribution):**
```bash
make build
```
The `build` target injects `git describe --tags --always --dirty` as `Version`, `git rev-parse HEAD` as `Commit`, and the current UTC time as `BuildDate` via `-ldflags -X` against the `version` package. The resulting binary shows the resolved version in the TUI header.

If you skip `make` and use a plain `go build .` (note the `.`, not `main.go` — `main.go` does not embed `vcs.*` settings), the binary still falls back to `runtime/debug.ReadBuildInfo()` and renders `dev (<short-sha>)` (or with `, dirty` when the working tree has changes).

### Releasing a New Version

To cut a release, just push a `vX.Y.Z` git tag — `.github/workflows/release.yml` is triggered on `push: tags: 'v*'` and will:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [updevru/finam-terminal](https://github.com/updevru/finam-terminal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
