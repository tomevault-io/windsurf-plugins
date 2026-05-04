---
trigger: always_on
description: A multiplayer dungeon game (MUD) inspired by '90s text MUDs like MajorMUD, built with Kotlin Multiplatform, Ktor, Jetpack Compose, and a React-based world editor.
---


# NeoMud — Project Instructions

A multiplayer dungeon game (MUD) inspired by '90s text MUDs like MajorMUD, built with Kotlin Multiplatform, Ktor, Jetpack Compose, and a React-based world editor.

## Repository Layout

```
NeoMud/
├── shared/     KMP module — models, protocol, shared between client and server
├── server/     Ktor 3.x + Netty — WebSocket game server, SQLite persistence
├── client/     Jetpack Compose — Android client with sprite rendering
├── maker/      React 18 + Express — web-based world editor (Vite dev server)
├── docs/       Screenshots and assets
└── scripts/    Utility scripts
```

## Build & Run

**Prerequisites**: JDK 21 (Corretto), Android SDK platform 34+, Node.js 18+

```bash
# Server
./gradlew :server:run          # Starts on :8080, WebSocket at /game, health at /health

# Client
./gradlew :client:installDebug  # Android emulator connects to 10.0.2.2:8080

# Maker
cd maker && npm install && npm run dev  # http://localhost:5173

# Tests
./gradlew :shared:jvmTest :server:test  # All server + shared tests
./gradlew :client:compileDebugKotlin    # Client compile check (no runtime tests yet)
```

**Gradle notes**: Configuration cache is enabled. `JAVA_HOME` must be set (configured in `settings.local.json` env).

**CRITICAL — World Bundle**: The server loads world data from `server/build/worlds/default-world.nmd`, built by `./gradlew packageWorld` from `maker/default_world_src/`. **Always run `./gradlew packageWorld --rerun-tasks`** after ANY change to `default_world_src/` (skills, items, zones, assets, etc.) and before running the server, running tests, or doing any verification. Gradle's UP-TO-DATE check for this task is unreliable — never trust it, always force with `--rerun-tasks`.

## Tooling Sanity (read this before debugging env)

- **If anything looks off** (PATH, MCP, "command not found", "playwright not loading"), run `./scripts/doctor.sh` first. It prints PASS/FAIL for every tool, env var, and MCP server. Do NOT start spelunking through `~/.claude/`, `~/.mcp.json`, etc. — they don't exist on this machine. The project-scoped `.mcp.json` (repo root) and `.claude/settings.local.json` are authoritative.
- **MCP servers**: declared in `.mcp.json` at the repo root. Auto-loaded because `enableAllProjectMcpServers: true` is in `.claude/settings.local.json`. To inspect runtime state: `claude mcp list`. After editing `.mcp.json`, the session must be restarted — there is no live reload.
- **Bash tool environment**: runs as a non-login non-interactive `bash`, but PATH is built from a shell snapshot that includes Homebrew, JAVA_HOME, and `~/.local/bin`. `npx`, `uvx`, `node`, `java`, `gradle` all resolve without absolute paths. Don't add absolute paths in scripts unless you actually need cross-shell portability.
- **Waiting on long-running scripts/CI**: use `Bash` with `run_in_background: true` plus the `Monitor` tool to stream output line-by-line — each new line wakes the model. Do **not** poll in a `sleep` loop, do not run `gh run watch` in the foreground (it eats the whole turn). `RemoteTrigger` spawns a *new* session and cannot resume the current one — don't try.
- **Secrets**: API keys live in `.claude/settings.local.json` env block. That file is gitignored (line 49 of `.gitignore`) and never tracked. Don't print key values to logs or copy them into committed files.

## Architecture Overview

### Protocol
- `shared/.../protocol/ClientMessage.kt` — all client-to-server messages (sealed class)
- `shared/.../protocol/ServerMessage.kt` — all server-to-client messages (sealed class)
- `shared/.../protocol/MessageSerializer.kt` — kotlinx.serialization with `classDiscriminator = "type"`
- Wire format is JSON over WebSocket. Both sides share the same Kotlin types at compile time.

### Server Core Loop
The server runs a **1.5-second tick-based game loop** (`GameLoop.kt`). Each tick:

1. **Grace period tick-down** — newly-arrived players get a brief combat grace window
2. **Cooldown tick-down** — per-skill cooldown counters decrement each tick
3. **Non-combat skill resolution** — meditate and track resolve before combat
4. **Combat resolution** (`CombatManager.processCombatTick()`) — for each player in attack mode, resolves in priority order:
   - Pending Bash → Pending Kick → Readied Spell (auto-cast) → Melee attack
5. **NPC behaviors** — wander, patrol, pursuit, attack (strategy pattern via `BehaviorNode`)
6. **Meditation regen** — MP restored for meditating players
7. **NPC spawning** — continuous spawn system per zone

### Command → Queue → Tick Pattern
**Commands are validation-only.** When a player sends a combat skill (bash, kick, spell, meditate, track), the corresponding command:
1. Validates prerequisites (cooldown, target exists, mana, etc.)
2. Queues the action on the session (`session.pendingSkill` or `session.readiedSpellId`)
3. Returns immediately — NO damage, NO kill handling, NO cooldown setting

The **game tick** resolves everything uniformly. This ensures:
- All kills flow through ONE handler in GameLoop (loot, XP, attack-mode-disable, broadcasts)
- No duplicate kill processing
- Consistent initiative ordering

### Key Server Types

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [roomsmith-games/NeoMud](https://github.com/roomsmith-games/NeoMud) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->
