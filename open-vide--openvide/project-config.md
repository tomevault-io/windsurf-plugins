---
trigger: always_on
description: React Native mobile app (Expo SDK 52, React 18.3, RN 0.76) managing AI CLI tools
---

# CLAUDE.md

## Project
React Native mobile app (Expo SDK 52, React 18.3, RN 0.76) managing AI CLI tools
(Claude Code, OpenAI Codex, Google Gemini CLI) on remote machines over SSH.

## How It Works: App ↔ Daemon Relay

The app does not talk to CLI tools directly. A lightweight **daemon** (`openvide-daemon`)
runs persistently on each remote machine, acting as a relay between the mobile app and
the AI CLI tools. This decouples the app from the SSH connection — sessions survive
network drops, app backgrounding, and reconnections.

### Data Flow
```
Mobile App → SSH exec → openvide-daemon CLI → Unix socket IPC → Daemon process
                                                                   ↓
                                                             spawns CLI tool
                                                          (claude / codex / gemini)
                                                                   ↓
                                                          stdout/stderr captured
                                                          line-by-line → output.jsonl
                                                                   ↓
App ← SSH stdout ← openvide-daemon session stream --follow ← tails output.jsonl
```

### Daemon (service/)
- **Zero-dependency** Node.js process, installed globally via `npm install -g openvide-daemon`.
- Self-daemonizes: the CLI auto-starts the daemon on first use (`ensureDaemon()`).
- Lives at `~/.openvide-daemon/` on the remote machine:
  - `daemon.pid` — PID file, touched every 30s as heartbeat (stale after 60s).
  - `daemon.sock` — Unix domain socket for newline-delimited JSON IPC.
  - `daemon.log` — daemon stdout/stderr.
  - `state.json` — all session records (atomic writes via rename).
  - `sessions/<id>/output.jsonl` — captured CLI output per session.
- **No HTTP server or open ports.** All app→daemon communication is via CLI commands
  executed over SSH. The daemon listens only on a local Unix socket.

### IPC Commands
| Command | Description |
|---------|-------------|
| `session.create` | Create session record + output dir |
| `session.send` | Spawn CLI tool process, begin capturing output |
| `session.stream --follow` | Tail output.jsonl via fs.watch (bypasses IPC, reads file directly) |
| `session.cancel` | SIGINT → 3s grace → SIGTERM |
| `session.get/list/remove` | CRUD on session records |
| `health` | PID, session counts |
| `stop` | Graceful shutdown (SIGTERM children → SIGKILL after 5s) |

### Output Format (output.jsonl)
Each line is one of three types:
- `{ t: "o", ts, line }` — stdout line from CLI tool (raw JSON)
- `{ t: "e", ts, line }` — stderr line
- `{ t: "m", ts, event, ... }` — meta: `turn_start`, `turn_end` (with exitCode), `error`

### App-Side Transport (DaemonTransport.ts)
Runs `openvide-daemon <cmd>` over SSH via `NativeSshClient.runCommand()`.
Tracks `daemonOutputOffset` per session so incremental reads never miss or
duplicate output. For streaming, runs `session stream --follow` over SSH and
parses each JSONL line through the tool-specific adapter.

### Session Lifecycle
1. **Install**: App runs `npm install -g openvide-daemon` over SSH (HostDetailScreen).
2. **Detect**: App runs `openvide-daemon version` during CLI detection (cliDetection.ts).
3. **Create**: `DaemonTransport.createSession()` → daemon creates record + output dir.
4. **Turn**: `DaemonTransport.sendTurn()` → daemon spawns CLI tool, captures output.
5. **Stream**: `DaemonTransport.streamOutput()` → tails output.jsonl over SSH, feeds
   lines to adapter → `CliStreamEvent[]` → `SessionEngine.processEvent()` → UI.
6. **Multi-turn**: `conversationId` extracted from CLI output (Claude: `session_id`,
   Codex: `thread_id`). Passed as `--resume` / `exec resume` on subsequent turns.
   Gemini injects history into prompt via `<previous_conversation>` tags.
7. **Reconnect**: `importDaemonSessions()` lists daemon sessions, imports any
   untracked ones with their current `outputLines` offset.

### Command Building (commandBuilder.ts)
- **Claude**: `claude -p '<prompt>' --output-format stream-json --verbose [--resume <id>]`
- **Codex**: `codex exec '<prompt>' --json --full-auto` (or `codex exec resume '<id>' '<prompt>'`)
- **Gemini**: `gemini -p '<prompt>' --output-format json -y`

### Environment (processRunner.ts)
Spawned CLI processes get an augmented PATH (`~/.local/bin`, `~/.cargo/bin`,
`~/.bun/bin`, `/opt/homebrew/bin`, etc.) and `CLAUDECODE` is removed from env
to prevent Claude Code from refusing to start inside the daemon's process tree.

## Architecture (App)
- **Styling**: NativeWind 4.1 (Tailwind CSS) — all UI uses className with cn() utility
  from src/lib/utils.ts (clsx + tailwind-merge). Colors defined in tailwind.config.js.
  Programmatic colors via src/constants/colors.ts for Navigation API / prop values.
- **State**: React Context (AppStoreContext.tsx) + useAppStore(). Persisted via
  AsyncStorage (non-sensitive) + expo-secure-store (SSH credentials).
  Schema versioned (PersistedState.version) with migration in storage.ts.
- **SSH**: NativeSshClient wraps @dylankenneally/react-native-ssh-sftp. Persistent
  shell per target, marker-based command completion detection.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [open-vide/openvide](https://github.com/open-vide/openvide) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
