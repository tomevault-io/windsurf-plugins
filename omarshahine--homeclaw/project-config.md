---
trigger: always_on
description: HomeClaw exposes Apple HomeKit accessories via a CLI tool, plugins for Claude Code and OpenClaw, and a stdio MCP server for Claude Desktop. It uses a unified Mac Catalyst architecture with an AppKit bridge bundle for the macOS menu bar.
---

# HomeClaw

HomeClaw exposes Apple HomeKit accessories via a CLI tool, plugins for Claude Code and OpenClaw, and a stdio MCP server for Claude Desktop. It uses a unified Mac Catalyst architecture with an AppKit bridge bundle for the macOS menu bar.

## Architecture

```
Claude Code → Plugin (.claude-plugin/) → stdio MCP server (Node.js) ─┐
Claude Desktop → stdio MCP server (Node.js) ─────────────────────────┤
OpenClaw → Plugin (openclaw/) → homeclaw-cli ────────────────────────┤
                                                                     ▼
                                              /tmp/homeclaw.sock (JSON newline-delimited)
                                                                     │
                                              HomeClaw (Mac Catalyst UIKit app)
                                                ├── HomeKitManager (direct, in-process)
                                                ├── SocketServer (for CLI/MCP clients)
                                                └── macOSBridge.bundle (NSStatusItem menu bar)
```

**Single-process design.** `HMHomeManager` requires a UIKit/Catalyst app with the HomeKit entitlement. By making the entire app Catalyst, HomeKit access is direct (no IPC), signing is unified (single archive), and App Store submission is clean. The macOSBridge plugin bundle provides the native macOS menu bar via `NSStatusItem`.

**Note:** The HTTP MCP server (port 9090, bearer token auth) has been disabled. The implementation is preserved in `Sources/homeclaw/MCP/_disabled/` and `Sources/homeclaw/Shared/_disabled/` for reference but is not compiled. All MCP clients now use the stdio server or CLI.

## Project Structure

```
Sources/
  homeclaw/              # Unified Catalyst app (Xcode target via XcodeGen)
    App/                 # UIApplicationDelegate entry point, scene delegates
    Bridge/              # BridgeProtocols.swift (Mac2iOS, iOS2Mac)
    MCP/_disabled/       # Preserved HTTP MCP server code (not compiled)
    HomeKit/             # HomeKitManager, SocketServer, CharacteristicMapper,
                         # AccessoryModel, DeviceMap, CharacteristicCache,
                         # HomeEventLogger, WebhookCircuitBreaker
    Views/               # SettingsView, IntegrationsSettingsView
    Shared/              # AppConfig, AppLogger, HomeClawConfig
    Shared/_disabled/    # Preserved KeychainManager (not compiled)
  macOSBridge/           # AppKit bundle (NSStatusItem menu bar)
    MacOSController.swift  # NSStatusItem + NSMenu
    Info.plist           # NSPrincipalClass: MacOSController
  homeclaw-cli/          # CLI tool (SPM executable + Xcode target)
    Commands/            # list, get, set, search, scenes, automations, status, config, device-map
    Commands/_disabled/  # Preserved token command (not compiled)
    SocketClient.swift   # Direct socket communication
Resources/               # Info.plist, entitlements, app icons
scripts/build.sh         # Build & install script
scripts/archive.sh       # Archive for App Store / TestFlight
mcp-server/              # Node.js stdio MCP server (wraps homeclaw-cli)
openclaw/                # HomeClaw — OpenClaw plugin
  openclaw.plugin.json   # Plugin manifest (configurable binDir)
  src/index.ts           # Plugin entry point
  skills/homekit/        # HomeKit skill definition

App bundle layout (after build):
  Contents/MacOS/HomeClaw          # Catalyst app executable
  Contents/MacOS/homeclaw-cli      # Bundled CLI binary
  Contents/PlugIns/macOSBridge.bundle  # AppKit menu bar plugin
  Contents/Resources/mcp-server.js     # Node.js stdio MCP server
  Contents/Resources/openclaw/         # Bundled OpenClaw plugin files
```

## Build System

Two build systems:
- **Xcode** (`xcodebuild`): Builds `HomeClaw` (Catalyst), `macOSBridge` (macOS bundle), and `homeclaw-cli` (macOS tool)
- **npm** (esbuild): Builds `mcp-server` Node.js MCP server

SPM (`Package.swift`) is retained for CI — it builds `homeclaw-cli` only. The main app is Catalyst-only (Xcode).

The `scripts/build.sh` orchestrates xcodegen + xcodebuild:

```bash
scripts/build.sh --release --install   # Full build + install to /Applications
scripts/build.sh --debug               # Debug build only
scripts/build.sh --team-id ABCDE12345  # Use a different Apple Developer team
npm run build:mcp                      # Build Node.js MCP server only
```

**npm workspaces**: Root `package.json` defines workspaces for `openclaw` and `mcp-server`. Run `npm install` from the project root.

### XcodeGen

The root `project.yml` defines all three targets (HomeClaw, macOSBridge, homeclaw-cli). The generated `.xcodeproj` is gitignored — regenerate after cloning:
```bash
xcodegen generate
```

### Development Workflow

```bash
# Generate project + build debug
scripts/build.sh --debug

# Open in Xcode for debugging
xcodegen generate && open HomeClaw.xcodeproj

# Test HomeKit connection over the socket
echo '{"command":"status"}' | nc -U /tmp/homeclaw.sock
```

## Critical: Entitlements & Distribution

### HomeKit is App Store-only on macOS


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [omarshahine/HomeClaw](https://github.com/omarshahine/HomeClaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
