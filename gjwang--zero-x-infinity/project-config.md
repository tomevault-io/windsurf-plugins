---
trigger: always_on
description: > **Top-Level AI Agent Configuration for Zero X Infinity**
---

# AGENTS.md

> **Top-Level AI Agent Configuration for Zero X Infinity**
>
> This file is the primary entry point for all AI agents working on this codebase.

---

## 🎯 Project Overview

**Zero X Infinity** is a production-grade cryptocurrency matching engine achieving **1.3M orders/sec** on a single core.

| Aspect | Details |
|--------|---------|
| **Language** | Rust |
| **Architecture** | LMAX Disruptor-style Ring Buffer Pipeline |
| **Database** | PostgreSQL (config) + TDengine (trading data) |
| **API** | REST + WebSocket with Ed25519 authentication |
| **Current Phase** | 0x10 Web Frontend (In Progress) |

---

## 📖 Essential Reading
> [!IMPORTANT]
> **CRITICAL**: You MUST read [Testing Pitfalls](./docs/src/testing-pitfalls.md) BEFORE writing or running any test scripts. Violating the rules in this file (especially `pkill -f` and Stale Binary Trap) will cause severe system instability.

Before making any changes, AI agents MUST read:

| Document | Purpose |
|----------|---------|
| [Project Roadmap](./docs/src/0x00-mvp-roadmap.md) | Current progress and planned phases |
| [API Conventions](./docs/standards/api-conventions.md) | REST API standards |
| [ID Specification](./docs/src/0x0A-b-id-specification.md) | Identity addressing rules |
| [Development Guidelines](./docs/standards/development-guidelines.md) | Coding standards |
| [Testing Pitfalls](./docs/src/testing-pitfalls.md) | Common traps and best practices |
| [Money Type Safety](./docs/standards/money-type-safety.md) | **CRITICAL**: Balance & Scaling rules |

---

## 🧭 Core Principle: Stay on Track

> **AI agents easily lose direction during complex tasks. To prevent this, ALL work MUST follow the universal methodology below, plus role-specific techniques.**

---

### Universal Methodology (ALL Roles)

Every AI agent, regardless of role, follows this pattern:

```
┌─────────────────────────────────────────────────────────────────────┐
│               UNIVERSAL WORK METHODOLOGY                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  PHASE 1: TOP VIEW (Before ANY work)                               │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ • State the goal in ONE sentence                            │   │
│  │ • Define what "DONE" looks like                             │   │
│  │ • List acceptance criteria as checkboxes                    │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  PHASE 2: ACCEPTANCE CHECKLIST (Before execution)                  │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ - [ ] Criterion 1: [specific, measurable]                   │   │
│  │ - [ ] Criterion 2: [specific, measurable]                   │   │
│  │ - [ ] Criterion N: [specific, measurable]                   │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  PHASE 3: EXECUTE (With continuous alignment)                      │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ • Before each action: "Does this serve the goal?"           │   │
│  │ • After each step: Update checklist                         │   │
│  │ • If drifting: STOP → Re-read goal → Realign                │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  PHASE 4: VERIFY (Before delivery)                                 │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ • Check EVERY acceptance criterion: ✅ or ❌                 │   │
│  │ • If ❌: Fix or document exception                          │   │
│  │ • Compare result against original goal                      │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  PHASE 5: DELIVER (Complete handover)                              │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ • Delivery summary: What was done                           │   │
│  │ • Acceptance status: All criteria passed?                   │   │
│  │ • Handover notes: What next session needs to know           │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Universal Checklist Template

```markdown
## Task: [Original Request]

### 🎯 Goal (ONE sentence)
[What success looks like]

### ✅ Acceptance Criteria (before starting)
- [ ] [Criterion 1]
- [ ] [Criterion 2]
- [ ] [Criterion 3]

### 📋 Progress Tracking (during execution)
- [x] Completed step
- [ ] Current step ← 
- [ ] Pending step

### ❌ Out of Scope
- [Explicitly excluded]

### 📦 Delivery Summary (after completion)
- **Status**: [All criteria met / Partial / Blocked]
- **Result**: [What was delivered]
- **Handover**: [What next session needs to know]
```

---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gjwang/zero_x_infinity](https://github.com/gjwang/zero_x_infinity) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
