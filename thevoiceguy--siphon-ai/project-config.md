---
trigger: always_on
description: Operating instructions for AI coding agents (Claude Code, Cursor, etc.) working in this repo.
---

# CLAUDE.md

Operating instructions for AI coding agents (Claude Code, Cursor, etc.) working in this repo.
**Read this file first, before any task. Re-check relevant sections before non-trivial changes.**

For the *what* and *why* of this project, see `docs/DEV_PLAN.md`. This file is the *how*.

---

## 1. What SiphonAI Is (One Paragraph)

SiphonAI is a SIP-to-WebSocket media bridge written in Rust. It accepts inbound SIP calls (either as a trunk endpoint or as a registered phone on a PBX), streams the call's audio over a WebSocket to a developer-supplied server, and plays audio received back over that WebSocket into the call. **It does not contain any AI code.** AI is the WebSocket server's job. SiphonAI's job is the bridge: SIP signaling, RTP media, codec handling, jitter, barge-in, DTMF, hold/transfer — all wrapped in a clean WS protocol.

**If you find yourself writing code that calls an AI provider (OpenAI, Anthropic, Deepgram, ElevenLabs, etc.), stop. That's the wrong layer. Ask the user.**

---

## 2. Architecture in 60 Seconds

```
PBX/Trunk ──SIP/RTP──► SiphonAI ──WebSocket──► Developer's WS server (BYO AI)
                          │
                          ├── siphon-rs        (SIP stack — external dep)
                          ├── forge-media      (RTP, codecs, SDP — external dep)
                          └── our code         (orchestration + WS protocol)
```

Two upstream Rust libraries do most of the heavy lifting:

- **siphon-rs** (https://github.com/thevoiceguy/siphon-rs) — RFC 3261 SIP stack. UAS, UAC, transactions, dialogs, REGISTER, REFER, auth.
- **forge-media** (https://github.com/thevoiceguy/forge-media) — RTP/RTCP, codecs (G.711/Opus), SDP (`forge-sdp`), jitter buffer, DTMF, audio injection, VAD.
- **hep-rs** (new, owned by us) — HEP3 codec, transport, `HepSink` trait. Used by all three of siphon-rs (SIP messages), forge-media (RTCP/QoS), and siphon-ai (logs/CDRs/events) to ship observability data to Homer.

**All three are owned by the same author as SiphonAI.** If you find a missing capability in any, the right answer is often a small PR upstream, not a workaround here. Ask the user before doing this.

**SiphonAI itself is the thin orchestration layer + the WebSocket protocol.** Most code in this repo is glue, state machines, config, and the WS protocol implementation.

For the full design rationale, see `docs/DEV_PLAN.md`.

---

## 3. Workspace Layout

```
siphon-ai/
├── Cargo.toml                    # workspace root
├── crates/
│   ├── admin-api-types/          # Admin API wire shapes (shared: daemon serializes, sightglass deserializes)
│   ├── core/                     # CallController, state machine, glue
│   ├── bridge/                   # WS client + protocol types + audio bridging
│   ├── sip-glue/                 # Adapter: siphon-rs events → core
│   ├── media-glue/               # Adapter: forge-engine → core (the "tap")
│   ├── routes/                   # Route matching engine (TOML dialplan → bridge config)
│   ├── cdr/                      # CDR generation (JSON), file sink, webhook sink
│   ├── webhooks/                 # Out-of-band lifecycle webhooks (HTTP POST)
│   ├── config/                   # TOML config + validation + reload
│   ├── protocol-testkit/         # siphon-ai-testkit: WS protocol conformance harness
│   └── telemetry/                # tracing + metrics + HEP wiring + admin/health endpoints
├── bins/
│   ├── sightglass/               # Terminal operator console (crate: siphon-ai-sightglass; docs/design/DESIGN_SIGHTGLASS.md)
│   └── siphon-ai/                # The daemon binary
├── sdks/
│   ├── python/                   # Server SDK (siphon-ai-server on PyPI-style layout)
│   └── typescript/               # Server SDK (siphon-ai-server npm-style layout)
├── examples/
│   ├── echo-ws-server-python/    # Reference WS server (echo) — built on sdks/python
│   ├── echo-ws-server-node/      # Same in Node — built on sdks/typescript
│   ├── openai-realtime-bridge-py/  # Reference: OpenAI Realtime bridge
│   └── homer-stack/              # Local Homer + dashboards via docker-compose
├── docker/
├── docs/
│   ├── DEV_PLAN.md               # The plan — read for context
│   ├── PROTOCOL.md               # WS protocol v1 spec (canonical)
│   ├── CONFIG.md                 # TOML config reference (every field documented)
│   ├── DIALPLAN.md               # Route matching semantics with examples
│   ├── HEP.md                    # HEP integration setup and Homer correlation
│   ├── DEPLOY.md
│   ├── REGISTRATION.md
│   └── design/                   # design notes + historical dev plans (internal)
└── test-harness/
    ├── sipp-scenarios/           # SIPp test scripts
    ├── hep-collector-stub/       # Tiny HEP3 receiver for tests
    └── interop/                  # Lab notes for Asterisk/CUCM
```

### Where Things Go

| If you're adding... | Put it in... |
|---|---|
| WS protocol message types or serialization | `crates/bridge/src/protocol.rs` |
| WS connection lifecycle, reconnect logic | `crates/bridge/src/conn.rs` |
| Mapping SIP dialog events to call state | `crates/sip-glue/` |
| Mapping forge audio frames to/from WS | `crates/media-glue/` |
| The call state machine itself | `crates/core/src/call.rs` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thevoiceguy/siphon-ai](https://github.com/thevoiceguy/siphon-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-07 -->
