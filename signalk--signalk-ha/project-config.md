---
trigger: always_on
description: Build a production-grade Home Assistant custom integration (HACS) that ingests Signal K data and exposes it as high-quality Home Assistant entities with minimal user effort and maximal reliability.
---

# AGENTS.md

## Mission

Build a production-grade Home Assistant custom integration (HACS) that ingests Signal K data and exposes it as high-quality Home Assistant entities with minimal user effort and maximal reliability.

Primary goals:
- Correct and complete use of the Signal K data model and delta protocol
- REST-based discovery + WebSocket-based updates
- First-class Home Assistant entity modeling, lifecycle management, and diagnostics
- Reliability and self-healing suitable for real sailors on unreliable networks
- Support multiple Signal K instances, where each config entry represents exactly one vessel

Non-negotiables:
- Correct Signal K WebSocket subscription protocol handling (handshake, subscribe, resubscribe).
- Self-healing networking (robust reconnect, backoff, lifecycle safety, clean unload).
- Diagnostics that make failures obvious and fixable without “IT time”.
- Configuration that is easy for sailors: defaults, presets, copy/paste friendly, clear validation.
- One connection per config entry, shared by all entities.
- Correctness, stability, and debuggability always beat feature breadth.

Sailors do not want to debug. Reliability beats features.

---

## Context: Current State

Integration `signalk_ha` is in active use with a production-oriented structure:

- `custom_components/signalk_ha/`
  - `__init__.py` (setup, unload, subscription refresh)
  - `auth.py` (access request flow)
  - `config_flow.py` (discovery + auth + options)
  - `coordinator.py` (WS loop + discovery coordinator + notifications)
  - `rest.py` (server discovery + REST fetch helpers)
  - `discovery.py` (entity discovery, metadata, icons)
  - `schema.py` (Signal K schema metadata, v1.7.1)
  - `mapping.py` (explicit path mappings + conversions)
  - `parser.py` (delta parsing + notifications extraction)
  - `subscription.py` (build subscribe payloads)
  - `sensor.py` (sensor + health entities)
  - `geo_location.py` (navigation.position)
  - `event.py` (notification event entities)
  - `notifications.py` (normalize notification path selection)
  - `diagnostics.py`

Core technical choices so far:
- Signal K server discovery via `GET {server_url}/signalk` (use `endpoints.v1`).
- REST discovery via `/signalk/v1/api/vessels/self`.
- WebSocket endpoint: `/signalk/v1/stream?subscribe=none` with explicit subscribe payloads.
- Delta parsing into a latest-value cache, coalesced into HA updates.
- Notifications pipeline: HA bus event (`signalk_ha_notification`) plus optional Event entities.
- Schema-driven discovery with explicit overrides in `mapping.py`.

---

## Architectural Principles (Authoritative)

- REST API is used for discovery and metadata refresh.
- WebSocket (Signal K delta protocol) is used for live updates.
- REST and WebSocket concerns must be cleanly separated.
- Entity discovery is idempotent and repeatable.
- Transport frequency and HA state update frequency are decoupled.
- Failures are expected and must be handled gracefully.
- Never delete entities automatically.

---

## Config Flow (Initial Setup)

### Connection Parameters

Remove the old path-selection page entirely.

Collect:
- host
- port
- tls (ws / wss)
- verify TLS certificate (yes/no)

Normalize:
- scheme
- default ports
- trailing slashes
- hostname casing

Derived base URLs:
- REST: from server discovery (`endpoints.v1["signalk-http"]`)
- WS: from server discovery (`endpoints.v1["signalk-ws"]`)

Validation:
- Verify Signal K discovery endpoint reachable.
- Fetch `/vessels/self`.
- Fail fast on network, TLS, authentication, or protocol errors.

Notes:
- Do not append trailing slash to WS URL unless required.
- Store server_id/server_version from discovery in the config entry.

---

## Vessel Identity and Device Registry

### Vessel Resolution

Always use:
- `GET /signalk/v1/api/vessels/self`

Do not assume MMSI is present or stable.

Persist a stable vessel identifier with priority:
1. MMSI if present and valid
2. Stable vessel URN/key if exposed
3. Fallback: hash of normalized server URL + vessel name (last resort)

Persist separately:
- vessel identifier (identity)
- vessel name (display only)

### Device Registry

- Exactly one HA device per config entry (one vessel).
- Store:
  - vessel name (display)
  - vessel identifier (identity)
  - normalized Signal K base URL or instance key

Rules:
- Vessel name is display-only.
- Entity IDs and unique_ids must not change if vessel name changes.
- Never encode vessel name into entity_id.

### unique_id Format

Preferred:
signalk:<config_entry_id>:<full_signalk_path>


Alternative (only if vessel_id is guaranteed stable):
signalk:<vessel_id>:<full_signalk_path>

## Discovery via REST

### Source and Frequency

- Use full `/vessels/self` data model.
- Discovery runs:
  - on startup
  - on configurable interval (default 24h)

Discovery failures must never remove or unload existing entities.

### Scope (Default)

- environment
- tanks
- navigation

### Entity Creation Rules

Create entities only for:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SignalK/signalk-ha](https://github.com/SignalK/signalk-ha) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-02 -->
