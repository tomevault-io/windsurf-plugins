---
trigger: always_on
description: > **Agent context file.** Read this before making any changes. Keep it up to date.
---

# CLAUDE.md — OpenCastor Development Guide

> **Agent context file.** Read this before making any changes. Keep it up to date.

## What Is OpenCastor?

OpenCastor is an open-source **productized open-core RCAN runtime** — Layer 4 of the OpenCastor stack. It connects LLM "brains" to robot "bodies" through a plug-and-play architecture and exposes them to messaging platforms for remote control.

- **Version**: 2026.4.17.0 (date-based: `YYYY.MM.DD.patch`)
- **RCAN**: v3.0 — see [rcan.dev/spec](https://rcan.dev/spec/)
- **License**: Apache 2.0 | **Python**: 3.10+ | **Tests**: 7804+ passing

## Quick Start

```bash
git clone https://github.com/craigm26/OpenCastor.git
cd OpenCastor
pip install -e ".[channels]"
cp .env.example .env
castor wizard        # interactive setup
castor gateway       # start API gateway
```

## Repository Layout

```
OpenCastor/
├── castor/                 # Core runtime
│   ├── api.py              # FastAPI gateway (main entry point)
│   ├── tiered_brain.py     # TieredBrain: fast/planner routing by task_category
│   ├── providers/          # LLM adapters (Gemini, Claude, GPT, Ollama, ...)
│   │   ├── task_router.py  # TaskRouter — routes tasks by category to providers
│   │   └── base.py         # BaseProvider ABC + Thought dataclass
│   ├── drivers/            # Hardware drivers (see full list below)
│   ├── channels/           # Messaging channels (WhatsApp, Telegram, Discord, ...)
│   │   └── rcan_mqtt_transport.py  # RCAN-over-MQTT carrier (compact/minimal encoding)
│   ├── contribute/         # Idle compute donation skill
│   │   ├── coordinator.py  # BOINC + simulated coordinators
│   │   ├── runner.py       # Work unit runner with cancellation
│   │   ├── work_unit.py    # WorkUnit / WorkUnitResult dataclasses
│   │   └── hardware_profile.py  # NPU/CPU detection
│   ├── rcan/               # RCAN protocol implementation
│   │   ├── registry.py     # RRN validation, REGISTRY_REGISTER/RESOLVE (§21)
│   │   ├── invoke.py       # InvokeRequest/Result, SkillRegistry (§19)
│   │   ├── parallel_invoke.py  # invoke_all(), invoke_race()
│   │   ├── message.py      # MessageType enum, RCANMessage
│   │   └── sdk_compat.py   # Compatibility layer for rcan-py SDK
│   ├── safety/             # Safety subsystem
│   │   ├── p66_manifest.py # P66 safety manifest — capability declarations
│   │   ├── bounds.py       # BoundsChecker — motor command validation
│   │   ├── monitor.py      # SensorMonitor — sensor health wiring
│   │   ├── authorization.py # HiTL authorization (§8)
│   │   ├── protocol.py     # SafetyLayer — wraps driver calls
│   │   └── state.py        # Safety state machine
│   ├── hardware/
│   │   └── so_arm101/      # SO-ARM101 6-DOF arm
│   │       ├── safety_bridge.py  # Routes arm commands through SafetyLayer
│   │       ├── vision.py         # Arm camera/vision pipeline
│   │       ├── rcan_bridge.py    # RCAN→arm command translation
│   │       └── cli.py            # Arm CLI utilities
│   ├── fleet/              # Fleet management, group policies
│   ├── privacy_mode.py     # Privacy mode — blocks cloud egress
│   └── sdk/                # Python SDK wrapper
├── sdk/js/                 # TypeScript/JS SDK (@opencastor/sdk)
│   └── src/index.ts        # CastorClient — typed wrappers for all API endpoints
├── website/                # Astro-based website (replaces old site/)
│   └── src/pages/          # Astro pages (index.astro, docs.astro, ...)
├── tests/                  # Test suite (pytest)
├── config/presets/         # RCAN config presets for common hardware
├── bob.rcan.yaml           # Bob robot config (gitignored — device-specific)
└── CHANGELOG.md            # Version history
```

## Key Abstractions

| Class | File | What it does |
|---|---|---|
| `TieredBrain` | `castor/tiered_brain.py` | Routes prompts: fast model or planner based on `task_category` |
| `TaskRouter` | `castor/providers/task_router.py` | Selects provider by `TaskCategory` (SENSOR_POLL → local-only, SAFETY → planner) |
| `BaseProvider` | `castor/providers/base.py` | LLM adapter ABC: `think()`, `think_stream()`, `health_check()` |
| `DriverBase` | `castor/drivers/base.py` | Hardware ABC: `move()`, `stop()`, `close()`, `health_check()`. Subclasses implement `_move()` — raw hardware call — while `move()` routes through `SafetyLayer` first |
| `SafetyLayer` | `castor/safety/protocol.py` | Wraps all driver commands; enforces bounds, HiTL gates, safety state |
| `BoundsChecker` | `castor/safety/bounds.py` | Validates motor commands against configured limits |
| `SensorMonitor` | `castor/safety/monitor.py` | Polls sensors; wires health signals into safety state |
| `P66Manifest` | `castor/safety/p66_manifest.py` | Declares robot capabilities and safety constraints (P66 standard) |
| `RegistryMessage` | `castor/rcan/registry.py` | RCAN §21 wire message. `RRNCategory` enum, `_validate_rrn()`, `metadata` block |
| `InvokeRequest` | `castor/rcan/invoke.py` | §19 INVOKE — skill name + params + timeout |
| `SkillRegistry` | `castor/rcan/invoke.py` | Maps skill names to handler callables |
| `FleetManager` | `castor/fleet/group_policy.py` | Group policies, config deep-merge |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [craigm26/OpenCastor](https://github.com/craigm26/OpenCastor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
