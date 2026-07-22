---
trigger: always_on
description: Infinitude is an alternative web service for Carrier Infinity Touch and compatible thermostats. It provides local control bypassing Carrier's cloud services, allowing direct web-based control of:
---

## Project Overview

Infinitude is an alternative web service for Carrier Infinity Touch and compatible thermostats. It provides local control bypassing Carrier's cloud services, allowing direct web-based control of:

- Temperature setpoints
- Schedules
- Dealer information
- Home Assistant integration via MQTT Discovery

Infinitude also optionally monitors and writes to the Carrier/Bryant RS485 (ABCD) bus to obtain higher resolution access to thermostat, air handler, heat pump, and other device registers. Serial data can be monitored via an attached serial port or networked serial bridge.

**Important:** Infinitude is not compatible with all thermostat firmware versions. See the [Compatibility Matrix](https://github.com/nebulous/infinitude/wiki/Infinitude-Compatibility-Matrix) for known-working combinations.

## Architecture

### Backend
- **Language:** Perl
- **Framework:** Mojolicious::Lite
- **Object System:** Moo
- **Caching:** CHI (file-based)
- **Protocol:** Custom CarBus RS485 implementation
- **MQTT:** Net::MQTT::Simple (optional, for Home Assistant Discovery)

### Frontend
- **Framework:** Alpine.js
- **UI:** Bootstrap 5
- **Icons:** Bootstrap Icons
- **No build step** — plain JS/CSS served directly

### Key Components
- Main web application with RESTish API
- CarBus protocol handler for RS485 communication
- XML parsing and processing
- Serial monitoring with real-time register tracking
- MQTT Discovery for Home Assistant climate entities and sensors
- Shared mutation layer (`Infinitude.pm`) for XML→RS485→MQTT writes

## Directory Structure

```
infinitude          # Main application entry point (Perl Mojolicious app)
lib/
  Infinitude.pm     # Shared mutation utilities (modify_system, set_* methods)
  Infinitude/
    MQTT.pm         # MQTT Discovery, state publishing, command subscription
  CarBus.pm         # RS485 bus protocol handler
  CarBus/Frame.pm   # Frame parsing and validation
  CarBus/SAM.pm     # SAM emulation and register definitions
  CarBus/SAM/ASCII.pm # SAM ASCII serial protocol interface
  samterm           # Interactive SAM ASCII terminal
  cbt.pl            # CarBus test/bridge script
  XML/Simple/Minded.pm  # Custom XML parser
docs/
  SAM-ASCII-Protocol.md # SAM RS-232 ASCII protocol specification
public/
  scripts/app.js    # Alpine.js frontend application
  styles/main.css   # Styles
  index.html        # Single-page app entry
defs/               # XML configuration schemas for systems
state/              # Runtime data and cache (gitignored)
t/                  # Test suite
contrib/
  cardump/          # C-based RS485 packet dumper utility
climate             # Command-line utility for RS485 communication
```

## Commands

### Running with Docker (Recommended)
```bash
# Basic run
docker run --rm -v $PWD/state:/infinitude/state -p 3000:3000 nebulous/infinitude

# With environment variables
docker run --rm -v $PWD/state:/infinitude/state \
  -e APP_SECRET='YOUR_SECRET_HERE' \
  -e PASS_REQS='1020' \
  -p 3000:3000 \
  nebulous/infinitude

# Using docker-compose
docker-compose up
```

### Running from Source
```bash
# Install dependencies
cpanm --installdeps .

# Development mode (port 3000)
./infinitude daemon

# Production mode (port 80)
./infinitude daemon -l http://:80
```

### Testing
```bash
# Run all tests (requires local perl modules)
perl -I ~/perl5/lib/perl5 -I lib t/01-xml-simple-minded.t
perl -I ~/perl5/lib/perl5 -I lib t/02-infinitude.t

# Run with prove
prove -I ~/perl5/lib/perl5 -I lib t/
```

## Configuration

Configuration is stored in `infinitude.json` and can be overridden via environment variables:

| Variable | Description |
|----------|-------------|
| `APP_SECRET` | Cookie signature string |
| `PASS_REQS` | Min seconds between Carrier server requests (0 = never) |
| `MODE` | `production` (default) or `development` |
| `SERIAL_TTY` | RS485 device (e.g., `/dev/ttyUSB0`) |
| `SERIAL_SOCKET` | TCP/RS485 bridge (e.g., `192.168.1.42:23`) |
| `LOGLEVEL` | Minimum log severity |
| `SCAN_THERMOSTAT` | Enable continuous thermostat table scanning |
| `EMULATE_SAM` | Enable SAM emulation on RS485 bus (1 = enabled, experimental) |
| `MQTT_BROKER` | MQTT broker address (e.g., `192.168.1.3:1883`) |
| `MQTT_USER` | MQTT username |
| `MQTT_PASS` | MQTT password |
| `MQTT_PREFIX` | HA discovery prefix (default: `homeassistant`) |
| `MQTT_TOPIC` | Base MQTT topic (default: `infinitude`) |

## Code Conventions

- **Perl Library Path:** All Perl commands must include `-I ~/perl5/lib/perl5 -I lib` to find local modules
- **Boilerplate:** All Perl files use `use strict`, `use warnings`, `use feature ':5.10'`
- **OOP:** Use Moo for object-oriented programming
- **Error Handling:** Use Try::Tiny for exceptions
- **Web Framework:** Mojolicious::Lite with hooks and routes
- **File Operations:** Path::Tiny for file handling

## Key Files

| File | Purpose |
|------|---------|
| `infinitude` | Main application - routes, API endpoints, business logic, MQTT wiring |
| `lib/Infinitude.pm` | Shared mutation layer — XML modify/save, RS485 writes, MQTT publish |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nebulous/infinitude](https://github.com/nebulous/infinitude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
