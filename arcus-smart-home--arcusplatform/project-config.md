---
trigger: always_on
description: The agent runs on Iris Hub v2/v3 hardware, managing ZigBee and Z-Wave radios, executing local device reflexes, handling security alarms, and maintaining a persistent WebSocket connection to the Arcus cloud platform. It is organized as 13+ Gradle submodules with a Guice plugin architecture.
---

# Agent — Hub/Gateway Software (`/agent`)

The agent runs on Iris Hub v2/v3 hardware, managing ZigBee and Z-Wave radios, executing local device reflexes, handling security alarms, and maintaining a persistent WebSocket connection to the Arcus cloud platform. It is organized as 13+ Gradle submodules with a Guice plugin architecture.

---

## Table of Contents

- [Module Overview](#module-overview)
- [Dependency Graph](#dependency-graph)
- [Key External Dependencies](#key-external-dependencies)
- [Startup and Lifecycle](#startup-and-lifecycle)
- [Message Routing](#message-routing)
- [Cloud Gateway](#cloud-gateway)
- [Hub Controller](#hub-controller)
- [Reflex Controller](#reflex-controller)
- [Z-Wave Controller](#z-wave-controller)
- [Alarm Controller](#alarm-controller)
- [Hardware Abstraction Layer](#hardware-abstraction-layer)
- [Database and Storage](#database-and-storage)
- [Configuration System](#configuration-system)
- [Spy Controller](#spy-controller)
- [Key Design Patterns](#key-design-patterns)
- [Message Flow Examples](#message-flow-examples)
- [Running Locally](#running-the-agent-locally-simulated-mode)
- [Build Output](#build-output)
- [Known Missing Components](#known-missing-components-closed-source)

---

## Module Overview

| Module | Purpose |
|--------|---------|
| `arcus-agent` | Main entry point (`IrisAgent`); loads config, bootstraps Guice |
| `arcus-system` | Core services: lifecycle, config, DB, storage, addressing, HAL facade, metrics, SSL, logging, watchdog |
| `arcus-hal/api` | Hardware Abstraction Layer interfaces (`IrisHal`, `IrisHalInternal`, `LEDState`, `SounderMode`) |
| `arcus-hal/common` | Shared HAL base implementations |
| `arcus-hal/hub-v2` | Hub v2/v3 HAL: LED, buzzer, reset button, battery, WiFi, watchdog, OS calls |
| `arcus-hal/simulated` | Mock HAL for desktop testing (no real I/O) |
| `arcus-gateway` | Netty WebSocket client — hub-to-platform connectivity with dual-interface failover |
| `arcus-router` | Async message routing with address-based dispatch (`LinkedTransferQueue`) |
| `arcus-hub-controller` | Hub device attributes, capabilities, and top-level message dispatch |
| `arcus-reflex-controller` | Local rule/automation execution (reflexes) — see [reflexes.md](reflexes.md) |
| `arcus-alarm-controller` | Hub-local security/safety alarm state machine |
| `arcus-zigbee-controller` | ZigBee protocol (zsmartsystems 1.4.16) — device discovery, IAS Zone, AlertMe support |
| `arcus-zw-controller` | Z-Wave protocol (Z/IP engine) |
| `arcus-os` | OS abstraction via JNA + Netty epoll (serial ports, watchdog) |
| `arcus-spy-controller` | Diagnostic message snooping (enabled via `IRIS_HUB_SPY_ACTIVE`) |
| `arcus-test-agent` | Shared test utilities and mocks |

---

## Dependency Graph

```
arcus-agent (entry point)
 +-- arcus-system (core foundation)
 +-- arcus-hub-controller
 |    +-- arcus-router --> arcus-system
 |    +-- arcus-reflex-controller
 |    |    +-- arcus-zw-controller
 |    |    +-- arcus-zigbee-controller
 |    +-- arcus-alarm-controller --> arcus-reflex-controller
 |    +-- arcus-zw-controller --> arcus-os
 |    +-- arcus-zigbee-controller --> arcus-os
 +-- arcus-hal:arcus-hal-api

arcus-gateway
 +-- arcus-system
 +-- arcus-router

arcus-spy-controller
 +-- arcus-system
 +-- arcus-router
```

## Key External Dependencies

| Library | Purpose |
|---------|---------|
| Netty | Async I/O (WebSocket, epoll) |
| SQLite4Java | Embedded database with platform-specific native bindings |
| Google Guice | Dependency injection (PRODUCTION stage) |
| RxJava | Reactive streams (Z-Wave scenes, reflex processing) |
| GSON | JSON serialization for platform messages |
| BouncyCastle | Cryptography and TLS |
| zsmartsystems ZigBee (v1.4.16) | ZigBee network manager and Ember EZSP dongle support |

---

## Startup and Lifecycle

### Entry Point

**`IrisAgent.java`** (`arcus-agent/.../com/iris/agent/IrisAgent.java`)

The agent's `main()` method:

1. Initializes logging (`IrisAgentLogging.setupInitialLogging()`) — selects DEV or STDOUT mode based on `IRIS_AGENT_LOGTYPE` env var
2. Starts the `WatchdogService` and registers a 5-minute startup timeout
3. Calls `BootUtils.initialize()` with the base path (first arg) and config directories (remaining args)
4. Blocks on `IrisHal.waitForShutdown()` until the agent is told to exit

### Bootstrap Sequence

**`BootUtils.java`** (`arcus-system/.../boot/BootUtils.java`)

The bootstrap builds a Guice injector from multiple module sources:

1. `IrisHal.start(basePath, configs)` — Initialize hardware abstraction and load config files
2. Collect bootstrap modules and application modules from the HAL
3. Create Guice injector in `PRODUCTION` stage with:
   - `LifecycleModule` (always first)
   - `AgentConfigurationProvider`
   - HAL bootstrap modules
   - HAL application modules
   - `MessagesModule`, `ProtocolMessagesModule`, `GsonModule`, `AgentModule`
4. `Bootstrap.bootstrap()` — Create injector
5. `IrisLifecycleManager.start()` — Trigger `@PostConstruct` and `@WarmUp` lifecycle phases on all bound services

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [arcus-smart-home/arcusplatform](https://github.com/arcus-smart-home/arcusplatform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
