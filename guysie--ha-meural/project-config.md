---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

HA-meural is a Home Assistant custom component that integrates NETGEAR Meural Canvas digital art frames. It provides media player entities with support for controlling artwork display, playlists, brightness, and various Canvas settings through both the Meural cloud API and local device interface.

**Repository**: https://github.com/GuySie/ha-meural

## Validation and Testing

### Validating the Integration

Run Home Assistant's hassfest validation (GitHub Actions will run this automatically on push):
```bash
# This validation runs via GitHub Actions (.github/workflows/hassfest.yaml)
# No local test suite exists
```

### Manual Testing

To test changes, install the integration in a Home Assistant instance:
1. Copy `custom_components/meural` to your Home Assistant's `custom_components` directory
2. Restart Home Assistant
3. Add the Meural integration via UI (*Settings* → *Devices & Services* → *Add Integration*)

## Architecture

### Dual Coordinator Pattern (v2.0.0+)

The integration uses two DataUpdateCoordinators for efficient polling:

**CloudDataUpdateCoordinator** (`coordinator.py:26-160`):
- Polls Meural cloud API every 60 seconds (device settings only), or 3600 seconds (1 hour) when all devices are sleeping
- Gallery data fetched separately via `async_refresh_galleries()` every 30 minutes (`GALLERY_UPDATE_INTERVAL`)
- Gallery refresh triggered synchronously at startup (in `__init__.py` after `async_config_entry_first_refresh()`), as a background task on regular poll when stale, lazily on media browser open, and after `synchronize()` service
- Handles authentication errors and triggers reauth flow
- Shared across all devices for a single account
- Aggregates sleep state across all entities to determine polling interval

**LocalDataUpdateCoordinator** (`coordinator.py:163-309`):
- Polls local device API every 10 seconds
- Each device has its own local coordinator instance
- Fetches real-time state: sleep status, local galleries, gallery status, gsensor orientation, lux, free space, WiFi signal
- When device is sleeping, skips gallery fetches but still polls `send_get_system()` so sensor data (lux, free space, WiFi signal) continues to update every 10 seconds
- Gracefully handles offline devices without failing the integration
- Preserves last known sleep state on transient connection failures (no flickering)
- Returns cached data when device is unreachable

### Core Components

**PyMeural** (`pymeural.py`):
- Cloud API client for Meural's REST API (https://api.meural.com/v0/)
- Uses AWS Cognito (boto3) for authentication with automatic token refresh
- Handles authentication token lifecycle with callback for persistent storage
- All API methods are async and use aiohttp
- Classifies Cognito auth failures by error code: `NotAuthorizedException`/`UserNotFoundException` raise `InvalidAuth` (genuinely bad credentials); anything else (WAF blocks, throttling, network errors) raises `CannotConnect` so it doesn't misreport as bad credentials
- On auth failure, applies exponential backoff (60s, doubling up to a 30 min cap) before allowing another full authentication attempt, to avoid hammering a blocked/rate-limited auth endpoint
- Backoff state is keyed by account email in module-level state (`_AUTH_BACKOFF_STATE`), not on the `PyMeural` instance, since Home Assistant recreates the client on every `ConfigEntryNotReady` setup retry; resets on successful auth or full HA restart

**LocalMeural** (`pymeural.py`):
- Local device API client for Canvas web server (http://DEVICE-IP/remote/)
- Controls device directly without cloud dependency
- Handles device sleep/wake detection

**MeuralBacklightLight** (`light.py`):
- Light entity controlling the Canvas backlight brightness
- Turning off suspends the Canvas device (equivalent to media player turn off)
- Turning on wakes the Canvas device (equivalent to media player turn on)
- Uses optimistic state updates for on/off; brightness changes apply immediately
- Stays in sync with the media player entity — both reflect the same sleep/wake state

**MeuralSensorEntities** (`sensor.py`):
- **Ambient Light** (`MeuralLuxSensor`): illuminance in lux from local API; enabled by default; useful for automations
- **Free Space** (`MeuralFreeSpaceSensor`): available Canvas storage in MB from local API; diagnostic; disabled by default
- **WiFi Signal** (`MeuralWifiSignalSensor`): WiFi signal strength in dBm from local API; diagnostic; disabled by default
- **Last Seen by Cloud** (`MeuralLastSeenSensor`): timestamp of last cloud contact from cloud API; diagnostic; disabled by default

**MeuralEntity** (`media_player.py`):
- Media player entity implementing standard Home Assistant media player features
- Coordinates between cloud and local data sources
- Registers custom services (set_brightness, preview_image, set_device_option, etc.)
- Detects physical device rotation via gsensor changes when orientationMatch is enabled
- Reloads current gallery after orientation change to force `current_item` update (local API limitation)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GuySie/ha-meural](https://github.com/GuySie/ha-meural) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
