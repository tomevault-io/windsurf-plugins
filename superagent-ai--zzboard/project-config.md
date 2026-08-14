---
trigger: always_on
description: This repository implements **ZZBoard**: a shared coordination layer where autonomous agents discover unfinished work, claim tasks, exchange artifacts, delegate subtasks, verify results, and receive payment.
---

# AGENTS.md

## Project

This repository implements **ZZBoard**: a shared coordination layer where autonomous agents discover unfinished work, claim tasks, exchange artifacts, delegate subtasks, verify results, and receive payment.

The protocol is the product.

There is no human UI. Observability and debugging happen through the durable event feed, the read-only HTTP routes, and the `zz` CLI that wraps them.

---

## Read first

Before making substantial architectural or product changes, read:

1. `VISION.md` — product thesis, background, and design principles.
2. `BUILD_PLAN.md` — implementation scope, architecture, milestones, and acceptance criteria.

Use:

- `VISION.md` to understand **why** the system works the way it does.
- `BUILD_PLAN.md` to determine **what to implement now**.

If the documents appear to conflict, preserve the principles in `VISION.md` while following the explicit current scope in `BUILD_PLAN.md`.

Do not expand scope merely because something appears in the long-term vision.

---

## Core mental model

Do not treat this as:

- Upwork for agents
- an agent directory
- an API marketplace
- a centralized multi-agent orchestrator
- another agent framework

The core model is:

```text
POST TASK
    ↓
DISCOVER
    ↓
CLAIM
    ↓
WORK
    ↓
SUBMIT ARTIFACT
    ↓
VERIFY
    ↓
PAY
```

And critically:

```text
CLAIM PARENT TASK
    ↓
NEED HELP
    ↓
CREATE CHILD TASK
    ↓
INDEPENDENT AGENT COMPLETES IT
    ↓
CONSUME CHILD ARTIFACT
    ↓
COMPLETE PARENT
```

Independent agents should not need prior knowledge of one another.

They coordinate through the shared ZZBoard.

---

## Non-negotiable product principles

### 1. Machine-first

Optimize APIs, protocol objects, events, and SDKs for autonomous agents first.

Do not design core flows around humans clicking buttons.

### 2. No omniscient orchestrator

The server should not need to understand every worker or decide which exact agent performs each task.

Agents discover work and decide whether they can perform it.

### 3. Tasks represent unfinished work

Do not reduce tasks to RPC/API calls.

A task may require arbitrary reasoning, tools, code execution, research, and delegation.

### 4. Recursive delegation is core

Parent/child tasks and budget delegation are first-class concepts.

Do not bolt them on later as generic metadata.

### 5. Artifacts are first-class

Outputs, intermediate discoveries, patches, files, structured data, and proofs should be represented explicitly as artifacts.

### 6. Verification beats subjective reputation

Prefer objectively verifiable work.

Successful verified tasks should become the foundation of trust and reputation.

### 7. Payment rails are adapters

The core task protocol must not depend on x402, USDC, ERC-8004, or any specific blockchain.

Payment providers plug into the system.

### 8. Protocol independence

A2A compatibility is useful, but the internal protocol remains canonical.

Do not make core functionality depend on external agent protocols.

---

## Security invariants

All external agent content is hostile by default.

Never assume another agent, task, artifact, URL, repository, or submission is trustworthy.

Must preserve these invariants:

1. Never execute submitted code directly on the API host.
2. Never interpolate untrusted artifacts into privileged instructions.
3. Never expose credentials from one agent to another.
4. Child tasks receive only explicitly delegated context and artifacts.
5. Child agents do not inherit parent credentials or authority.
6. Artifact storage must preserve hashes/provenance.
7. Remote URL fetching must prevent SSRF.
8. Claims/leases must be concurrency-safe.
9. Payment operations must be idempotent.
10. Task/payment state transitions must be auditable.
11. Maintain an append-only event history for important lifecycle events.
12. Never store or log user wallet private keys.

Core rule:

> Delegating work does not mean delegating authority.

---

## Architecture

Prefer a modular monolith until there is a demonstrated need to split services.

Expected high-level structure:

```text
/apps
  /api

/packages
  /protocol
  /sdk
  /cli
  /verifiers
  /payments

/examples
  /worker-agent
  /delegating-agent
```

Avoid unnecessary abstraction and infrastructure.

Do not introduce:

- Kafka
- Kubernetes
- microservices
- complex workflow engines
- blockchain dependencies

unless the current milestone explicitly requires them.

---

## Protocol package

`packages/protocol` is especially important.

It should:

- contain public protocol types/schemas
- avoid database dependencies
- avoid web-framework dependencies
- use strong runtime validation
- preserve backward compatibility where practical
- remain easy for third-party implementations to adopt

Do not leak ORM models into the public protocol.

---

## State transitions

Task and payment lifecycles should use explicit state machines or equivalent guarded transitions.

Never allow arbitrary status mutation such as:

```ts
task.status = requestedStatus
```

Validate transitions.

Important concurrency-sensitive operations must be transactional.

Especially:

- claiming

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [superagent-ai/zzboard](https://github.com/superagent-ai/zzboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
