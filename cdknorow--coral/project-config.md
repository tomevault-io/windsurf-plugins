---
trigger: always_on
description: The Go codebase (`coral-go/`) is the **primary implementation** of Coral. The `coral-python/` is a legacy reference that is no longer authoritative.
---

# CLAUDE.md - Coral Go

## Mission

The Go codebase (`coral-go/`) is the **primary implementation** of Coral. The `coral-python/` is a legacy reference that is no longer authoritative.

**RULES:**
1. The Go codebase is the source of truth. Improve it freely without deferring to Python patterns.
2. Only modify code under `coral-go/` and `tests/`.

## Testing

### Go Unit Tests
```bash
cd coral-go && go test ./...
```

### Legacy Parity Tools (historical reference)
These tools were used during the Python-to-Go migration and may still be useful for comparison:

```bash
# Parity harness (requires both Python and Go servers)
python tests/parity_harness.py

# DB compare tool
cd coral-go && go build -o db-compare ./cmd/db-compare/
./db-compare <python-db> <go-db> [board-py-db] [board-go-db]
```

## Project Structure

```
coral/                  # Legacy Python implementation (historical reference only)
coral-go/               # Primary Go implementation
  cmd/                  # CLI entry points (coral, launch-coral, coral-board, db-compare)
  internal/             # Core packages
    agent/              # Agent implementations (claude, gemini)
    background/         # Background services (git poller, indexer, scheduler, etc.)
    board/              # Message board store
    config/             # Configuration
    jsonl/              # JSONL log reader
    license/            # License checking
    pulse/              # Pulse event parser
    ptymanager/         # PTY/tmux session management
    server/             # HTTP server, routes, frontend assets
    store/              # SQLite storage layer
    tmux/               # Tmux client
  go.mod / go.sum       # Go module dependencies
tests/                  # Parity test harness
  parity_harness.py     # Main harness — starts both servers, runs scenarios, compares DBs
  parity/
    test_scenarios.py   # API scenario tests (tags, settings, webhooks, board, etc.)
Casks/                  # Homebrew Cask definition
Formula/                # Homebrew Formula
scripts/                # macOS build script
icons/                  # App icons and screenshots
```

## Development Workflow

1. Implement or fix features in `coral-go/`.
2. Write or update Go unit tests.
3. Run `go test ./...` to ensure no regressions.
4. Build and manually verify as needed.

## Development

### Agent Docs
Agent docs live in `coral-go/agent_docs/` (single source of truth) and are synced to the static embed directory at build time. The Makefile handles this automatically. Production builds sync via `bundle-frontend.sh`.

### Building (via Makefile)
```bash
cd coral-go && make build       # production build
cd coral-go && make dev         # dev build (skips EULA + license)
cd coral-go && make run         # dev build + run on :8420
cd coral-go && make test-server # dev build + run on :8450 (0.0.0.0)
cd coral-go && make test        # run all tests
```
All Makefile targets sync agent_docs automatically before building.

### Building (manual)
```bash
cd coral-go && go build -o coral ./cmd/coral/
```

### Dev Mode
```bash
cd coral-go && go build -tags dev -o coral ./cmd/coral/ && ./coral --host 127.0.0.1 --port 8420
```
The `dev` build tag skips EULA and license validation.

### Test Server
To spin up a test server for manual verification, use `0.0.0.0` (not `127.0.0.1`) so it's reachable from other machines, and a non-default port to avoid conflicts.

**CRITICAL: Always use a separate data directory for test servers.** The production Coral instance uses `~/.coral/` — running a test server against the same database will corrupt live sessions, kill running agents, and lose state. Use `CORAL_DATA_DIR` to isolate test data:
```bash
cd coral-go && go build -tags dev -o coral ./cmd/coral/ && CORAL_DATA_DIR=/tmp/coral-test ./coral --host 0.0.0.0 --port 8450
```

### Database
- SQLite with WAL mode, stored at `~/.coral/sessions.db`
- Message board DB at `~/.coral/messageboard.db`
- **Never run test servers against the production `~/.coral/` directory.** Always set `CORAL_DATA_DIR` to a temp/test path when running manual test servers or integration tests that start a full server process.

## Releases

### Local Builds
Build installers for each platform from any OS:
```bash
./installers/build-windows.sh 0.7.0   # → installers/dist/coral-windows-amd64-0.7.0.zip
./installers/build-macos.sh 0.7.0     # → installers/dist/Coral-0.7.0.dmg (or .tar.gz on Linux)
./installers/build-linux.sh 0.7.0     # → installers/dist/coral-linux-amd64-0.7.0.tar.gz
```

### Tagged Release (CI)
Pushing a tag triggers the GitHub Actions release workflow. Tag suffixes control build behavior:

#### Build Tiers

Tiers are selected via **compile-time build tags** (not ldflags):

| Tier | Build Tag | EULA | License | Demo Limits |
|------|-----------|------|---------|-------------|
| Prod | (default) | Required | Required | None (LS plan controls) |
| Dev | `-tags dev` | Skipped | Skipped | None |
| Beta | `-tags beta` | Required | Skipped | 2 teams / 8 agents |

#### Tag Naming Conventions

| Tag Format | Tier | Windows Build |
|---|---|---|
| `v0.x.x` | prod | skipped |
| `v0.x.x-dev` | dev | skipped |
| `v0.x.x-beta` | beta | skipped |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cdknorow/coral](https://github.com/cdknorow/coral) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
