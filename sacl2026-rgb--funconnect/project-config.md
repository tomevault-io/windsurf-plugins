---
trigger: always_on
description: **Read this first.** Every agent in this project starts here. This file is the
---

# AGENTS.md — FunConnect Multi-Agent Directive

**Read this first.** Every agent in this project starts here. This file is the
shared context — who we are, what we're building, what's already proven, and
where your own detailed docs live.

---

## 1. What We're Building

**FunConnect** — a device-to-cloud pipeline for classroom IoT. Students connect
microcontrollers (CyberPi, micro:bit, Musebricks) to a live web dashboard
through a single protocol. No MQTT broker. No Python bridge. No laptop relay.

```
CyberPi → ws:// → Cloudflare Edge (TLS termination) ─┐
micro:bit V2 → USB serial → Browser WebSerial relay ─┤
micro:bit V1.5 → WebHID CMSIS-DAP flash (zero-click) │
micro:bit V1.5 → MSD flash (fallback)                 │
CyberPi → Web Serial esptool flash (zero-click) ─────┤
                                                      ▼
                                              Worker → Durable Object (per-device)
                                                       ├── SQLite buffer (hot)
                                                       ├── Madgwick AHRS (~5ms)
                                                       ├── Alarm flush → D1 (cold, queryable)
                                                       └── WSS broadcast → dashboards
```

One API token sees everything. One protocol. One deploy.

**Live URL:** `https://funconnect-v1.funconnect.workers.dev`
**Account:** `CF_ACCOUNT_ID_PLACEHOLDER` (EMAIL_PLACEHOLDER)
**Zone:** `cyberpi.trade` (`CF_ZONE_ID_PLACEHOLDER`)
**Token:** FunConnect (`CF_TOKEN_PLACEHOLDER`)

---

## 2. The Five-Layer Contract (Invariant)

This is the architecture's spine. Every agent's work plugs into one layer.
Swap a layer without changing anything above or below.

| Layer | What | Owned By |
|---|---|---|
| **Transport** | Device speaks WSS + JSON. DO accepts WebSocket. | Firmware + Edge |
| **Storage** | DO-local SQLite (UPSERT telemetry_buffer) → D1 (batch flush, row-ceiling) | Edge |
| **Models** | Madgwick AHRS, signature classification, disturbance corpus | Researcher |
| **Language** | RAG chatbot, prompt engineering, context selection | Agent AI |
| **Interface** | SPA — auth, catalog, dashboard, chat widget | Beauty |

**Rule:** No layer reaches around the contract. Firmware doesn't touch D1.
Beauty doesn't call Madgwick directly. AI doesn't bypass the DO.

---

## 3. Agent Directory

Every agent has a dedicated directory with its own detailed docs. **Read your
own doc before writing code.** Read another agent's doc only when you need to
understand their interface.

| Agent | Directory | Charter Doc | Status |
|---|---|---|---|
| **Alpha** | (root) | `ALPHA.md` | Active — architecture, coordination, non-negotiables |
| **Firmware** | `Firmware/` | `Firmware/FIRMWARE.md` (547 lines) | CyberPi Phase 1–3 done. micro:bit relay + 8 smoke tests + 4 catalog programs delivered. Automation proven. |
| **Edge** | `Edge/` | `Edge/EDGE.md` (992 lines) | Live. UPSERT + dead-alarm fix deployed. py2hex compiler + micro:bit catalog + DAPLink updater shipped. |
| **Researcher** | `Researcher/` | `Researcher/RESEARCHER.md` (1270 lines) | madgwick.ts delivered. 64-signature corpus stable. |
| **Agent AI** | `AI/` | `AI/AI.md` (338 lines) | Live on Qwen 3.7. Keyed reference designed. |
| **Beauty** | `Beauty/` | `Beauty/BEAUTY.md` | Active — catalog UI + MSD flash deployed. webhid-flash.js + WebhidFlashOverlay built (July 20). CyberPi Web Serial integration pending. |
| **Detective** | — | — | Future — architecture audit |
| **Security** | — | — | Future — attack surface, HMAC design |

### How agents interact

- **Alpha** coordinates — assigns work, resolves cross-agent decisions, enforces non-negotiables.
- **Agents own their layer.** Edge doesn't change firmware code. Beauty doesn't touch the DO.
- **When you need context from another agent's domain**, read their charter doc first, then ask Alpha if you still have questions.
- **When your work changes a shared contract** (wire protocol, D1 schema, REST API shape), tell Alpha so the directive can be updated.

---

## 4. Shared Architecture (What Every Agent Must Know)

### 4.1 Wire Protocol (locked — Phase 1–2)

```
DEVICE → DO:
  hello:  {"type":"hello","device_id":"mbot2-01","ts":<epoch_ms>}
  state:  {"type":"state","device_id":"mbot2-01","telemetry":{...},"health":{...}}
  alert:  {"type":"alert","device_id":"mbot2-01","event":"disturbance",
           "accel_peak":<g>,"omega_peak":<rad/s>,"signature":<0-63>,
           "samples":[[ax,ay,az,gx,gy,gz],...],"ts":<epoch_ms>}

DO → DEVICE:
  welcome:     {"type":"welcome","device_id":"mbot2-01"}
  sync:        {"type":"sync","led":false,"doTs":<epoch_ms>}
  ack (state): {"type":"ack","ref":"state","doTs":<epoch_ms>,
                "alert_depth":<N>,"last_flush_ms":<ts>}
  ack (alert): {"type":"ack","ref":"alert"}
  error:       {"type":"error","message":"..."}
  echo:        {"command":"echo","params":{"text":"..."}}
  exec:        {"command":"exec","code":"..."}
  fs_test:     {"command":"fs_test"}

DO → DASHBOARD:
  state:       {"type":"state","device_id":"...","telemetry":{...}}
  alert:       {"type":"alert","device_id":"...","madgwick_json":"..."}

  Alert replay (dashboard connect): Edge sends oldest-first in two

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sacl2026-rgb/FunConnect](https://github.com/sacl2026-rgb/FunConnect) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
