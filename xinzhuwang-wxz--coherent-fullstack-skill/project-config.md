---
trigger: always_on
description: This skill prevents these issues by establishing **contracts before code**, **type constraints as AI rules**, and **automated verification**.
---

# Coherent Fullstack Skill

> Make AI code a coherent full-stack software — contract-first, type-driven, convention-enforced

## What is This Skill?

**Coherent Fullstack** is a development methodology and skill designed for AI-assisted full-stack development. It solves the most common failure mode in AI coding: **frontend-backend-database misalignment**.

When you ask AI to build a full-stack app, it typically generates:
- Backend with random field names and types
- Frontend that expects completely different field names
- Database schema that doesn't match either
- APIs that return 404 or 422 errors at runtime

This skill prevents these issues by establishing **contracts before code**, **type constraints as AI rules**, and **automated verification**.

## The Core Problem

### Why AI Fullstack Projects Fall Apart

```
User Request: "Build a user profile API"

AI Backend generates:
┌─────────────────────────────┐
│ POST /auth/register        │
│ Body: { username, fullName }│
│ Returns: { userId, created }│
└─────────────────────────────┘

AI Frontend expects:
┌─────────────────────────────┐
│ POST /api/register          │
│ Body: { name, email }       │
│ Returns: { id, name, xp }    │
└─────────────────────────────┘
```

**Result**: Every API call fails. 422 errors. 404 errors. Runtime type mismatches.

This isn't a model capability problem — it's an **alignment problem**. The AI has no shared source of truth, so it generates independently in each context.

### The Three Failure Modes (from Vibe Coding Research)

| Failure Mode | Description | Consequence |
|-------------|-------------|-------------|
| **Context Loss** | AI forgets earlier decisions after many turns | Drift toward divergent implementations |
| **Assumption Drift** | AI fills gaps with reasonable defaults that compound | Field names, types, paths diverge |
| **Pattern Violations** | AI uses generic best practices instead of project conventions | Inconsistent with existing codebase |

## The 60/40 Rule

The most important shift in AI development isn't "write code faster" — it's **becoming an architecture constraint setter**.

```
┌────────────────────────────────────────────────────────────┐
│                    60/40 DEVELOPMENT RULE                  │
├────────────────────────────────────────────────────────────┤
│                                                            │
│   60% ALIGNMENT           │  40% EXECUTION                 │
│   ─────────────           │  ─────────────                 │
│   • Define boundaries     │   • AI generates code          │
│   • Write contracts       │   • Run tests                  │
│   • Set验收标准           │   • Fix regressions            │
│   • Establish patterns    │                                │
│                            │                                │
│   Once alignment fails,   │   Bottleneck shifts from      │
│   you'll pay back 10x in  │   "writing code" to            │
│   cross-service patches   │   "aligning systems"           │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

### The New "Source Code" Concept

In the AI era, **real source code** isn't Python/TypeScript/Rust — it's **reviewable artifacts**:

1. **Mermaid diagrams** → structural boundaries
2. **OpenAPI / JSON Schema** → interface and type contracts
3. **English Specs / Gherkin** → behavioral constraints
4. **Planning snapshots** → compress discussions into AI-executable context

## 4-Step Workflow

### Step 1: Define Contract (Before Any Code)

```yaml
# contracts/user.yaml
User:
  id: string (UUID)
  name: string (2-50 chars)
  email: string (valid email)
  avatar_url: string | null
  created_at: ISO8601 datetime

POST /api/users:
  request: { name, email, password }
  response: { user: User, token: string }
  errors: 400 (validation), 409 (email exists)

GET /api/users/{id}:
  response: { user: User }
  errors: 404 (not found)
```

### Step 2: Implement Backend

Follow the contract. Every field must match exactly:
- JSON field names
- Types
- Optional vs required
- Error codes

### Step 3: Implement Frontend

Generate TypeScript interfaces from the contract:
```typescript
// types/user.ts
interface User {
  id: string;
  name: string;
  email: string;
  avatar_url: string | null;
  created_at: string;
}
```

### Step 4: Verify Alignment

Run automated checks:
```bash
./scripts/verify-alignment.sh
```

## When to Use This Skill

### ✅ Perfect Fit
- Building new full-stack features from scratch
- Adding backend endpoints that frontend needs
- Extending existing projects with new data models
- Any AI-assisted development where backend/frontend/database must agree

### ⚠️ Less Relevant
- Backend-only or frontend-only projects
- Projects with existing, well-documented APIs
- Simple CRUD apps with no complex relationships

## Quick Start (3 Minutes)

### 0. Install

```bash
# Option 1: Clone as standalone
git clone https://github.com/YOUR_USERNAME/coherent-fullstack-skill.git

# Option 2: Add as submodule to existing project
git submodule add https://github.com/YOUR_USERNAME/coherent-fullstack-skill.git .coherent-fullstack


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xinzhuwang-wxz/coherent-fullstack-skill](https://github.com/xinzhuwang-wxz/coherent-fullstack-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
