---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

DGAIOT is an open-source IoT platform for edge intelligence — a fork/customization of EMQX (Erlang MQTT broker v4.3.x). It transforms raw device connectivity into business-semantic operations via an OWL ontology engine. **Erlang 24.3**, **Rebar3** build system, OTP umbrella project with 26 applications.

**Core philosophy**: Model → Ontology → Device Access → Time-Series → Rules → Dashboard (6-step FDE workflow).

## Build & Run

```bash
# Full build + release (dgiot profile, the default)
make

# Full build + start in console mode (for dev/debug)
make run

# Build with the original EMQX profile
DGIOT_WITH_EMQX=true make run

# Verify both native and EMQX modes compile cleanly
make verify
```

**Do NOT use `make compile`** — this command is unsupported and will hang.

### Hot Compile (Primary Dev Workflow)

After `make run` is running, hot-reload changed modules without restart:

```bash
# Hot compile a plugin (general)
_build/emqx/rel/emqx/bin/emqx eval 'dgiot_plugin:compile(dgiot).'

# Hot compile a specific plugin (e.g., modbus)
_build/emqx/rel/emqx/bin/emqx eval 'dgiot_plugin:compile(dgiot_modbus).'

# Hot reload
_build/emqx/rel/emqx/bin/emqx eval 'dgiot_plugin:reload_plugin(dgiot_modbus).'
```

Hot compile must produce **zero warnings**. Unused variables must use `_` prefix; unused functions must be removed or exported.

## Testing

```bash
# All EUnit tests (with coverage)
make eunit

# All Common Test suites (with coverage)
make ct

# PropEr property-based tests
make proper

# Tests for a specific app (e.g., dgiot_modbus, dgiot_parse, dgiot_ontology)
make dgiot_modbus-ct
make dgiot_parse-ct

# Plugin test framework (Modbus examples)
make test-modbus                # All Modbus tests
make test-modbus-simple         # Basic connectivity
make test-plugin PLUGIN=dgiot_xxx TESTCASE=simple

# List available plugins and test cases
make list-plugins
make list-testcases PLUGIN=dgiot_modbus
```

## Static Analysis

```bash
make xref       # Cross-reference checks
make dialyzer   # Type analysis (slow, use sparingly)
make cover      # Coverage report
```

## Architecture: 7-Layer Design

All code follows strict layer separation. Each layer communicates only with adjacent layers via defined interfaces. Never skip layers.

| Layer | Responsibility | Examples |
|-------|---------------|----------|
| 1. Transport | TCP/UDP connections, device registration, raw data forwarding | `dgiot_modbusrtu_tcp.erl` |
| 2. Protocol | Frame parse/encode, CRC check, format conversion | `modbus_rtu.erl` |
| 3. Message Routing | MQTT routing, task queues, parent-device aggregation | Message routing functions |
| 4. Business | Data decoding, attribute calculation, alarms, state mgmt | `dgiot_task.erl` |
| 5. Data | Time-series storage, queries, aggregation (TDengine) | `dgiot_tdengine_adapter.erl` |
| 6. Cache | Real-time cache, device state cache, session mgmt | Cache logic in `dgiot_task.erl` |
| 7. API | REST/gRPC queries, control commands, dashboard data | API handler modules |

**Key rules**:
- Transport layer: NEVER decode data or call business logic
- Protocol layer: NEVER store data or call business functions — use hooks
- Business layer: NEVER directly access the database — use data layer APIs
- Cross-layer calls use the hook system: `dgiot_hook:run_hook/2`

## Umbrella App Structure

```
apps/
  dgiot/             Core platform (supervisor, transport, crypto, rules, channels)
  dgiot_api/         REST API handlers and routers
  dgiot_parse/       Parse Server integration (PostgreSQL ORM substitute)
  dgiot_ontology/    OWL ontology engine (new, active development)
  dgiot_bridge/      Device bridge/connector framework
  dgiot_device/      Device management (thing models)
  dgiot_task/        Task and data processing pipeline
  dgiot_tdengine/    TDengine time-series DB adapter
  dgiot_http/        HTTP protocol adapter
  dgiot_modbus/      Modbus RTU/TCP protocol adapter
  dgiot_meter/       Meter reading protocols
  dgiot_topo/        Topology visualization
  dgiot_dlink/       Data link/channel management
  dgiot_bamis/       BI/analytics views
  emqx_*             EMQX plugins (auth, management, rule engine, retainer, etc.)
src/                  EMQX broker core (connection, channel, session, MQTT protocol)
lib-ce/               Community Edition: dashboard, modules, telemetry
```

## Extension Mechanisms

Three ways to extend the platform:

1. **Hook system** (`dgiot_hook`): Pub-sub callbacks. `one_for_one` (single handler) or `one_for_more` (chained). Register: `dgiot_hook:add/2,3`. Fire: `dgiot_hook:run_hook/2`. Hooks are the ONLY way for protocol and business layers to communicate.

2. **Protocol decoders** (`dgiot_protocol`): Modules declare `-protocol([MsgType1, MsgType2]).` in source. On startup `dgiot_protocol:start/0` auto-registers them as `{MsgType, parse_frame}` and `{MsgType, to_frame}` hooks. New protocol modules discovered via `dgiot_plugin:check_module/1`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dgiot/dgiot](https://github.com/dgiot/dgiot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
