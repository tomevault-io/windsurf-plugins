---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Starfish Proxy is a Minecraft proxy server designed for use with Hypixel. It acts as a man-in-the-middle between the Minecraft client (1.8.9) and server, enabling packet inspection, modification, and custom functionality through a plugin system.

## Development Commands

```bash
# Install dependencies
npm install

# Start the proxy server (runs on localhost:25565)
npm start

# Build Windows executable
npm run build

# In-game proxy commands
/proxy help                           # Show proxy help
/proxy server <name>                  # Switch servers
/proxy addserver <name> <host:port>   # Add new server
/proxy removeserver <name>            # Remove server
/proxy reauth                         # Force re-authentication
/proxy plugins                        # List loaded plugins

# Plugin commands
/<plugin-name> help                   # Plugin-specific help
/<plugin-name> config                 # Open plugin config UI
/<plugin-name> enable/disable         # Toggle plugin state
```

## Architecture Overview

### Core Components

1. **`src/proxy.js`** - Main entry point, orchestrates the proxy server
   - Manages player sessions and authentication
   - Integrates plugin system, commands, and storage
   - Handles Microsoft authentication with rate limiting

2. **`src/session/`** - Session management
   - `player-session.js`: Manages client↔proxy↔server connections
   - `game-state.js`: Tracks player game state
   - Handles packet forwarding and event emission

3. **`src/plugin-api/`** - Plugin system core
   - Provides wrapped API instances to plugins
   - Modules: `players`, `world`, `entities`, `inventory`, `communication`, `commands`, `display-names`, `server`, `events`, `security`
   - Supports hot-swapping (enable/disable without restart)
   - Dependency resolution with topological sorting
   - Official plugin signature verification

4. **`src/command-system/`** - In-game command framework
   - Module-based command organization
   - Auto-generates config UIs from schemas
   - Built-in help with pagination

5. **`src/storage/`** - Persistent data management
   - Config files: `config/starfish-config.json`, `config/plugins/*.config.json`
   - Plugin data: `data/*.data.json`
   - Auth cache: `auth_cache/[username]/`

6. **`src/packets/`** - Packet processing architecture
   - Packet security enforcement
   - Protocol abstraction layer
   - Safe packet modification system

### Plugin Architecture

Plugins are loaded from `plugins/` directory and receive a wrapped API that prevents direct access to proxy internals. Each plugin can:
- Subscribe to game events and packet interception
- Modify safe packets (chat, display, audio)
- Register commands with help text
- Store persistent configuration and data
- Declare dependencies on other plugins
- Access protected methods (official plugins only)

Plugin loading follows these steps:
1. Scan and extract metadata (name, version, dependencies)
2. Verify signatures for official plugins
3. Resolve dependency graph
4. Load in dependency order
5. Create wrapped API instances per plugin

### Security Restrictions

**IMPORTANT**: The proxy enforces packet safety to maintain compatibility with Hypixel's anti-cheat:
- **Read-only packets**: Player movement, rotation, combat actions, block interactions
- **Modifiable packets**: Chat messages, sounds, titles, particles, tab list display names
- **Cancellable packets**: Chat messages, commands (with restrictions)

## Key Development Guidelines

1. **Plugin Development**:
   - Always use the provided API, never access proxy internals directly
   - Respect packet modification restrictions
   - Clean up resources when disabled (automatic for most API calls)
   - Use the config schema system for settings
   - Follow semantic versioning for plugin versions
   - Declare dependencies using `metadata.dependencies` object
   - Official plugins cannot depend on unofficial ones

2. **Command System**:
   - Commands are module-based (e.g., `/proxy help`, `/anticheat toggle`)
   - Use `ChatBuilder` for consistent formatting
   - Follow the theme system for colors

3. **Event Handling**:
   - High-level events: `player.move`, `chat`, `world.change`, etc.
   - Low-level: `packet.in.[name]`, `packet.out.[name]`
   - Events only fire when plugins are enabled

4. **Configuration**:
   - Main config: Server targets, authentication settings
   - Plugin configs: Auto-generated UI from schemas
   - All configs persist across restarts

## Current Project Status

### Completed Features
- Core proxy functionality with Microsoft authentication
- Plugin system with hot-swapping
- Command system with help and config UIs
- Denicker plugin (nick detection)
- Build system for Windows executables

### TODO List (In Progress)

#### Proxy Core
- Test and complete plugin API
  - Add versioning support
  - Add official plugin system with obfuscation and asymmetric cryptography support
- Implement network settings and buffer size configuration

#### Plugins to Complete/Create

**Anticheat Plugin**:
- Rewrite broken detection checks
- Add icon to cheater display names

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [UrchinGG/Starfish-Proxy](https://github.com/UrchinGG/Starfish-Proxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
