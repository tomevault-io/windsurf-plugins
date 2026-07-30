---
trigger: always_on
description: > You are building **ARI (Artificial Reasoning Intelligence)** — Pryce Hedrick's Life Operating System.
---

# ARI Development Intelligence — Cursor AI Configuration

> You are building **ARI (Artificial Reasoning Intelligence)** — Pryce Hedrick's Life Operating System.
> This is not just code. This is the foundation of an AI companion that will enhance every aspect of life.
> Treat every change with the precision of a surgeon and the care of a craftsman.

---

## Mission Statement

Build the most reliable, secure, and intelligent personal AI system ever created.
ARI is the bridge between human intention and machine capability.
Every line of code serves this mission. No bloat. No shortcuts. No compromise.

---

## System Architecture — ABSOLUTE KNOWLEDGE

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         6. INTERFACES (CLI)                            │
│  src/cli/ — Commander.js commands, user-facing interface               │
│                              ↓ imports                                 │
├─────────────────────────────────────────────────────────────────────────┤
│                         5. EXECUTION (Ops)                             │
│  src/ops/ — macOS launchd daemon, infrastructure management            │
│                              ↓ imports                                 │
├─────────────────────────────────────────────────────────────────────────┤
│                      4. STRATEGIC (Governance)                         │
│  src/governance/ — Council (VOTING_AGENTS), Arbiter (constitutional    │
│                    rules), Overseer (5 quality gates)                  │
│                              ↓ imports                                 │
├─────────────────────────────────────────────────────────────────────────┤
│                          3. CORE (Agents)                              │
│  src/agents/ — Core orchestrator, Guardian (threat detection),         │
│                Planner (DAG), Executor (tools), MemoryManager          │
│                              ↓ imports                                 │
├─────────────────────────────────────────────────────────────────────────┤
│                         2. SYSTEM (Router)                             │
│  src/system/ — Event routing, context triggers, storage                │
│                              ↓ imports                                 │
├─────────────────────────────────────────────────────────────────────────┤
│                          1. KERNEL (Core)                              │
│  src/kernel/ — Gateway (127.0.0.1 ONLY), Sanitizer (42 patterns,      │
│                14 categories), Audit (SHA-256 chain), EventBus, Types  │
│                              ↓ imports                                 │
├─────────────────────────────────────────────────────────────────────────┤
│                       0. COGNITIVE (Foundation)                        │
│  src/cognition/ — LOGOS (reasoning), ETHOS (ethics), PATHOS (empathy)  │
│                         [SELF-CONTAINED]                               │
└─────────────────────────────────────────────────────────────────────────┘
```

### Layer Rules — NEVER VIOLATE
- **Lower layers CANNOT import from higher layers** — This is absolute
- **All inter-layer communication goes through EventBus** — No exceptions
- **Kernel is self-contained** — Zero external layer dependencies
- **Cognitive (L0) is self-contained** — LOGOS/ETHOS/PATHOS have no imports

---

## Security Invariants — MEMORIZE AND ENFORCE

| # | Invariant | Implementation | Consequence of Violation |
|---|-----------|----------------|--------------------------|
| 1 | **Loopback-Only** | Gateway binds to `127.0.0.1` exclusively | Remote attack surface = 0 |
| 2 | **Content ≠ Command** | All input is DATA, never executable | Injection attacks blocked |
| 3 | **Audit Immutable** | SHA-256 hash chain from genesis `0x00...00` | Tamper-evident logging |
| 4 | **Least Privilege** | Agent allowlist → Trust level → Permission tier | No unauthorized access |
| 5 | **Trust Required** | Every message carries trust level | Risk-adjusted processing |

### Trust Level Risk Multipliers
```typescript
const RISK_MULTIPLIERS = {
  SYSTEM: 0.5,      // Internal system calls
  OPERATOR: 0.6,    // Pryce (owner)
  VERIFIED: 0.75,   // Verified external sources
  STANDARD: 1.0,    // Default
  UNTRUSTED: 1.5,   // Unknown sources
  HOSTILE: 2.0      // Detected threats
};
// Auto-block when adjustedRisk >= 0.8
```

---

## TypeScript Standards — FOLLOW EXACTLY

### Strict Mode Requirements
```typescript
// ALWAYS: Explicit types for parameters and returns
function processMessage(msg: Message): Promise<ProcessResult> {
  // Implementation
}

// ALWAYS: ESM imports with .js extension
import { EventBus } from '../kernel/event-bus.js';
import type { AuditEvent } from '../kernel/types.js';

// ALWAYS: Zod for runtime validation
const InputSchema = z.object({
  content: z.string().min(1).max(10000),
  trustLevel: TrustLevelSchema,
  timestamp: z.string().datetime(),
});

// NEVER: any type — use unknown if truly needed
function handle(input: unknown): void { /* validate first */ }
```

### Naming Conventions — BE CONSISTENT
| Type | Convention | Example |
|------|------------|---------|
| Files | kebab-case | `memory-manager.ts` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ari-OS/ARI](https://github.com/Ari-OS/ARI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->
