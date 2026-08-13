---
trigger: always_on
description: Shotoku is a local-first authorization layer for AI agents.
---

# AGENTS.md

# Shotoku

Shotoku is a local-first authorization layer for AI agents.

Before an agent performs an action — calling a paid API, using an MCP tool, executing code, sending an email, deploying infrastructure, or spending money — Shotoku evaluates the request, applies policy, and records the decision locally.

Built first with x402. Designed for any payment or action rail.

No custody ever.

## Product One-Liner

Shotoku lets developers require approvals, enforce limits, and record auditable decisions before AI agents act.

## Core Question

Every feature should serve one question:

> Should this agent be allowed to perform this action?

If a feature does not improve authorization, approvals, policy enforcement, explanations, or auditability, it probably does not belong in v1.

---

# Current Repo Structure

Use the current repo structure:

```txt
shotoku/
├── core/
│   └── src/
├── cli/
│   └── src/
├── docs/
├── package.json
├── pnpm-workspace.yaml
└── tsconfig.base.json
```

Do not introduce `packages/core` or `packages/cli` unless explicitly asked. This repo currently uses `core/` and `cli/` directly.

## Packages

- `core/` — authorization, policy, ledger, types, and integrations.
- `cli/` — command-line interface and Ink TUI.
- `docs/` — API docs, decisions, positioning, non-goals, quickstarts.

---

# Brand and Positioning

## What Shotoku Is

Shotoku is:

- Local-first
- Developer-first
- Open-source
- Authorization-first
- Approval-oriented
- Audit-focused
- Simple enough for agent builders and “vibe coders” to understand

## What Shotoku Is Not

Shotoku is not:

- A wallet
- A payment processor
- A payment network
- A custody provider
- A banking product
- An enterprise procurement system
- An ERP
- A generic workflow engine
- A cloud-first compliance suite

## Important Positioning Rule

Do not position Shotoku as only an “agent spending” product.

Spending is the first wedge. Authorization for agent actions is the category.

Good:

```txt
Local-first authorization layer for AI agents.
```

Bad:

```txt
Payment layer for AI agents.
```

Good:

```txt
Built first with x402. Designed for any rail.
```

Bad:

```txt
x402 company.
```

---

# Technical Principles

## Hard Rules

1. Commit every day.
2. Ship something every Friday.
3. No features during launch polish unless critical.
4. Tests before features.
5. No vibe-coded mystery code.
6. Every line should be explainable.
7. TypeScript strict mode stays on.
8. Do not use `any`.
9. Prefer interfaces for object shapes.
10. Keep modules small and focused.
11. No custody logic.
12. No private key storage.
13. No cloud requirement in the core authorization path.

## Engineering Style

- TypeScript strict.
- Prefer pure functions in `core`.
- Avoid side effects except in clearly named I/O modules.
- Separate types from implementation.
- Keep public APIs small.
- Make error messages human-readable.
- Do not expose raw stack traces to CLI users.
- Use `readonly` fields for request/response models.
- Prefer explicit imports from Node built-ins, for example `node:crypto`.

---

# Core API Shape

The core API revolves around `authorize()`.

## Request

```ts
export interface AuthorizeRequest {
  readonly actor: string;
  readonly action: AgentAction;
  readonly resource: string;
  readonly rail?: ExecutionRail;
  readonly amount?: number;
  readonly context?: Record<string, unknown>;
}
```

## Response

```ts
export interface AuthorizeResponse {
  readonly approved: boolean;
  readonly status: AuthorizationStatus;
  readonly reasons: readonly ReasonItem[];
  readonly decisionId: string;
  readonly timestamp: string;
}
```

## Reason Item

```ts
export interface ReasonItem {
  readonly type:
    | "policy_match"
    | "budget_check"
    | "limit_check"
    | "blocked"
    | "escalated";

  readonly text: string;
}
```

## Status

```ts
export type AuthorizationStatus = "approved" | "denied" | "pending_approval";
```

## Execution Rail

```ts
export type ExecutionRail = "x402" | "mcp" | "api" | "code" | "custom";
```

## Agent Action

```ts
export type AgentAction =
  | "purchase"
  | "api_call"
  | "execute_code"
  | "send_email"
  | "mcp_tool"
  | "custom";
```

---

# Desired Core File Structure

Prefer this structure as the project grows:

```txt
core/src/
├── authorize.ts
├── types.ts
├── policy.ts
├── ledger.ts
├── explain.ts
├── errors.ts
└── index.ts
```

## Responsibilities

### `types.ts`

Only domain types and interfaces.

### `authorize.ts`

Public authorization entrypoint.

### `policy.ts`

Pure policy evaluation.

### `ledger.ts`

Local decision storage and audit log.

### `explain.ts`

Human-readable explanations.

### `errors.ts`

Typed internal errors and user-safe error formatting.

### `index.ts`

Public exports only.

Keep `index.ts` boring:

```ts
export * from "./authorize.js";
export * from "./types.js";
```

---

# CLI Principles

The CLI is the screenshot surface. It should look intentional.

## Voice

- Terse.
- Precise.
- No marketing words.
- Present tense.
- Every message should explain what happened and what to do next.

## Output Symbols

Use:

```txt
✓ APPROVED
✗ DENIED
◷ PENDING APPROVAL
```

## Approved Output

```txt
✓ APPROVED  dec_abc123
  • OpenAI is allowlisted
  • Daily budget remaining: $475

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shotoku-dev/shotoku](https://github.com/shotoku-dev/shotoku) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
