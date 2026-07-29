---
trigger: always_on
description: Rules for working in Source/Thunder/ — Server, ServiceMap, Controller, Config, PostMortem, Probe
---


# Thunder `Source/Thunder/` — Daemon Runtime

`Source/Thunder/` is the **top-level daemon**: it assembles `core/`, `com/`, and `plugins/` into the running `Thunder` process. This directory contains the most security-sensitive and lifecycle-critical code in the framework. Every change here affects all plugins and all clients.

## Layer Scope
- Depends on `Source/core/`, `Source/com/`, `Source/plugins/`.
- **Never** include `ThunderInterfaces/` directly — the daemon itself has no business logic, only infrastructure.
- Entry point: `PluginHost.cpp` → `Server` (in `PluginServer.h` / `PluginServer.cpp`).

## `PluginServer.h` — Master File (~4800+ lines)
- Contains `Server`, `ServiceMap`, inner `Service` class, `Channel`, `ThrottleQueueType`, `ConfigObserver`.
- Read this file before touching any lifecycle or dispatch path — many subtleties are documented in comments there.
- Do not split into smaller files without very careful ABI and linkage consideration.

## `ServiceMap` — Plugin Registry
- All access to the plugin map is guarded by `_adminLock` (`Core::CriticalSection`).
- **Acquire `_adminLock` as briefly as possible** — never call plugin code (`Initialize`, `Deinitialize`, JSON-RPC handlers) while holding it.
- `_pluginHandling` (per-`Service`) guards per-plugin interface pointers — separate from `_adminLock`. Never hold both simultaneously.
- Lookup by callsign: `ServiceMap::FromLocator()` resolves a URL segment to a `Service*`.
- Plugin lifecycle states follow `PluginHost::IShell::state` (see `Source/plugins/IShell.h`): `UNAVAILABLE`, `DEACTIVATED`, `ACTIVATION`, `PRECONDITION`, `ACTIVATED`, `DEACTIVATION`, `HIBERNATED`, `DESTROYED`. Only the framework transitions states — never force-set `_state` directly.

## `Service` — Plugin Wrapper
- `Service` wraps the loaded plugin library and its `IPlugin` implementation.
- Activation path: `Service::Activate()` → loads the plugin shared library → calls `Initialize()` → transitions to `ACTIVATED`.
- Deactivation path: `Service::Deactivate()` → calls `Deinitialize()` → unloads the plugin shared library.
- Deactivation reasons are represented by `PluginHost::IShell::reason` enum: `REQUESTED`, `AUTOMATIC`, `FAILURE`, `STARTUP`, `SHUTDOWN`, `CONDITIONS`, `WATCHDOG_EXPIRED`, `INITIALIZATION_FAILED`. The reason is **not** passed as a parameter to `IPlugin::INotification::Deactivated()` — retrieve it inside the callback via `shell->Reason()`.
- `Service::Submit(job)` enqueues a job through `ThrottleQueueType` — use this for all plugin-context async work.

## Dispatch & Throttle
- `ThrottleQueueType` limits concurrent in-flight jobs per plugin — prevents a misbehaving plugin from starving the `WorkerPool`.
- JSON-RPC invocations arrive on the `WorkerPool` thread, not the network thread — do not block the network thread.
- HTTP / Web requests go through `Service::Process()` → `IWeb::Process()` — also on the `WorkerPool`.
- `Channel::Submit()` posts a response back to the client — always call on the response path even for error cases.

## `Controller` — Built-in Management Plugin

Controller is the only plugin that is part of `Source/Thunder/` itself. Key JSON-RPC methods it exposes:

| Method | Purpose |
|--------|--------|
| `activate` | Activate a plugin by callsign |
| `deactivate` | Deactivate a plugin by callsign |
| `status` | Query plugin status, version, state |
| `configuration` | Read/write plugin config JSON |
| `storeconfig` | Persist plugin config to disk |
| `discovery` | List all known plugins |
| `harakiri` | Request daemon self-termination |

- The Controller's callsign is `"Controller"` (configurable but conventionally fixed).
- Do **not** shortcut `ServiceMap` state transitions by calling internal methods — always go through `Controller`'s JSON-RPC interface or `Service::Activate()`/`Deactivate()` public methods.
- `Controller` also manages `ConfigObserver` (hot-reload of plugin `.json` configs) and proxy stub hot-reload.

## `Config.h` — Daemon Configuration Schema
- Add new daemon-level config options here as `Core::JSON::Container` members — never via environment variables or compile-time constants.
- Key fields: `port`, `binding`, `communicator`, `persistentpath`, `datapath`, `systempath`, `proxystubpath`, `configs`, `exitreasons`, `postmortempath`, `observe`.
- Config is loaded once at startup from the file pointed to by `-c` argument (default `/etc/Thunder/config.json`).
- Path substitution tokens (`%datapath%`, `%persistentpath%`, `%volatilepath%`, `%systempath%`, `%proxystubpath%`) are resolved by `Config::Substitute()` — use them in plugin config strings, never absolute paths.

## `PostMortem.cpp` — Crash Diagnostics
- On abnormal exit, Thunder writes a post-mortem dump to `postmortempath`.
- Add new dump sections to `PostMortem.cpp` via the existing `Dump*()` function pattern — never add ad-hoc `fprintf`/syslog calls for crash data.
- `exitreasons` config controls which shutdown reasons trigger a dump (e.g. `WATCHDOG`, `FAILURE`).

## `Probe.cpp` — Process Health Monitor
- `Probe` implements the watchdog for OOP plugin processes — it detects unresponsive child processes.
- Do not bypass the probe mechanism with custom liveness checks.

## `IRemoteInstantiation.h` — OOP Plugin Launch Protocol

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rdkcentral/Thunder](https://github.com/rdkcentral/Thunder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
