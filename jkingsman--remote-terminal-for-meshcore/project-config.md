---
trigger: always_on
description: **NEVER make git commits.** A human must make all commits. You may stage files and prepare commit messages, but do not run `git commit`.
---

# RemoteTerm for MeshCore

## Important Rules

**NEVER make git commits.** A human must make all commits. You may stage files and prepare commit messages, but do not run `git commit`.

If instructed to "run all tests" or "get ready for a commit" or other summative, work ending directives, run:

```bash
./scripts/quality/all_quality.sh
```

This is the repo's end-to-end quality gate. It runs backend/frontend autofixers first, then type checking, tests, and the standard frontend build. All checks must pass green, and the script may leave formatting/lint edits behind.

## Overview

A web interface for MeshCore mesh radio networks. The backend connects to a MeshCore-compatible radio over Serial, TCP, or BLE and exposes REST/WebSocket APIs. The React frontend provides real-time messaging and radio configuration.

**For detailed component documentation, see these primary AGENTS.md files:**
- `app/AGENTS.md` - Backend (FastAPI, database, radio connection, packet decryption)
- `frontend/AGENTS.md` - Frontend (React, state management, WebSocket, components)

Ancillary AGENTS.md files which should generally not be reviewed unless specific work is being performed on those features include:
- `app/fanout/AGENTS_fanout.md` - Fanout bus architecture (MQTT, bots, webhooks, Apprise, SQS)
- `frontend/src/components/visualizer/AGENTS_packet_visualizer.md` - Packet visualizer (force-directed graph, advert-path identity, layout engine)

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                         Frontend (React)                        │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────────────┐ │
│  │ StatusBar│  │ Sidebar  │  │MessageList│  │  MessageInput   │ │
│  └──────────┘  └──────────┘  └──────────┘  └──────────────────┘ │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │      CrackerPanel (global collapsible, WebGPU cracking)    │ │
│  └────────────────────────────────────────────────────────────┘ │
│                           │                                     │
│                    useWebSocket ←──── Real-time updates         │
│                           │                                     │
│                      api.ts ←──── REST API calls                │
└───────────────────────────┼─────────────────────────────────────┘
                            │ HTTP + WebSocket (/api/*)
┌───────────────────────────┼──────────────────────────────────────┐
│                      Backend (FastAPI)                           │
│  ┌──────────┐  ┌──────────┐  ┌──────────────┐  ┌────────────┐    │
│  │ Routers  │→ │ Services │→ │ Repositories │→ │  SQLite DB │    │
│  └──────────┘  └──────────┘  └──────────────┘  └────────────┘    │
│        ↓                         │                ┌───────────┐  │
│  ┌──────────────────────────┐    └──────────────→ │ WebSocket │  │
│  │ Radio runtime seam +     │                     │  Manager  │  │
│  │ RadioManager lifecycle   │                     └───────────┘  │
│  │ / event adapters         │                                    │
│  └──────────────────────────┘                                    │
└───────────────────────────┼──────────────────────────────────────┘
                            │ Serial / TCP / BLE
                     ┌──────┴──────┐
                     │ MeshCore    │
                     │   Radio     │
                     └─────────────┘
```

## Feature Priority

**Primary (must work correctly):**
- Sending and receiving direct messages and channel messages
- Accurate message display: correct ordering, deduplication, pagination/history loading, and real-time updates without data loss or duplicates
- Accurate ACK tracking, repeat/echo counting, and path display
- Historical packet decryption (recovering incoming messages using newly-added keys)
- Outgoing DMs are stored as plaintext by the send endpoint — no decryption needed

**Secondary:**
- Channel key cracker (WebGPU brute-force)
- Repeater management (telemetry, CLI commands, ACL)

**Tertiary (best-effort, quality-of-life):**
- Raw packet feed — a debug/observation tool ("radio aquarium"); interesting to watch or copy packets from, but not critical infrastructure
- Map view — visual display of node locations from advertisements
- Network visualizer — force-directed graph of mesh topology
- Fanout integrations (MQTT, bots, webhooks, Apprise, SQS) — see `app/fanout/AGENTS_fanout.md`
- Read state tracking / mark-all-read — convenience feature for unread badges; no need for transactional atomicity or race-condition hardening

## Error Handling Philosophy

**Background tasks** (WebSocket broadcasts, periodic sync, contact auto-loading, etc.) use fire-and-forget `asyncio.create_task`. Exceptions in these tasks are logged to the backend logs, which is sufficient for debugging. There is no need to track task references or add done-callbacks purely for error visibility. If there's a convenient way to bubble an error to the frontend (e.g., via `broadcast_error` for user-actionable problems), do so, but this is minor and best-effort.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jkingsman/Remote-Terminal-for-MeshCore](https://github.com/jkingsman/Remote-Terminal-for-MeshCore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
