---
trigger: always_on
description: This is an autonomous development workspace. Agents build agents.
---

# AGENTS.md — ⚡ AT Protocol Agent Network

This is an autonomous development workspace. Agents build agents.

## First Session

1. Read this file completely
2. Read `PI-POC.md` — the implementation plan with security gates
3. Read `docs/O11Y.md` — observability architecture
4. Check `.agents/skills/` for available skills
5. Check the hive for current work: `swarm hive cells --status open`

## What We're Building

A decentralized agent communication and memory network on Cloudflare, using Pi as the agent runtime.

**Core properties:**
- **Private by default** — All memories encrypted (envelope encryption, X25519)
- **Pi runtime** — Tool-calling, streaming, self-extending agents
- **Cloudflare native** — Durable Objects, D1, R2, Vectorize, Queues
- **AT Protocol inspired** — DIDs, lexicons, firehose patterns

## Phase Overview

| Phase | Focus | Security Gate |
|-------|-------|---------------|
| 1 | Encrypted single agent | All memories encrypted, no plaintext in D1 |
| 2 | Semantic memory | Search on embeddings, decrypt only on retrieval |
| 3 | Multi-agent coordination | E2E encryption between agents |
| 4 | Selective sharing | Private by default, public requires explicit action |

## Project Structure

```
atproto-agent-network/
├── AGENTS.md            # You are here
├── PI-POC.md            # Implementation plan with security gates
├── README.md            # Project overview
├── REFERENCES.md        # Key resources (Pi, Cirrus, Cloudflare blogs)
├── docs/
│   ├── O11Y.md          # Observability architecture
│   ├── CONCEPTS.md      # AT Protocol concepts
│   ├── IDENTITY.md      # DID patterns
│   ├── MEMORY.md        # Repository/memory design
│   ├── LEXICONS.md      # Message schemas
│   ├── FIREHOSE.md      # Real-time coordination
│   ├── SECURITY.md      # Encryption patterns
│   └── IMPLEMENTATION.md # Code examples
├── .agents/
│   └── skills/          # Skills for autonomous development
├── apps/
│   ├── network/         # Cloudflare Worker + Durable Objects (API)
│   └── ios/             # Native iOS client (XcodeGen, SwiftUI) [NEW]
└── src/                 # Implementation (to be created)
    ├── agent/           # Pi agent wrapper for Cloudflare DO
    ├── crypto/          # Encryption utilities
    ├── memory/          # D1 + R2 encrypted storage
    ├── comms/           # Inter-agent messaging
    ├── relay/           # Event coordination
    └── cli/             # zap observability CLI
```

## Skills Available

Skills live in `.agents/skills/`. Load a skill when working on its domain:

| Skill | When to Use |
|-------|-------------|
| `cloudflare-do` | Durable Objects, WebSockets, hibernation patterns |
| `pi-agent` | Pi runtime integration, tools, extensions, session trees |
| `envelope-encryption` | X25519, DEK management, envelope encryption |
| `d1-patterns` | D1 schema, encrypted records, queries |
| `vectorize-search` | Semantic search, embeddings, Vectorize API |
| `zap-cli` | Observability CLI, event logging, decision traces |
| `ios-app` | Native iOS client (XcodeGen, SwiftUI, WebSocket feed), turbo integration for iOS build/test |
| `ios-publish` | TestFlight publishing workflow (archive/export/upload/status checks) for `apps/ios` |

## HITL Gates

Human-in-the-loop checkpoints. **Do not proceed past these without approval:**

1. **Phase 1 complete** — Verify all memories encrypted, security audit
2. **Phase 2 complete** — Verify search works without decrypting content
3. **Phase 3 complete** — Verify E2E encryption between agents
4. **Phase 4 complete** — Verify sharing requires explicit action

When you hit a gate, ping the Oracle (Joel) with:
- What was completed
- Security verification results
- Any decisions that need Oracle Context™

## Workflow: The Ralph Loop

### TDD is the Law

**Every feature has tests first. No exceptions.**

```
    RED           GREEN         REFACTOR
     │              │              │
     ▼              ▼              ▼
┌─────────┐   ┌─────────┐   ┌─────────┐
│  Write  │   │  Write  │   │  Clean  │
│ failing │──▶│ minimal │──▶│  code,  │
│  test   │   │  code   │   │ keep    │
│         │   │         │   │ tests   │
└─────────┘   └─────────┘   └─────────┘
```

### Sprint Lifecycle

```
┌─────────────────────────────────────────────────────────────────┐
│                        RALPH LOOP SPRINT                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. PLANNING (#21 template)                                      │
│     └── Select issues from backlog                               │
│     └── Verify TDD instructions complete                         │
│     └── Identify HITL gates                                      │
│                                                                  │
│  2. EXECUTION (ralph_iterate)                                    │
│     └── For each story:                                          │
│         ├── Write tests (RED)                                    │
│         ├── Implement (GREEN)                                    │
│         ├── Refactor                                             │
│         ├── Commit with "Closes #X"                              │

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [joelhooks/atproto-agent-network](https://github.com/joelhooks/atproto-agent-network) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
