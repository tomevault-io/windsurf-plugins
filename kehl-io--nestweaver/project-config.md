---
trigger: always_on
description: cargo build                                                 # build all crates
---

# NestWeaver

## Build & Test

```sh
cargo build                                                 # build all crates
cargo build --release                                       # release binary
cargo test                                                  # run all tests
cargo test -p nestweaver-schema                             # test one crate
cargo clippy --workspace --all-targets -- -D warnings       # lint (zero warnings)
cargo fmt --all -- --check                                  # format check
cargo fmt --all                                             # format in place
```

## Daemon Architecture

All CLI commands and MCP tool calls route through a background daemon process
that owns the LadybugDB write lock. The daemon auto-starts on the first CLI
invocation and self-terminates after an idle timeout. The client auto-restarts
the daemon on version mismatch. Shutdown is graceful: the daemon drains active
write RPCs before exiting (up to `NESTWEAVER_DRAIN_TIMEOUT_SECS`, default 660s).
Indexing is CPU-throttled to a rolling 5s duty-cycle window so a saturated
daemon stays under macOS CPU-violation limits; tune with
`NESTWEAVER_INDEX_CPU_PERCENT` (percent of one core, 1–99, default 50; `0` or
`>=100` disables throttling). The var reaches the daemon two ways: from the
shell env for directly-spawned daemons, or baked into the launchd plist's
`EnvironmentVariables` at `daemon start` time (launchd jobs don't inherit the
shell env — re-run `daemon start` after changing it).

**The daemon is the sole writer to the DB file.** Never run `sqlite3` or other
tools against the DB while the daemon is running. The `--no-daemon` flag and
`NESTWEAVER_NO_DAEMON=1` env var exist only for CI/testing. Bypassing the
daemon risks WAL corruption from concurrent access. If you see "database
locked" errors, stop the daemon (`nestweaver daemon stop`) rather than using
`--no-daemon`.

## macOS App (preferred on Mac)

On macOS, prefer the native `.app` bundle over the CLI daemon. It provides:
- Menubar status icon for quick access to the web UI
- Metal GPU acceleration (GUI session context required for MTLCompilerService)
- Automatic daemon lifecycle with crash recovery
- Daemon coexistence — detects running daemons and connects instead of duplicating

```sh
# Build and launch
bash app/build.sh
open target/release/NestWeaver.app

# NestWeaver.app is source-build-only until a release job publishes a .app or DMG.
```

The web UI defaults to port 3000 (`nestweaver ui`); the macOS .app uses 9377. The app is menubar-only (no Dock icon).

When helping users install on macOS, recommend the `.app` bundle first. Fall back to `nestweaver daemon start` only for headless/CI environments.

## Run

```sh
# Index a repo and query it
nestweaver index                         # auto-detects repo root from .git
nestweaver index --repo ./testdata/js    # explicit repo path
nestweaver index --repo ./client --name my-client  # explicit repo name for multi-repo setups
nestweaver index --stats                 # show timing and statistics after indexing
nestweaver watch                         # live re-indexing via filesystem watcher with debouncing
nestweaver watch ./my-project            # watch a specific directory
nestweaver context greet                 # task-focused subgraph via PPR
nestweaver context greet --intent find-definition          # intent-tuned PPR
nestweaver context greet --limit 20                        # cap connected nodes
nestweaver context src/main.js           # seed from all symbols in a file
nestweaver search "greet"
nestweaver symbol "greet" --json
nestweaver impact "greet" --depth 3
nestweaver impact "fetchRegions" --repo my-service  # filter impact to a specific repo
nestweaver repo-map --token-budget 2000
nestweaver summary --level symbol        # hierarchical code summaries (symbol/file/cluster)

# Graph analysis
nestweaver hubs                          # most connected hub nodes (degree centrality + PageRank)
nestweaver bridges                       # architectural chokepoints (betweenness centrality)
nestweaver clusters                      # functional communities (adaptive resolution: 0.3 for >10K symbols, 0.5 default)
nestweaver pr-impact                     # PR blast radius with risk scoring (Low/Medium/High)
nestweaver pr-impact --sarif             # SARIF 2.1.0 for GitHub code scanning / VS Code SARIF viewer
nestweaver pr-impact --strict            # exit 2 on a contract-verified breaking change (advisory by default)
nestweaver affected-tests --base-ref main  # tiered regression-test selection for a diff
nestweaver rts-eval record-truth --sha X --failed-test-files a.test.ts  # CI reports full-suite outcome
nestweaver rts-eval report               # measured recall/breadth of past selections (nw-037 loop)
nestweaver dead-code                     # detect unreachable symbols via entry point reachability

# Export
nestweaver export --format cypher        # graph export (cypher, graphml, mermaid)
nestweaver export --format msgpack       # graph snapshot for WASM engine

# Markdown brain (`.brainignore` for glob exclusion patterns; `--ignore` flag for ad-hoc)
nestweaver brain add ~/Documents/Obsidian/MyVault

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kehl-io/nestweaver](https://github.com/Kehl-io/nestweaver) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
