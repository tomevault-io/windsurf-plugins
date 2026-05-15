---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

UnrealBridge is a TCP bridge between external tools (Claude Code) and Unreal Engine 5.3+ (verified clean BuildPlugin against 5.3.2 / 5.4.4 / 5.5.4 / 5.6.1 / 5.7.1 via `tools/build_matrix.py`; some libraries are gated to 5.7+ and a few inline shims handle 5.3 — see `docs/version-compatibility.md`). UE 5.2 and earlier are not supported. It consists of:
- A UE Editor plugin (`Plugin/UnrealBridge/`) that runs a TCP server inside the editor
- A Python CLI client (`.claude/skills/unreal-bridge/scripts/bridge.py`) used by the `unreal-bridge` skill
- API reference docs and helper scripts for querying/manipulating UE assets via Python

## Key Commands

**Sync plugin to UE project and compile:**
```bash
sync_plugin.bat
```
Mirrors `Plugin/UnrealBridge/` into the target project's `Plugins/UnrealBridge/` (excluding `Binaries`/`Intermediate`). The DST path lives in `sync_plugin.bat` itself — do not hardcode it anywhere else.

**Test bridge connection:**
```bash
python .claude/skills/unreal-bridge/scripts/bridge.py ping
```

**Execute Python in UE:**
```bash
python .claude/skills/unreal-bridge/scripts/bridge.py exec "print('hello')"
python .claude/skills/unreal-bridge/scripts/bridge.py exec-file script.py
```

## Architecture

### TCP Protocol
Length-prefixed JSON over TCP on an OS-assigned port (default bind `127.0.0.1`, port auto-allocated at startup). Clients find the editor via the UDP multicast discovery service (see next section) — port 9876 is no longer hardcoded on the TCP data channel.
- Request: `[4 bytes big-endian length][JSON: {"id":"...", "script":"...", "timeout":30, "token":"..." (optional)}]`
- Response: `[4 bytes big-endian length][JSON: {"id":"...", "success":bool, "output":"...", "error":"..."}]`
- Token auth: required when the server binds non-loopback. The token is written to `<Project>/Saved/UnrealBridge/token.txt`; clients read it and add `"token":"<value>"` to every request. Constant-time compared on the server.
- Special commands (handled inline on the worker thread, bypass the Python exec queue):
  - `{"id":"...", "command":"ping"}` → `pong` (TCP-only liveness)
  - `{"id":"...", "command":"gamethread_ping", "timeout":2.0}` → `alive`/`unresponsive` + `latency_ms` (GT liveness)
  - `{"id":"...", "command":"debug_resume"}` → unsticks a paused BP breakpoint via `FKismetDebugUtilities::RequestAbortingExecution`

### Discovery Protocol
UDP multicast on `239.255.42.99:9876` (the one constant the whole system shares). Multiple editors can bind via `SO_REUSEADDR`.
- Probe (client → group): `{"v":1, "type":"probe", "request_id":"<uuid>", "filter":{"project":"<name|path|*>"}}`
- Response (server → probe source, unicast): `{"v":1, "type":"response", "request_id":"<uuid>", "pid":..., "project":"...", "project_path":"...", "engine_version":"...", "tcp_bind":"...", "tcp_port":..., "token_fingerprint":"<sha1(token)[:16]>"}`
- Client loop: probe → collect for `--discovery-timeout` ms (default 800) → filter by `--project=...` → connect TCP. Empty `token_fingerprint` means no token required.

### Server configuration (CLI / env / editor ini)
Priority CLI > env > `EditorPerProjectUserSettings.ini [UnrealBridge]` > default.

| CLI | Env | INI key | Default | Effect |
|---|---|---|---|---|
| `-UnrealBridgeBind=` | `UNREAL_BRIDGE_BIND` | `Bind` | `127.0.0.1` | Interface to bind TCP to |
| `-UnrealBridgePort=` | `UNREAL_BRIDGE_PORT` | `Port` | `0` | TCP port — `0` = OS-assigned ephemeral |
| `-UnrealBridgeToken=` | `UNREAL_BRIDGE_TOKEN` | `Token` | *(empty)* | Required when bind is not loopback |
| `-UnrealBridgeDiscoveryGroup=` | `UNREAL_BRIDGE_DISCOVERY_GROUP` | `DiscoveryGroup` | `239.255.42.99:9876` | Multicast group + port |
| `-UnrealBridgeDiscoveryEnabled=` | `UNREAL_BRIDGE_DISCOVERY` | `DiscoveryEnabled` | `1` | `0` = disable discovery responder |
| `-UnrealBridgeNoDiscovery` *(flag)* | — | — | — | Shorthand for `DiscoveryEnabled=0` |

### Plugin Module Structure
- **UnrealBridgeModule** — Module entry point at PostEngineInit. Parses config, starts TCP server + discovery responder, maps `/Plugin/UnrealBridge/` → Shaders dir
- **UnrealBridgeServer** — TCP listener, accepts clients on background threads, dispatches Python execution to GameThread via `IPythonScriptPlugin::ExecPythonCommandEx`. Uses `__UB_ERR__` sentinel to separate stdout from stderr in captured output
- **UnrealBridgeBlueprintLibrary** — Blueprint introspection: class hierarchy, variables, functions, components, interfaces, graph analysis (call graph, execution flow, node inspection, pin connections), timelines, event dispatchers, cross-graph search, write ops (set variable defaults, component properties, add variables)
- **UnrealBridgeAssetLibrary** — Asset search (keyword with include/exclude tokens), derived class queries, asset references/dependencies, DataAsset queries, folder listing
- **UnrealBridgeAnimLibrary** — AnimBlueprint introspection: state machines, AnimGraph nodes, linked layers, slots, curves, anim sequence/montage/blend space info, skeleton bone tree
- **UnrealBridgeDataTableLibrary** — DataTable row inspection

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TornLux/UnrealBridge](https://github.com/TornLux/UnrealBridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
