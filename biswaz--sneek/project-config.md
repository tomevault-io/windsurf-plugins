---
trigger: always_on
description: macOS app + CLI daemon for managing custom commands with secret resolution, SSH tunnels, and Claude Code MCP integration.
---

# Sneek

macOS app + CLI daemon for managing custom commands with secret resolution, SSH tunnels, and Claude Code MCP integration.

## Why This Exists

Managing connections to production infrastructure (databases, APIs, internal services) requires juggling passwords, SSH keys, tunnel configs, and connection strings. These secrets end up in plaintext config files, shell history, or environment variables.

Sneek solves this: define custom commands (like `pg-prod`) in a GUI, configure secrets (pulled from Keychain/1Password/Bitwarden at runtime — never stored), optional SSH tunnels, and expose them to Claude Code as MCP tools or to the terminal as shell scripts.

The core requirement: **give Claude Code access to prod databases without exposing secrets** (passwords, IPs, SSH tunnel options) in the conversation or config files.

## Architecture

**GUI + CLI Daemon** pattern (chosen over monolith or generated-scripts-only):
- The daemon runs without the GUI open — Claude Code can use commands even when the app isn't visible
- The GUI is just a config editor — no runtime logic
- Both share `SneekLib`

```
┌──────────────┐     ┌─────────────────────────────────────────┐     ┌──────────────┐
│  Sneek.app   │     │              sneekd                     │     │  Claude Code  │
│  (SwiftUI)   │────▶│  Tunnel Manager  │  Session Manager     │◀────│  (MCP stdio)  │
│  config only │ IPC │  Secret Resolver │  MCP Server          │     │              │
└──────────────┘     │  Script Generator│  IPC Server           │     └──────────────┘
                     └─────────────────────────────────────────┘
                              ▲                                        ┌──────────────┐
                              │ IPC                                    │   Terminal    │
                              └────────────────────────────────────────│ sneekd run   │
                                                                      └──────────────┘
```

**Why this over alternatives:**
- **Not a monolith** — daemon can run headless, MCP server doesn't need the GUI
- **Not generated scripts only** — persistent sessions need a long-running process; tunnel health monitoring needs a daemon
- **Not native DB drivers** — chose persistent CLI sessions (generic, works with any tool) over native drivers (would need per-DB Swift drivers, not generic)

## Project Structure

```
Sources/
  SneekLib/                  # Core library (shared by app and daemon)
    Models.swift             # CommandConfig, SecretRef, TunnelConfig, MCPConfig, SneekConfig
    ConfigStore.swift        # JSON config loading/saving from ~/.config/sneek/
    SecretResolver.swift     # SecretProvider protocol + Keychain/1Password/Bitwarden/Env
    TemplateEngine.swift     # {{variable}} interpolation
    TunnelManager.swift      # SSH tunnel spawn/health check/teardown (actor)
    SessionManager.swift     # Persistent subprocess sessions with sentinel parsing (actor)
    IPCProtocol.swift        # Unix domain socket IPC client/server
    MCPServer.swift          # stdio JSON-RPC MCP server for Claude Code
    ScriptGenerator.swift    # Shell script + Claude MCP config generation
    Daemon.swift             # Orchestrates all components, handles IPC requests
  sneekd/
    Sneekd.swift             # CLI entry point (swift-argument-parser subcommands)
  SneekApp/
    SneekApp.swift           # App entry point (menubar + window scenes)
    AppState.swift           # Observable state, config loading, daemon status
    MenuBarView.swift        # Menubar popover (search, command list, badges)
    CommandEditorView.swift  # Full command editor form
Tests/
  SneekLibTests/             # 136 checks across 49 tests
    TestRunner.swift         # Custom test harness (no XCTest — see note below)
    Main.swift               # Test entry point
    ModelsTests.swift        # JSON round-trip, all secret providers, edge cases
    ConfigStoreTests.swift   # Load, save, delete, reload, global config
    SecretResolverTests.swift # Real env, mock providers, merge logic
    TemplateEngineTests.swift # Substitution, missing vars, edge cases
    TunnelManagerTests.swift  # Mock tunnel lifecycle
    SessionManagerTests.swift # Oneshot, blocked patterns
    MCPServerTests.swift      # JSON-RPC protocol, tool list, filtering
    ScriptGeneratorTests.swift # Script content, permissions, install-mcp
    IntegrationTests.swift    # Real stack: keychain, sessions, read-only, MCP
docs/
  superpowers/specs/
    2026-04-02-sneek-design.md  # Full design spec
```

## Build & Test

```bash
# REQUIRED: Xcode must be installed (not just CommandLineTools)
# SwiftUI and the test runner need the full Xcode SDK
export DEVELOPER_DIR=/Applications/Xcode.app/Contents/Developer

swift build              # Build all 3 targets (sneekd, Sneek, SneekTests)
swift run SneekTests     # Run tests (custom runner — NOT swift test)
swift run Sneek          # Launch GUI — auto-starts the daemon (one command, dev + prod)
swift run sneekd --help  # CLI help (daemon ops only — GUI handles start/stop)
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [biswaz/sneek](https://github.com/biswaz/sneek) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->
