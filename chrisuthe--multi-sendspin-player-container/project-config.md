---
trigger: always_on
description: > This file provides context for Claude Code and other AI agents working on this project.
---

# CLAUDE.md - AI Agent Configuration

> This file provides context for Claude Code and other AI agents working on this project.

## Project Overview

**Multi-Room Audio Controller** - A C# ASP.NET Core 8.0 application for managing Sendspin audio players. Enables whole-home audio with USB DACs connected to a central server.

### Purpose

Transform a single Docker host with multiple USB audio devices into a multi-room audio system. Each audio zone gets its own player that streams from Music Assistant via the Sendspin protocol.

### Key Users

- Home automation enthusiasts with multi-room audio setups
- Music Assistant users wanting additional audio endpoints
- Docker/NAS users looking for centralized audio management

---

## Reference Documentation

- **Home Assistant Add-on Development:**
  - [Add-on Configuration](https://developers.home-assistant.io/docs/add-ons/configuration/) - config.yaml schema
  - [Add-on Communication](https://developers.home-assistant.io/docs/add-ons/communication/) - Ingress, supervisor
  - [Add-on Publishing](https://developers.home-assistant.io/docs/add-ons/publishing/) - Repository setup
- **Reference Add-ons:**
  - [home-assistant/addons/vlc](https://github.com/home-assistant/addons/tree/master/vlc) - Official VLC add-on (audio player pattern)
- **SDK Documentation:**
  - SendSpin.SDK v7.3.0 - Sendspin protocol handling

---

## Architecture

```
ASP.NET Core 8.0 Application
├── Controllers/                  # REST API endpoints
│   ├── PlayersEndpoint.cs       # /api/players CRUD
│   ├── DevicesEndpoint.cs       # /api/devices
│   ├── ProvidersEndpoint.cs     # /api/providers
│   ├── TriggersEndpoint.cs      # /api/triggers (12V relay control)
│   └── HealthEndpoint.cs        # /api/health
├── Services/
│   ├── PlayerManagerService.cs   # SDK player lifecycle
│   ├── ConfigurationService.cs   # YAML persistence
│   ├── TriggerService.cs        # Relay board management, player↔relay mapping
│   └── EnvironmentService.cs     # Docker vs HAOS detection
├── Relay/                        # 12V trigger hardware abstraction
│   ├── IRelayBoard.cs           # Common relay board interface
│   ├── HidRelayBoard.cs         # USB HID relay boards (DCT Tech)
│   ├── FtdiRelayBoard.cs        # FTDI relay boards (Denkovi + generic)
│   └── MockRelayBoard.cs        # Mock board for testing
├── Audio/                        # Audio output layer
│   ├── BufferedAudioSampleSource.cs  # Bridges timed buffer to audio output
│   ├── PulseAudio/              # PulseAudio backend (primary)
│   └── Alsa/                    # ALSA backend (Docker fallback)
├── Utilities/
│   ├── ClientIdGenerator.cs     # MD5-based IDs
│   └── AlsaCommandRunner.cs     # Volume control
├── Models/
│   ├── TriggerModels.cs         # Trigger/relay data models
│   └── ...                      # Other request/response types
├── wwwroot/                      # Static web UI
└── Program.cs                    # Entry point
```

### Key Files to Understand First

1. `src/MultiRoomAudio/Program.cs` - Entry point, DI setup
2. `src/MultiRoomAudio/Services/PlayerManagerService.cs` - Core player management
3. `src/MultiRoomAudio/Services/ConfigurationService.cs` - YAML config persistence
4. `src/MultiRoomAudio/Services/EnvironmentService.cs` - HAOS vs Docker detection

---

## Development Commands

```bash
# Restore dependencies
dotnet restore src/MultiRoomAudio/MultiRoomAudio.csproj

# Build project
dotnet build src/MultiRoomAudio/MultiRoomAudio.csproj

# Run locally (Windows/macOS - audio won't work)
dotnet run --project src/MultiRoomAudio/MultiRoomAudio.csproj

# Build Docker image
docker build -f docker/Dockerfile -t multiroom-audio .

# Run with Docker (Linux with audio)
docker run -d --name multiroom \
  -p 8096:8096 \
  --device /dev/snd \
  -v $(pwd)/config:/app/config \
  multiroom-audio

# Build for HAOS
docker build -f docker/Dockerfile \
  --platform linux/amd64,linux/arm64 \
  -t ghcr.io/chrisuthe/multiroom-audio-hassio .
```

---

## Testing Guidelines

When testing locally on macOS (where PulseAudio is not available):

1. **Always use mock hardware mode**: Run with `MOCK_HARDWARE=true` to get simulated audio devices

   ```bash
   MOCK_HARDWARE=true dotnet run --project src/MultiRoomAudio/MultiRoomAudio.csproj
   ```

2. **Check for existing instances**: Before starting the app, check if an old instance is running

   ```bash
   pgrep -f "MultiRoomAudio"
   ```

3. **Kill only MultiRoomAudio processes**: When stopping, target only the specific process

   ```bash
   pkill -f "MultiRoomAudio"
   ```

4. **Always run the app for user testing**: Before committing changes, start the app so the user can test interactively

5. **Verify process state changes**: When starting or stopping the app:
   - Wait **5 seconds** after starting before checking if it spawned
   - Wait **5 seconds** after killing before checking if it's gone
   - Use `pgrep -f "MultiRoomAudio"` to verify the process state
   - If the expected state isn't found, **retry the check once** before reporting failure

6. **Clear test data between runs**: Remove temporary config files created during testing

   ```bash
   rm -f src/MultiRoomAudio/config/sinks.yaml
   rm -f src/MultiRoomAudio/config/players.yaml  # if needed

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chrisuthe/Multi-SendSpin-Player-Container](https://github.com/chrisuthe/Multi-SendSpin-Player-Container) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
