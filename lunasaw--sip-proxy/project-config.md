---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SIP Proxy is a GB28181-2016 communication framework built on Java 17 and Spring Boot 3.3.1. It is a multi-module Maven project providing complete SIP protocol communication for video surveillance systems.

It is delivered as a **Maven library**, not a standalone service. Business systems (typically a `sip-gateway` layer the user implements on top) embed it in-process and interact via Spring Events (inbound) and `CommandSender` beans (outbound). A single JVM can act as platform server (`gb28181-server`) and device client (`gb28181-client`) simultaneously for cascading scenarios.

Current version: **1.7.0** (see `CHANGELOG.md`). Recent breaking changes worth knowing before touching outbound code:

- **1.7.0** — Outbound dialog rewrite. `BYE` and `SUBSCRIBE` refresh/unsubscribe are now dialog-aware: `ServerCommandSender.deviceBye(callId)` (deviceId removed), `ClientCommandSender.sendByeCommand(callId)`, `SipSender.doByeRequest(callId)` (old `(FromDevice, ToDevice)` overload **deleted**, not deprecated). Calls without an established dialog throw `DialogNotFoundException` instead of getting a silent `481`. New `DialogRegistry` + `DialogRegistryCleaner` are core to INVITE/SUBSCRIBE flows. See [doc/plans/1.7.0/OUTBOUND-DIALOG-PLAN.md](doc/plans/1.7.0/OUTBOUND-DIALOG-PLAN.md).
- **1.5.0** — Listener-layered API. Removed 4 client business handler interfaces + 10 old client events; added `QueryListener` / `ControlListener` / `ConfigListener` / `SubscribeListener` / `NotifyListener` (client) and `DeviceResponseListener` / `DeviceNotifyListener` / `DeviceLifecycleListener` / `DeviceSessionListener` (server). See [doc/architecture/LISTENER-LAYERED-DESIGN.md](doc/architecture/LISTENER-LAYERED-DESIGN.md) and [doc/architecture/LISTENER-MIGRATION-GUIDE.md](doc/architecture/LISTENER-MIGRATION-GUIDE.md).

## Build and Development Commands

```bash
# Build
mvn clean compile
mvn clean install

# Test
mvn test                                    # unit tests (all modules)
mvn verify                                  # integration tests (failsafe plugin)
mvn test -Dspring.profiles.active=test      # with test profile

# Single test class
mvn test -pl gb28181-client -Dtest=CancelRequestProcessorTest

# Single test method
mvn test -pl gb28181-client -Dtest=CancelRequestProcessorTest#methodName

# Single module
mvn clean install -pl gb28181-test

# Protocol-purity check (CI gate, run during `mvn verify`)
bash scripts/check-sip-common-purity.sh
```

## Module Structure

```
sip-proxy
├── sip-common          # Core SIP protocol stack (JAIN-SIP wrapper, listeners, caching, metrics)
├── gb28181-common      # GB28181 data models (JAXB XML entities, no business logic)
├── gb28181-client      # Device client (ClientSendCmd, inbound request/response processors)
├── gb28181-server      # Platform server (ServerSendCmd, inbound request/response processors)
└── gb28181-test        # Integration tests and runnable examples
```

Dependency order: `sip-common` ← `gb28181-common` ← `gb28181-client` / `gb28181-server` ← `gb28181-test`

## Architecture

### SIP Message Processing Pipeline

```
SIP Message
  → AbstractSipListener          (unified event dispatch, TraceId propagation)
  → XXXRequestProcessor          (message type: REGISTER, INVITE, MESSAGE, NOTIFY, BYE…)
  → XXXRequestSubProcessor       (MESSAGE only: routes by GB28181 cmdType)
  → XXXRequestHandler            (business logic implementation)
```

### Outbound Commands

- **`ClientCommandSender`** / **`ServerCommandSender`** — strategy-pattern command senders for outbound SIP messages. `ServerCommandSender` requires `DeviceSessionCache` to look up device sessions.

### Dialog-Aware Outbound (1.7.0+)

INVITE and SUBSCRIBE go through **stateful** transmission (`SipMessageTransmitter.transmitStateful` / `transmitStatefulPreRegister`), which uses a `ClientTransaction` so JAIN-SIP auto-creates a `Dialog`. The dialog is recorded in `DialogRegistry` (in-process, keyed by `callId`, with `kind=INVITE|SUBSCRIBE` and `expiresAtMs`).

- **BYE** must reference an existing dialog: `deviceBye(callId)` / `sendByeCommand(callId)` / `doByeRequest(callId)`. No dialog → `DialogNotFoundException`. Dialog cleanup: `AbstractSipListener.processDialogTerminated` → `DialogRegistry.remove` (INVITE primary path).
- **SUBSCRIBE refresh / unsubscribe** must also be dialog-aware: `refreshSubscribe(callId, expires)` / `unsubscribe(callId)` on both senders, plus `SipSender.doSubscribeRefresh(callId, content, expires)` and `CommandContext.forSubscribeRefresh(...)`.
- **`DialogRegistryCleaner`** (`@Scheduled` 60s) sweeps expired SUBSCRIBE entries because RFC 6665 §4.4.1 case 3 has no `DialogTerminatedEvent`.
- **INVITE 200 OK ACK** uses `dialog.sendAck` (see `InviteResponseProcessor.sendAck`), symmetric with BYE.

**Implication for new outbound logic:** if you're adding any in-dialog request (re-INVITE, UPDATE, INFO, NOTIFY-from-server, etc.), use the stateful path and look up the dialog from `DialogRegistry` — never construct a fresh `From`/`To` pair without the to-tag.

### Event Bus & Listener API

**v1.5.0+**: Business接入有两种等价方式：


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lunasaw/Sip-Proxy](https://github.com/lunasaw/Sip-Proxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
