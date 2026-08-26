---
trigger: always_on
description: handles the choice and reports it through `scanner.last_arp_method` and
---

# CLAUDE.md

Guidance for Claude Code (claude.ai/code) when working in this repository.

## Overview

MyNeS (My Network Scanner) is a Flask application that discovers, identifies and
monitors every device on a home network — including the ones an IP scan cannot
see (Bluetooth LE, Zigbee, Z-Wave) — and integrates with Home Assistant.

Target users run home labs: Raspberry Pi / Orange Pi clusters, NAS boxes, AI
workstations, and a Home Assistant install. Design decisions should favour that
audience: no cloud dependency, works on a LAN, degrades gracefully when an
optional dependency or privilege is missing.

## Commands

```bash
# One command, any OS: creates the venv, installs deps, checks nmap, runs.
python scripts/run.py

# Manual
python3 -m venv .venv && source .venv/bin/activate
pip install -e ".[all]"
python -m mynes                      # or: mynes

# Tests
.venv/bin/python -m pytest tests/ -q

# Individual module self-checks (each is runnable and asserts its own logic)
.venv/bin/python -m mynes.monitoring.rules
.venv/bin/python -m mynes.core.arp 192.168.1.0/24
.venv/bin/python -m mynes.discovery.mdns
.venv/bin/python -m mynes.integrations.home_assistant

# Docker (host networking is what makes LAN discovery work)
docker compose -f deploy/docker-compose.yml up -d
```

Default port: **5883** (`MYNES_PORT` to change).

## Architecture

```
mynes/
├── paths.py              Paths resolved from the package, not the CWD.
│                         Env overrides: MYNES_HOME/_CONFIG_DIR/_DATA_DIR.
├── core/
│   ├── scanner.py        LANScanner: the orchestrator. scan_network() ->
│   │                     get_devices(). Everything else hangs off this.
│   ├── arp.py            Layer-2 discovery. Raw ARP when privileged, else
│   │                     ping sweep + OS ARP cache. See "Privileges" below.
│   ├── models.py         Unified device model / normalisation
│   ├── network.py        Interface + gateway detection
│   ├── config.py         ConfigManager: config/config.json read/write
│   ├── topology.py       Parent/child uplink tree (traceroute + manual + infra
│   │                     heuristics). See core/subnets.py for the L3 grouping
│   │                     it hangs off, not the same question.
│   ├── subnets.py        Which subnet each device is actually in - a real
│   │                     interface/Docker CIDR when known, else the device's
│   │                     own /24. Feeds the topology/graph subnet overlay.
│   ├── diagnostics.py    On-demand ping/traceroute/port-probe/DNS for one
│   │                     device - thin OS-binary wrappers, parsing kept pure
│   │                     and tested separately from the subprocess calls.
│   └── version.py        Git-derived version
├── discovery/            One module per protocol, all optional, all isolated.
│   ├── base.py           DiscoveryBackend + DiscoveredDevice. safe_discover()
│   │                     never raises: a dead protocol yields [] and logs.
│   ├── mdns.py           mDNS/DNS-SD via zeroconf. Matter arrives here too
│   │                     (_matter._tcp / _matterc._udp) — there is no separate
│   │                     Matter stack.
│   ├── ssdp.py           SSDP/UPnP, stdlib sockets only, zero dependencies
│   ├── bluetooth.py      BLE via bleak. Devices keyed by BT address, no IP.
│   └── mqtt.py           Reads Zigbee2MQTT / Z-Wave JS / Tasmota / HA discovery
│                         retained topics — the only way to see radio devices.
├── analysis/             oui, identifier, hostname, advanced, enhanced
│   ├── fingerprint.py    Active service fingerprinting: RTSP/HTTP/SSH/FTP
│   │                     banners + an NBNS (UDP 137) SMB/NetBIOS probe, pure
│   │                     classify()/suggest_name() over the signals gathered.
│   └── os_detect.py      OS family + best-effort WiFi-vs-wired connection-
│                         medium guessing, consolidated from what used to be
│                         three separate, duplicated guessers. Everything here
│                         is a scored guess, never claimed as measured fact.
├── monitoring/
│   ├── rules.py          PURE functions: (previous, current) -> [Alert].
│   │                     No I/O. Test here first; it is the cheapest layer.
│   ├── notify.py         Channels (stdlib only). Add one = add one function
│   │                     to SENDERS.
│   ├── scheduler.py      One daemon thread: scan -> diff -> alert -> notify
│   └── store.py          Capped JSON alert history + monitor state
├── integrations/
│   ├── home_assistant.py MQTT Discovery push + REST pull/compare
│   └── docker.py         Container/network detection
├── platform/             Desktop OS integration. Nothing here runs a
│   ├── privileges.py     privileged command without an explicit --apply.
│   │                     Linux setcap / macOS ChmodBPF+access_bpf / Win Npcap.
│   ├── service.py        launchd LaunchAgent | systemd --user | Scheduled Task.
│   │                     All USER-level: no sudo, uninstall = delete one file.
│   └── files/            ChmodBPF script + its LaunchDaemon plist
├── tray.py               pystray menu bar / notification area icon (optional)
├── security/             credentials (Fernet + PBKDF2), sanitizer

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fxerkan/my_network_scanner](https://github.com/fxerkan/my_network_scanner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
