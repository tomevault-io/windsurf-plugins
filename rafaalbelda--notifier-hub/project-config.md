---
trigger: always_on
description: This is a **Home Assistant custom integration** that provides a centralized notification hub supporting multiple channels: text notifications, Alexa TTS, Google/Cast TTS, phone calls, Home Assistant lifecycle notices, persistent notifications, and Auto Volume controls.
---

# Notifier Hub Integration - AI Agent Guidelines

This is a **Home Assistant custom integration** that provides a centralized notification hub supporting multiple channels: text notifications, Alexa TTS, Google/Cast TTS, phone calls, Home Assistant lifecycle notices, persistent notifications, and Auto Volume controls.

## Quick Architecture

```
NotifierHub (coordinator)
├── NotificationManager (text/persistent messages)
├── AlexaManager (TTS + media playback)
├── GoogleManager (TTS + media playback)
└── PhoneManager (voice calls)
```

**Entry Point**: [__init__.py](custom_components/notifier_hub/__init__.py) initializes the hub, registers services/events, creates runtime entities, installs the optional dashboard, and coordinates message dispatch.

## Key Patterns & Conventions

### Manager Pattern
- **AlexaManager** and **GoogleManager** use `asyncio.Queue` for sequential TTS processing (prevents overlapping playback)
- **Worker Pattern**: Each manager has an async task consuming queue items one at a time
- **Volume Management**: Saves current volume → plays TTS → restores original volume
- **Task Cleanup**: Always cancel tasks on unload with proper error handling

### Text Processing
- Use helper functions from [helpers.py](custom_components/notifier_hub/helpers.py):
  - `check_bool()` - Convert string/value to boolean
  - `return_list()` - Convert comma-separated strings/lists/tuples to list
  - `remove_tags()` - Strip HTML/SSML tags for TTS
  - `has_numbers()` - Detect time/large numbers for duration estimation
  - `estimate_speech_duration()` - Calculate TTS playback time (formula: words × 0.42 + adjustments)
- Service-specific cleanup: Telegram needs captions, Pushover needs URLs, Discord needs embeds

### Configuration
- **Merged Config**: Combines `entry.data` (setup) + `entry.options` (runtime updates)
- **Runtime Updates**: `notifier_hub.set_config` and editable entities update config entry options
- **Constants**: All magic strings in [const.py](custom_components/notifier_hub/const.py)
- **Validation**: Voluptuous schemas with sensible defaults
- **Config Flow**: [config_flow.py](custom_components/notifier_hub/config_flow.py) uses entity selectors for media players
- **Dashboard Install**: `install_dashboard` copies the bundled Lovelace YAML to `/config/notifier_hub_dashboard.yaml`

### State-Based Routing
Messages are routed through state checks:
- **Location**: Prefer configured `persons`; fall back to `location_tracker` entity (can suppress text and speech notifications)
- **Speech Home Only**: Adds an implicit `location: home` check to speech channels when enabled
- **DND**: Blocks TTS and phone if `dnd_entity` is "on"
- **Guest Mode**: Overrides location check if enabled
- **Priority**: Bypasses all toggles if `priority_message_entity` is "on"
- **Toggles**: `text_notifications`, `screen_notifications`, `speech_notifications`, `alexa_notifications`, `google_notifications`, `phone_notifications`, `ha_event_notifications`, `auto_volume`

### Auto Volume
- Auto Volume uses editable `time.*` period starts and `number.*` period volumes
- Period state is exposed by `sensor.notifier_hub_day_period` and `sensor.notifier_hub_day_period_volume`
- Explicit per-message `alexa.volume` / `google.volume` takes precedence over Auto Volume
- Respect `auto_volume_exclude_players` before changing player volumes

## Common Development Tasks

### Adding a New Notification Channel
1. Create a new manager class in `custom_components/notifier_hub/` extending the manager pattern
2. Initialize it in [__init__.py](custom_components/notifier_hub/__init__.py) during setup
3. Add config options to [const.py](custom_components/notifier_hub/const.py)
4. Update config schema in [__init__.py](custom_components/notifier_hub/__init__.py)
5. Update [services.yaml](custom_components/notifier_hub/services.yaml) with new service fields
6. Add routing logic to `NotifierHub.dispatch()` method

### Modifying TTS Processing
- **Alexa**: [alexa_manager.py](custom_components/notifier_hub/alexa_manager.py) handles SSML generation with voice, prosody, language tagging, speechcons
- **Google**: [google_manager.py](custom_components/notifier_hub/google_manager.py) routes to configurable `tts.*` service
- **Player Resolution**: Both managers support entity IDs, friendly names, groups, and sensor values
- **Speech Duration**: [helpers.py](custom_components/notifier_hub/helpers.py) `estimate_speech_duration()` used for timing

### Updating Text Notification Services
- [notification_manager.py](custom_components/notifier_hub/notification_manager.py) handles dispatch to `notify.*` services
- Each service has a payload builder: Telegram (photos), Pushover (priority), Discord (embeds), mobile_app (TTS injection), generic
- Text substitution chains apply regex replacements before sending (see `SUB_NOWRAP`, `SUB_WRAP`)

### Adding Entities
- Extend `NotifierHubEntity` from [entity.py](custom_components/notifier_hub/entity.py)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rafaalbelda/notifier_hub](https://github.com/rafaalbelda/notifier_hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
