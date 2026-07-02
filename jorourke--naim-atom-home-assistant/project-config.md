---
trigger: always_on
description: This is a **HACS (Home Assistant Community Store)** custom integration that provides local network control of **Naim audio devices** (primarily Naim Atom). The integration creates a fully-featured media player entity in Home Assistant with real-time status updates.
---

# Naim Atom Home Assistant Integration

## Project Overview

This is a **HACS (Home Assistant Community Store)** custom integration that provides local network control of **Naim audio devices** (primarily Naim Atom). The integration creates a fully-featured media player entity in Home Assistant with real-time status updates.

**Version:** 0.2.0
**Type:** Local IoT integration (no cloud dependency)
**Communication:** Dual protocol (HTTP API + WebSocket)

## Architecture

```
Home Assistant Media Player Entity
         ↓
    NaimPlayer (media_player.py)
         ↓
    ┌────────────┬────────────┐
    ↓            ↓            ↓
HTTP Client  WebSocket   State Manager
(15081)      (4545)      (thread-safe)
    ↓            ↓            ↓
        Naim Atom Device
```

### Core Components

- **`media_player.py`** (690 lines) - Main entity implementation with debounce logic
- **`client.py`** (241 lines) - HTTP API client + WebSocket client
- **`websocket.py`** (89 lines) - Low-level TCP socket wrapper
- **`config_flow.py`** (140 lines) - UI configuration flow
- **`const.py`** - Configuration constants (ports, intervals, steps)
- **`exceptions.py`** - Custom exception hierarchy

### Communication Protocols

#### HTTP API (Port 15081)
Used for **control commands** with immediate feedback:
- Power: `PUT /power?system=on|lona`
- Volume: `PUT /levels/room?volume=0-100&mute=0|1`
- Playback: `GET /nowplaying?cmd=playpause|next|prev|seek`
- Source: `GET /inputs/{input}?cmd=select`

#### WebSocket (Port 4545)
Used for **real-time status updates**:
- Persistent TCP connection
- Line-delimited JSON messages
- Auto-reconnect with exponential backoff
- Incremental JSON parsing for streaming data

## Code Patterns & Conventions

### Async/Await Everywhere
- All I/O operations are async
- Use `asyncio.Lock()` for thread-safe state updates
- Use `async_get_clientsession()` for HTTP requests

### Debounce Mechanism (Critical)
```python
# Prevent feedback loops between UI actions and WebSocket updates
_last_user_volume_action: float  # 2 second window
_last_user_mute_action: float    # 2 second window
_debounce_timeout: float = 2.0   # Ignore device echo
_update_debounce_timeout: float = 1.0  # Rate limit polling
```

**Why:** When user changes volume via UI → HTTP command → device updates → WebSocket echoes change → UI would flicker. Debounce prevents this.

### Error Handling Pattern
```python
try:
    await self._api_client.set_value(...)
except aiohttp.ClientError as error:
    _LOGGER.error("Error: %s", error)
    # Revert optimistic state update
    await self._state.update(old_value)
```

### Retry Logic
```python
for retry in range(max_retries):
    try:
        return await operation()
    except Exception:
        if retry < max_retries - 1:
            await asyncio.sleep(2**retry)  # Exponential backoff
```

### State Management
```python
class NaimPlayerState:
    _lock: asyncio.Lock  # Thread-safe updates

    async def update(self, **kwargs):
        async with self._lock:
            # Update state atomically
```

## Development Workflows

### Adding New Features

1. **Research existing patterns** - Read similar features first
2. **Update client.py** - Add API methods if needed
3. **Update media_player.py** - Implement entity methods
4. **Add tests** - Update test files
5. **Test manually** - Use Home Assistant dev environment

### Fixing Bugs

1. **Check debounce logic** - Most UI issues are timing-related
2. **Check WebSocket parsing** - JSON streaming can be fragile
3. **Check state locking** - Race conditions cause flicker
4. **Add logging** - DEBUG level logs help diagnose issues

### Testing

**Run tests:**
```bash
uv run pytest tests/
```

**Check formatting:**
```bash
uv run ruff format --check custom_components/ tests/
```

**Fix formatting:**
```bash
uv run ruff format custom_components/ tests/
```

**Run linter:**
```bash
uv run ruff check custom_components/ tests/
```

**Test structure:**
- `test_client.py` - API client and WebSocket tests
- `test_media_player.py` - Entity behavior tests
- `test_config_flow.py` - Configuration UI tests
- `conftest.py` - Shared fixtures

**Mocking:**
- Use `aioresponses` for HTTP mocking
- Use `AsyncMock` for async methods
- Mock WebSocket connections for integration tests

## Common Tasks

### Adding a New Source

1. Update `SOURCE_TO_INPUT_MAP` in media_player.py:
```python
SOURCE_TO_INPUT_MAP = {
    "New Source": "newsource",  # Add here
    # ...
}
```

2. Add to config flow if needed (config_flow.py)

### Changing Debounce Timing

Edit constants in media_player.py:
```python
_debounce_timeout: float = 2.0  # User action ignore window
_update_debounce_timeout: float = 1.0  # Polling rate limit
```

### Adding New Device Commands

1. Add method to `NaimApiClient` in client.py
2. Add corresponding method to `NaimPlayer` entity
3. Update supported features flags if needed

## Configuration Constants

```python
DOMAIN = "naim_media_player"
DEFAULT_PORT = 4545           # WebSocket
DEFAULT_HTTP_PORT = 15081     # HTTP API

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jorourke/naim-atom-home-assistant](https://github.com/jorourke/naim-atom-home-assistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-02 -->
