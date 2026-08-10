---
trigger: always_on
description: aioswitcher is a Python 3.12+ async library for integrating with Switcher smart switches and outlets (power plugs, wall switches, curtain motor controllers, thermostat controllers, water heater controllers). It uses `uv` for dependency management and `pytest` with strict asyncio mode for testing.
---

# AGENTS.md — aioswitcher

## Project

aioswitcher is a Python 3.12+ async library for integrating with Switcher smart switches and outlets (power plugs, wall switches, curtain motor controllers, thermostat controllers, water heater controllers). It uses `uv` for dependency management and `pytest` with strict asyncio mode for testing.

## AI Policy

This project has an [AI policy](AI_POLICY.md). Always read it and ensure all suggestions, code, and contributions comply. If any behavior seems to conflict with the policy, warn the user and ask for guidance.

## Client Projects

aioswitcher is a dependency for two major projects — always consider them when making changes, especially breaking ones.

1. **Home Assistant ([switcher_kis integration](https://www.home-assistant.io/integrations/switcher_kis/))** — the primary client. It uses the Bridge for device discovery and the API for control. The integration is tightly coupled to this library's device dataclasses, state responses, and error semantics.
   - [GitHub source](https://github.com/home-assistant/core/tree/dev/homeassistant/components/switcher_kis)
2. **[switcher_webapi](https://switcher-webapi.figenblat.com/)** — a containerized REST API wrapper that exposes device control (TCP only) as HTTP endpoints. It enables non-Python users and systems to control Switcher devices without the UDP discovery layer. This client depends on `SwitcherApi` and the message/response types.
   - [GitHub source](https://github.com/TomerFi/switcher_webapi)

When making API changes or renaming things, check if they impact these consumers. Do not break the public interface without coordinating with both clients.

## Repo Structure

```text
src/aioswitcher/
  __init__.py          # package entry, exports api, bridge, device, schedule
  bridge.py            # UDP broadcast discovery — SwitcherBridge + DatagramParser
  api/
    __init__.py        # SwitcherApi — TCP socket-based device control
    messages.py        # Response parsers (login, state, schedules)
    packets.py         # Binary protocol packet templates (hex strings)
    remotes.py         # Breeze IR remote database and command builders
  device/
    __init__.py        # DeviceType enum, all dataclasses (SwitcherPowerPlug, etc.)
    tools.py           # Helpers: hex encoding, CRC signing, timestamp, watts→amps
  schedule/
    __init__.py        # Days enum, ScheduleState enum
    parser.py          # Schedule data parser
    tools.py           # Time/weekday helpers
tests/                 # pytest suite (strict asyncio mode)
scripts/               # CLI helper scripts (discover, control, validate_token, get_login_key)
docs/                  # mkdocs documentation
```

## Scripts

CLI tools available only from the repository — they are not bundled with the `aioswitcher` PyPI package. **The `aioswitcher` package must be installed on the machine where you run them.**

These scripts expose command-line interfaces for operations the library implements (device control, discovery) and for operations outside the library (fetching login keys from devices, validating cloud tokens).

Files:
- `discover_devices.py` — runs the Bridge and prints discovered devices (JSON via `dataclasses.asdict`). No code changes needed for new devices.
- `control_device.py` — large `argparse` subcommand-based CLI (~1100 lines). The `DEVICES` dict maps human-readable names to `DeviceType` enum members. Each action has an argparser subcommand + a named async function + a dispatch case in `main()`. Token-based methods get a `-k` flag. New API methods need new subparsers, async functions, and dispatch cases. New device types need a `DEVICES` entry.
- `get_device_login_key.py` — pings a device's UDP broadcast port to extract its login key.
- `validate_token.py` — validates a Switcher cloud token via the device API.

## Protocol Types

- **Type 1** (water heaters, power plug): port **9957** (TCP), UDP port **20002/10002**. No token required.
- **Type 2** (breeze, runners, heater): port **10000** (TCP), UDP port **20003/10003**. Breeze/runners may require a token.

## Architecture

### Discovery (UDP)
1. `SwitcherBridge` opens UDP sockets on all broadcast ports.
2. `UdpClientProtocol.datagram_received` receives raw bytes, passes to `DatagramParser`.
3. `DatagramParser` extracts device type from hex bytes at offset 74–76.
4. `_parse_device_from_datagram` routes to device-specific branch by `DeviceCategory`, builds dataclass, calls `on_device` callback.

### Control (TCP)
1. `SwitcherApi` connects via `open_connection()` → `_reader`/`_writer`.
2. Every command starts with `_login()` (Type1: 1 round-trip, Type2 token: 2 round-trips).
3. Build packet string from `packets.py` templates → `_send_packet()` signs with CRC key → sends → reads 1024 bytes.
4. Response bytes parsed via response classes in `messages.py`.

## Adding a New Device Type

When a new device type is announced, changes span ~8 files:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TomerFi/aioswitcher](https://github.com/TomerFi/aioswitcher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
