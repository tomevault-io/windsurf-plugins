---
trigger: always_on
description: MCP server for controlling DAWs (Bitwig Studio, Ableton Live) from Claude.
---

# DAW MCP Project

MCP server for controlling DAWs (Bitwig Studio, Ableton Live) from Claude.

## Scope: Session View / Clip Launcher Only

**This project intentionally targets only the clip launcher (Bitwig) / session view (Ableton) paradigm.** Arrangement view is out of scope.

**Rationale:**
- Clip launcher provides discrete, addressable units (track × slot) - ideal for AI-driven workflows
- Arrangement view has continuous timeline addressing - fragile, less intuitive for AI
- Bitwig's Control Surface API doesn't expose arrangement clips at all
- Ableton's API has arrangement access, but asymmetric support across DAWs is undesirable
- The user arranges clips into songs - that's human creative territory

**Implications:**
- No arrangement clip creation, reading, or note manipulation
- No Reaper support (Reaper lacks a clip launcher paradigm entirely)
- MIDI/OSC alternatives were considered but rejected - not worth the complexity

**Workflow:** AI generates clips in launcher slots → User arranges them into songs on timeline.

## Project Structure

- `bitwig-extension/` - Java extension for Bitwig Studio (TCP server on port 8181)
- `ableton-extension/` - Python MIDI Remote Script for Ableton Live (port 8182)
- `mcp-server/` - TypeScript MCP server that bridges Claude to DAW extensions

## Building (Development)

```bash
# Bitwig extension
cd bitwig-extension && gradle build && gradle copyExtension

# MCP server
cd mcp-server && npm install && npm run build
```

## Release Build

Creates a distributable ZIP with all components:

```bash
./scripts/release.sh 1.0.0
```

**Output:** `release/daw-mcp-1.0.0.zip` (~325KB)

**Contents:**
- `BitwigMCP.bwextension` - Java extension (cross-platform, single file)
- `AbletonMCP/` - Python Remote Scripts (cross-platform)
- `mcp-server.js` - Bundled MCP server (~215KB, requires Node.js)
- `config.example.json` - Example configuration
- `README.md` - Installation instructions

**How it works:**
1. Builds Bitwig extension via Gradle
2. Bundles MCP server with esbuild into single JS file
3. Copies Ableton Python scripts
4. Creates ZIP archive

**User installation:**
1. Extract ZIP
2. Copy `BitwigMCP.bwextension` to Bitwig extensions folder
3. Copy `AbletonMCP/` to Ableton Remote Scripts folder
4. Add to Claude config:
   ```json
   {
     "mcpServers": {
       "daw": {
         "command": "node",
         "args": ["/path/to/mcp-server.js"]
       }
     }
   }
   ```

## Architecture

```
Claude <-> MCP Server (stdio) <-> TCP <-> DAW Extension <-> DAW API
                                   │
                                   ├── :8181 Bitwig (Java)
                                   └── :8182 Ableton (Python)
```

## Key Files

### Bitwig Extension (Java)

| File | Purpose |
|------|---------|
| `bitwig-extension/src/main/java/com/pxaudio/bitwigmcp/BitwigMCPExtension.java` | Main extension entry point, creates API objects |
| `bitwig-extension/src/main/java/com/pxaudio/bitwigmcp/server/MCPServer.java` | TCP server, JSON-RPC handling |
| `bitwig-extension/src/main/java/com/pxaudio/bitwigmcp/handlers/ClipHandler.java` | MIDI note read/write operations |
| `bitwig-extension/src/main/java/com/pxaudio/bitwigmcp/handlers/CommandDispatcher.java` | Routes commands to handlers |
| `bitwig-extension/.../config/ConfigReader.java` | Java config file loading |

### Ableton Extension (Python)

| File | Purpose |
|------|---------|
| `ableton-extension/__init__.py` | Entry point for Live Remote Script |
| `ableton-extension/manager.py` | ControlSurface subclass, tick scheduler |
| `ableton-extension/tcp_server.py` | Non-blocking TCP server |
| `ableton-extension/handlers/clip.py` | MIDI note operations |

**Linux Development:** Ableton Live 12 runs via Lutris in Wine prefix at `/home/pta/Games/ableton`. Note: Wine doesn't follow symlinks, so files must be copied:
```bash
cp -r /home/pta/Develop/audio/daw-mcp/ableton-extension/* \
  /home/pta/Games/ableton/drive_c/users/pta/Documents/Ableton/User\ Library/Remote\ Scripts/AbletonMCP/
```

### MCP Server (TypeScript)

| File | Purpose |
|------|---------|
| `mcp-server/src/index.ts` | MCP tool definitions and command mapping |
| `mcp-server/src/daw-client.ts` | TCP client for DAW communication (lazy connections) |
| `mcp-server/src/config.ts` | Configuration file loading |
| `mcp-server/src/music-analysis.ts` | Tonal.js chord/scale/key detection |
| `mcp-server/src/euclidean.ts` | Euclidean rhythm generation (uses Tonal.js RhythmPattern) |

## Configuration

Both extensions and the MCP server read from a shared config file:

| Platform | Path |
|----------|------|
| Linux | `~/.config/daw-mcp/config.json` |
| macOS | `~/Library/Application Support/daw-mcp/config.json` |
| Windows | `%APPDATA%\daw-mcp\config.json` |

**Example config:**
```json
{
  "defaultDaw": "bitwig",
  "gridResolution": 16,
  "bitwig": {
    "port": 8181,
    "cursorClipLengthBeats": 128,
    "scenes": 128
  },
  "ableton": {
    "port": 8182
  },
  "mcp": {
    "selectionDelayMs": 400,
    "requestTimeoutMs": 10000
  },
  "tools": {
    "transpose_clip": true,
    "batch_operations": false
  }
}
```

### Grid Resolution

The global `gridResolution` setting affects both DAWs:

| gridResolution | stepSize | Musical Value |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ptaczek/daw-mcp](https://github.com/ptaczek/daw-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
