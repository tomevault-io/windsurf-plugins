---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Home Assistant custom integration for controlling smart adjustable beds via Bluetooth Low Energy (BLE). It replaces the broken `smartbed-mqtt` addon with a native HA integration that uses Home Assistant's Bluetooth stack directly.

**Current status:** 44 bed protocols implemented. Linak, Keeson, Richmat, MotoSleep, Jensen, Svane, Vibradorm, Octo, Okin UUID/Okimat, Okin CB24, SUTA, and BedTech tested. Other brands need community testing.

## GitHub Comment Approval

- Never post GitHub comments (issues, pull requests, discussions, releases, etc.) without explicit and specific user approval for that exact comment action.
- If approval is missing or ambiguous, ask before posting.

## Architecture

```
custom_components/adjustable_bed/
├── __init__.py          # Integration setup, platform loading, service registration
├── config_flow.py       # Device discovery and setup wizard
├── coordinator.py       # BLE connection management (central hub)
├── const.py             # Constants, UUIDs, bed type definitions, feature flags
├── entity.py            # Base entity class
├── adapter.py           # BLE adapter selection, device lookup
├── detection.py         # Bed type auto-detection from BLE services/names
├── controller_factory.py # Factory for creating bed controller instances
├── validators.py        # Config validation (MAC addresses, PIN, variants)
├── redaction.py         # Data redaction for diagnostics
├── support_report.py    # Support report generation
├── richmat_features.py  # Richmat feature detection helpers
├── actuator_groups.py   # Actuator group logic
├── diagnostics_utils.py # Diagnostics helper utilities
├── beds/                # Bed controller implementations
│   ├── base.py          # Abstract base class (BedController)
│   ├── linak.py         # Linak protocol (tested)
│   ├── richmat.py       # Richmat Nordic/WiLinke protocols (tested)
│   ├── keeson.py        # Keeson KSBT/BaseI4/I5/Ergomotion/Serta protocols (tested)
│   ├── motosleep.py     # MotoSleep HHC ASCII protocol (tested)
│   ├── solace.py        # Solace 11-byte packet protocol
│   ├── leggett_gen2.py  # Leggett & Platt Gen2 ASCII protocol
│   ├── leggett_okin.py  # Leggett & Platt Okin binary protocol
│   ├── leggett_wilinke.py # Leggett & Platt WiLinke 5-byte protocol
│   ├── reverie.py       # Reverie XOR checksum protocol
│   ├── okin_uuid.py     # Okin 6-byte via UUID protocol
│   ├── okin_handle.py   # Okin 6-byte via handle protocol
│   ├── okin_7byte.py    # Okin 7-byte protocol
│   ├── okin_nordic.py   # Okin 7-byte via Nordic UART
│   ├── okin_cb24.py     # Okin CB24 protocol via Nordic UART
│   ├── okin_cb35.py     # Okin CB35 Star protocol (Sealy Posturematic)
│   ├── okin_ore.py      # Okin ORE protocol (A5 5A format)
│   ├── okin_protocol.py # Shared Okin protocol utilities
│   ├── okin_64bit.py    # OKIN 64-bit 10-byte protocol
│   ├── malouf.py        # Malouf NEW_OKIN/LEGACY_OKIN protocols
│   ├── jiecang.py       # Jiecang/Glide/Comfort Motion protocol
│   ├── octo.py          # Octo standard/Star2 protocols (PIN auth)
│   ├── jensen.py        # Jensen JMC400 protocol (tested, PIN auth)
│   ├── svane.py         # Svane LinonPI multi-service protocol
│   ├── bedtech.py       # BedTech 5-byte ASCII protocol
│   ├── reverie_nightstand.py  # Reverie Protocol 110 (nightstand)
│   ├── sleepys.py       # Sleepy's Elite BOX15/BOX24 protocols
│   ├── vibradorm.py     # Vibradorm VMAT protocol (tested)
│   ├── rondure.py       # Rondure 8/9-byte FurniBus protocol
│   ├── remacro.py       # Remacro 8-byte SynData protocol
│   ├── coolbase.py      # Cool Base (Keeson BaseI5 with fan control)
│   ├── scott_living.py  # Scott Living 9-byte protocol
│   ├── sbi.py           # SBI/Q-Plus (Costco) with position feedback
│   ├── suta.py          # SUTA Smart Home AT protocol
│   ├── timotion_ahf.py  # TiMOTION AHF 11-byte bitmask protocol
│   ├── limoss.py        # Limoss / Stawett TEA-encrypted protocol
│   ├── logicdata.py     # Logicdata SimplicityFrame XXTEA+CRC16+SLIP protocol
│   ├── okin_cst.py      # Okin CSTProtocol 14-byte dual-field commands
│   └── diagnostic.py    # Debug controller for unsupported beds
├── binary_sensor.py     # BLE connection status entity
├── button.py            # Preset and massage button entities
├── cover.py             # Motor control entities (open=up, close=down)
├── number.py            # Number entities (angle settings, etc.)
├── select.py            # Select entities (variant selection, etc.)
├── sensor.py            # Position angle feedback entities
├── switch.py            # Light control entities
├── diagnostics.py       # HA diagnostics download support
├── ble_diagnostics.py   # BLE protocol capture for new bed support
└── unsupported.py       # Unsupported device guidance (Repairs integration)
```

### Key Components

**AdjustableBedCoordinator** (`coordinator.py`): Central BLE connection manager
- Handles device discovery via HA's Bluetooth integration
- Connection retry with progressive backoff (3 attempts, 5-7.5s delays)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kristofferR/ha-adjustable-bed](https://github.com/kristofferR/ha-adjustable-bed) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
