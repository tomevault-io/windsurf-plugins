---
trigger: always_on
description: **Read this file first. Every task. Every agent. Every contributor.**
---

# AGENTS.md — EmbedIQ Project North Star

**Read this file first. Every task. Every agent. Every contributor.**

This is the single entry point for understanding EmbedIQ before writing any code,
creating any task, or making any architectural decision.

---

## 1. What EmbedIQ Is (3 Sentences)

EmbedIQ is a fully Apache 2.0 **application framework for embedded and edge** — the layer that sits above the substrate (RTOS,
bare-metal, or Linux) and gives developers clean, reusable structure for building
production firmware and Linux gateway applications.

It provides a message-driven actor model (Functional Blocks), a reusable component
library (cloud, OTA, telemetry, observability), and hardware abstraction that enables
host/Linux simulation without physical hardware.

It is **substrate-agnostic, hardware-vendor independent, and forever free** — with no
cloud-vendor lock-in. Runs on FreeRTOS, bare-metal MCUs, and Linux gateway devices
from the same codebase.

---

## 2. The Four Core Principles (Non-Negotiable)

These are not guidelines. They are binding rules that define EmbedIQ.
Violating any one of them is an architectural defect, not a style issue.

```
PRINCIPLE 1 — Everything is a message (at FB boundaries)
  No Functional Block ever calls another FB directly.
  All cross-FB communication is a typed message through the bus.
  Intra-FB helper function calls are normal C — this rule applies at FB boundaries only.

PRINCIPLE 2 — Everything has a contract
  Every layer has a stable interface header and pluggable implementations.
  You program to the contract. RTOS, BSP, and platform swap beneath it.
  Core headers are the source of truth. No other file overrides them.

PRINCIPLE 3 — Everything is described
  Every module carries machine-readable metadata.
  Every message has a schema (messages.iq).
  Every topology can be exported.

PRINCIPLE 4 — The wrong patterns are structurally visible
  Fixed contracts between layers mean cross-FB coupling, hardware dependencies
  in application code, and untyped data passing cannot be hidden — they fail
  compilation, fail CI, or are obvious in code review.
  Structural discipline is enforced by the architecture, not hoped for.
```

---


## 2A. Message ID Namespace — Agent Must Verify

Before defining any message, verify the ID falls in the correct range.

```
0x0000 – 0x03FF   Core system (framework internal only — never use in your code)
0x0400 – 0x13FF   Official EmbedIQ components (assigned by embediq/embediq)
0x1400 – 0xFFFF   Community / third-party — reserve range in messages_registry.json first
```

**Agent rule:** When generating a new message definition, always state which namespace range you are using and why. If writing a community FB, check messages_registry.json first for range availability.

---

## 2B. Queue Overflow Policy — Agent Must Know

When writing code that publishes messages:

- **HIGH queue full** → sender blocks. Design HIGH publishers to be rare and fast.
- **NORMAL queue full** → oldest message dropped, new message enqueued. Observable event emitted.
- **LOW queue full** → new message dropped. Observable event emitted.

**Agent rule:** Never write code that assumes queue operations always succeed. Check return values. HIGH queue blocking is intentional — a caller that fills HIGH repeatedly has a design problem.

---

## 2C. Boot Phase Model — Agent Must Declare

Every FB must declare its boot_phase. If not declared, default = APPLICATION (Phase 3).

```
EMBEDIQ_BOOT_PHASE_PLATFORM       = 1  // fb_uart, fb_timer, fb_gpio — hardware only
EMBEDIQ_BOOT_PHASE_INFRASTRUCTURE = 2  // fb_nvm, fb_watchdog, fb_cloud — services
EMBEDIQ_BOOT_PHASE_APPLICATION    = 3  // developer FBs (default)
EMBEDIQ_BOOT_PHASE_BRIDGE         = 4  // External FBs, Studio connections
```

**Agent rule:** When generating an FB, always include boot_phase in the config. When writing fb_nvm, fb_watchdog, fb_cloud — use INFRASTRUCTURE. When writing application FBs — use APPLICATION. Never declare a Phase 2 FB with depends_on pointing to a Phase 3 FB.

---
## 3. Layer Model — The Complete Layer Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│  LAYER 4 — COMMERCIAL (future)                                  │
│  EmbedIQ Studio · EmbedIQ Cloud · AI Coder                     │
├─────────────────────────────────────────────────────────────────┤
│  CLIENT SDKs (future)                                           │
│  embediq-python · embediq-js · embediq-rust                     │
├─────────────────────────────────────────────────────────────────┤
│  LAYER 3 — ECOSYSTEM                                            │
│  Bridge daemon · bridge/websocket · bridge/unix_socket          │
│  Community Driver FBs · 3rd-party FB wrappers                   │
├─────────────────────────────────────────────────────────────────┤
│  LAYER 2 — DRIVER FBs (Apache 2.0)                               │
│  fb_uart · fb_timer · fb_gpio · fb_watchdog · fb_nvm             │
│  fb_telemetry · embediq_cfg                                       │
│  fb_cloud_mqtt · fb_ota · fb_provisioning (Phase 2/3, Apache 2.0)│
├─────────────────────────────────────────────────────────────────┤
│  LAYER 1 — FRAMEWORK ENGINE                                     │

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ritzylab-dev/embediq](https://github.com/ritzylab-dev/embediq) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
