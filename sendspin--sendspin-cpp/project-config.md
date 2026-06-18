---
trigger: always_on
description: Standalone C++ library implementing the Sendspin synchronized audio streaming protocol. Builds on both ESP-IDF (ESP32) and host platforms (macOS/Linux). Designed to be consumed by ESPHome but has no ESPHome dependencies.
---

# sendspin-cpp

Standalone C++ library implementing the Sendspin synchronized audio streaming protocol. Builds on both ESP-IDF (ESP32) and host platforms (macOS/Linux). Designed to be consumed by ESPHome but has no ESPHome dependencies.

## Architecture

The library provides `SendspinClient` as the main public API. It handles the full protocol lifecycle: WebSocket connections, time synchronization, audio decoding/sync, and message routing.

### Key classes

- `SendspinClient` (`client.h`): main orchestration class, owns connections, time sync, and message routing
- `PlayerRole` (`player_role.h`): audio streaming role, owns `SyncTask`, writes decoded audio via `on_audio_write` callback
- `ControllerRole` (`controller_role.h`): sends playback commands to the server
- `MetadataRole` (`metadata_role.h`): receives track metadata and progress
- `ArtworkRole` (`artwork_role.h`): receives album artwork images
- `VisualizerRole` (`visualizer_role.h`): receives spectrum/beat visualization data
- `ColorRole` (`color_role.h`): receives audio-derived RGB color palette from the server
- `SyncTask` (`sync_task.h`): decodes encoded audio, synchronizes to server timestamps, writes PCM via audio write callback
- `SendspinConnection` (`connection.h`): abstract WebSocket connection base
- `SendspinServerConnection` / `SendspinClientConnection`: platform-specific WebSocket transports (ESP uses `esp_websocket_client`/`esp_http_server`, host uses IXWebSocket)
- `SendspinTimeFilter` (`time_filter.h`): 2D Kalman filter for NTP-style time sync
- `SendspinTimeBurst` (`time_burst.h`): burst-based time message coordinator
- `SendspinDecoder` (`decoder.h`): FLAC/Opus/PCM decoder wrapper

### Role composition

Roles are added to the client at runtime via `add_player()`, `add_metadata()`, etc. Each role receives a `SendspinClient*` at construction time and uses it to access shared services (time sync, state publishing, message sending). The consumer provides behavior by implementing listener interfaces (`PlayerRoleListener`, `MetadataRoleListener`, etc.) and setting them via `set_listener()`. Required callbacks are pure virtual; optional callbacks have default no-op implementations. The client dispatches messages to roles via null-pointer checks on role pointers.

Roles can be disabled at compile time via `SENDSPIN_ENABLE_*` cmake options (host build) or Kconfig entries (ESP-IDF build). When a role is disabled, its source files are not compiled and its `add_*()` declaration, accessor, and `unique_ptr` member are removed from `client.h`. `#ifdef` guards live in exactly two places: `cmake/sources.cmake` (source lists) and `include/sendspin/client.h` / `src/client.cpp` (dispatch points).

```cpp
// Implement listener interfaces
struct MyPlayerListener : PlayerRoleListener {
    size_t on_audio_write(uint8_t* data, size_t len, uint32_t timeout_ms) override {
        return audio_output.write(data, len, timeout_ms);
    }
    void on_stream_start() override { /* ... */ }
};

struct MyMetadataListener : MetadataRoleListener {
    void on_metadata(const ServerMetadataStateObject& m) override { /* ... */ }
};

struct MyNetworkProvider : SendspinNetworkProvider {
    bool is_network_ready() override { return true; }
};

MyPlayerListener player_listener;
MyMetadataListener metadata_listener;
MyNetworkProvider network_provider;

SendspinClient client(config);
auto& player = client.add_player(player_config);
player.set_listener(&player_listener);
auto& metadata = client.add_metadata();
metadata.set_listener(&metadata_listener);
client.set_network_provider(&network_provider);
client.add_controller();
client.start_server();
```

### Platform integration

The platform (e.g., ESPHome) provides:

- A `PlayerRoleListener` implementation with `on_audio_write()` to receive decoded PCM audio
- An optional `SendspinPersistenceProvider` for saving/loading preferences
- A `SendspinNetworkProvider` for network readiness
- An optional `SendspinClientListener` for high-performance WiFi power management callbacks
- Playback progress feedback via `notify_audio_played()`

## Project layout

```text
include/sendspin/     - Public API headers (client.h, config.h, types.h, *_role.h)
src/                        - Cross-platform source files (.cpp) and private headers (.h)
src/platform/               - Platform abstraction headers and host-only source files
src/esp/                    - ESP-IDF networking implementations and headers
src/host/                   - Host (IXWebSocket) networking implementations and headers
cmake/                      - CMake modules (sources.cmake, host.cmake)
examples/common/            - Shared PortAudio audio sink used by host examples
examples/basic_client/      - Standalone host example with PortAudio audio output
examples/tui_client/        - Terminal UI host example with PortAudio audio output
```

### Header visibility


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Sendspin/sendspin-cpp](https://github.com/Sendspin/sendspin-cpp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
