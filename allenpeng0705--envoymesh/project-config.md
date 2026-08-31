---
trigger: always_on
description: EnvoyMesh is a **decentralized, peer-to-peer mesh for autonomous AI agents**. Agents discover each other, negotiate tasks, share data, and communicate without any central server or cloud dependency. Identity is cryptographic (Ed25519), messages are signed, security is policy-based, and everything is auditable.
---


---
description: 
alwaysApply: true
---

---
description: 
alwaysApply: true
---

# EnvoyMesh — AGENTS.md

## Project Overview

EnvoyMesh is a **decentralized, peer-to-peer mesh for autonomous AI agents**. Agents discover each other, negotiate tasks, share data, and communicate without any central server or cloud dependency. Identity is cryptographic (Ed25519), messages are signed, security is policy-based, and everything is auditable.

**Core principles:**
- No central account server — identity is self-sovereign (Ed25519 keys → DIDs)
- Local-first / distributed — libp2p direct paths, optional CID-verifiable data
- Security by isolation — Diplomat (network) / Bond Engine (policy) / Brain (model) / Vault (data) separation
- Semantic consistency — typed intents agents can reason about, not opaque bytes
- Observability — JSONL audit with correlation IDs stitching multi-peer flows

**Mobile app (product):** **EnvoyGo** — Flutter thin client in `apps/envoygo/`. Pairs to a home node; UI + JSON-RPC over WebSocket/libp2p. This is what "mobile / phone / iOS / Android" means in this repo.

**Capacitor backup removed:** the Phase 11 `apps/mobile/` + in-process `mobile-node` / `mobile-storage` / `mobile-vault` stack was deleted (it was a backup experiment, not the product path). `packages/mobile-identity` survives as the browser-safe identity used by the Social web build — do **not** remove it.

**Active roadmap:** See `docs/implementation-plan.md`. Phase 45 (web content browsing) and later work target Social (desktop) + **EnvoyGo** (mobile).

---

## Repository Structure

```
EnvoyMesh/
├── apps/
│   ├── node/          # Node.js runtime: CLI, mesh, WebSocket API for Social
│   ├── relay/         # Relay node binary
│   ├── social/        # Social/chat UI (Vite + React) — desktop primary UI
│   │   ├── src/components/  # Header, ErrorBoundary, views/ (Chat, Search, Profile, Settings, etc.)
│   │   ├── src/context/     # NodeStateContext (shared state, event-driven)
│   │   ├── src/lib/         # storage.ts, display.ts, direct-call-client.ts
│   │   └── test/            # Component + context tests (vitest + testing-library)
│   ├── envoygo/       # ★ PRODUCT mobile app — Flutter thin client (pair to home)
│   └── tauri/         # End-user native wrapper: WebView loads Social web UI + spawns Node (no Electron)
├── packages/
│   ├── protocol/      # Core protocol: Zod schemas, payload constructors, canonical JSON
│   ├── identity/      # Ed25519 keys, signing/verification, device certificates, mandates
│   ├── bonds/         # Policy engine: trust tiers, capability gating, mandate authorization
│   ├── network/       # libp2p wrapper: TCP/QUIC, mDNS, DHT, circuit relay, envelope streams
│   ├── vault/         # Local file vault: indexing, chunking, search, path safety
│   ├── models/        # Model router: provider selection, semantic firewall, LiteLLM adapter
│   ├── local-store/   # On-disk persistence: JSONL audit/journal, trust store, peer directory
│   ├── mobile-identity/# Browser-safe pure-JS Ed25519 (noble-curves) — Social web build alias
│   └── api/           # Shared TypeScript interfaces (NodeService, types)
├── docs/              # User stories, scenarios, security model, implementation plan
├── tsconfig.base.json # Shared TS configuration
├── tsconfig.json      # Project references (builds all packages & apps)
├── vitest.config.ts   # Vitest with path aliases matching tsconfig
└── AGENTS.md          # This file
```

### Workspace package dependency graph

```
protocol  (Zod schemas, no deps beyond zod)
   ├── identity      (node:crypto Ed25519 — desktop)
   ├── mobile-identity (@noble/curves — browser-safe identity; Social web build alias)
   ├── vault         (desktop file vault)
   ├── models        (protocol deps only)
   ├── bonds         (protocol deps only)
   └── api           (shared TypeScript interfaces)
local-store       (depends on bonds, identity, protocol)
network           (depends on protocol + libp2p ecosystem)
apps/node         (depends on everything desktop)
apps/social       (React SPA — desktop Social UI)
apps/envoygo      (★ PRODUCT mobile — Flutter thin client → home JSON-RPC)
```

---

## Key Concepts & Architecture

### Identities (three-tier)

| Identity | Derivation | Purpose |
|----------|-----------|---------|
| **Owner** | `envoy:owner:<sha256(pubkey)>` | Long-lived human identity, signs mandates & device certificates |
| **Device** | `envoy:device:<sha256(pubkey)>` | A specific device (laptop, phone, server), authorized by owner |
| **Agent** | `envoy:agent:<sha256(ownerId + agent-pubkey)>` | AI agent running on owner's node, authorized by owner-signed mandate |
| **Peer** | `envoy_<sha256(pubkey)>` | Runtime identity for message signing (lasts as long as the key) |

**Agent identity** (Phase 9): The agent has its own peer ID derived from `sha256(ownerId + agent-pubkey)`. The owner signs a mandate/credential linking the agent to the owner. Peers can verify: "This agent is authorized by `envoy:owner:abc123`."


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [allenpeng0705/EnvoyMesh](https://github.com/allenpeng0705/EnvoyMesh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
