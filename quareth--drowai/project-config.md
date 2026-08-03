---
trigger: always_on
description: Guidance for **LLM coding assistants** working in this repository.
---

# AGENTS.md
Guidance for **LLM coding assistants** working in this repository.

This repo evolves quickly and some docs drift. **Code is the source of truth**: validate behavior in the wired paths before changing docs or making architectural claims.

## Core principles (apply to every task)
1. Think before coding
- Don’t assume. State assumptions explicitly.
- If something is unclear, stop and ask.
- If multiple interpretations exist, present them—don’t pick silently.

2. Simplicity first
- Minimum code that solves the problem. Nothing speculative.
- No abstractions for single-use code.
- If 200 lines could be 50, rewrite it.

3. Surgical changes
- Touch only what the request requires.
- Don’t refactor adjacent code “because you’re there”.
- If your change creates unused imports/vars, remove those (but don’t delete pre-existing dead code unless asked).

4. Goal-driven execution (TDD)
- Define verifiable success criteria.
- Prefer: reproduce with a test → fix → keep the test.

5. Add docstring to front of the each newly created module, which explains purpose and responsibility of the file briefly.
- If you see any file/module that haven't any docstring, understand the module and its purpose and add docstring.

6. Separation of Concerns, Modularity

- Do not create monolithic code, if you see code that violates Separation of Concerns inform the user.

7. Before editing any file, read the first 20 lines. Most of the files have docstring at the start that explains the responsibility of that file. Read that docstring and be sure you are editing the correct file.

## Secure by Design

- Follow Secure by Design principles. If something user requested causes a security breach, notify the user.
# Principles: Least Privilege, Depth in Defense, Separation of Duties, Segmentations

## Architectural boundaries (Clean Architecture-ish)
Use these boundaries as a *guide* (the codebase isn’t perfectly layered):

┌─────────────────────────────────────┐
│ Presentation                        │  FastAPI routers, HTTP/WebSocket adapters
├─────────────────────────────────────┤
│ Application                         │  Orchestration/services, workflows
├─────────────────────────────────────┤
│ Domain                              │  Task/user concepts, state machines
├─────────────────────────────────────┤
│ Infrastructure                      │  DB, Docker, external APIs
└─────────────────────────────────────┘

Rules of thumb:
- Dependencies point inward.
- Keep routers thin; put orchestration in `backend/services/`.
- Keep control-plane, data-plane, and execution-plane responsibilities separate.
- Do not bypass the runtime-provider boundary. Control-plane code authorizes, records, and dispatches; execution-plane code runs task work through local Docker or managed runner providers.

                          TASK-ISOLATED EXECUTION MODEL

+---------------------------------------------------------------------------------------+
|                             Shared Control Plane (Global)                             |
| AuthN/AuthZ | API Routing | Tenant/Task Registry | Orchestrator | Stream Bus          |
| Runner Control | Runtime Provider Dispatch                                             |
|                                                                                       |
| Contract: runtime side effects use tenant_id + task_id + runtime identity;          |
| streams remain task-keyed after tenant/user authorization.                          |
+-----------------------------------+-------------------------------+-------------------+
                                    |                               |
                                    v                               v

                    +--------------------------------+   +--------------------------------+
                    |        Task Context A          |   |        Task Context B          |
                    |        (task_id = A)           |   |        (task_id = B)           |
                    |--------------------------------|   |--------------------------------|
                    | - Task state                   |   | - Task state                   |
                    | - Tenant/user scope            |   | - Tenant/user scope            |
                    | - Workspace refs/files         |   | - Workspace refs/files         |
                    | - Runtime placement/provider   |   | - Runtime placement/provider   |
                    | - Task event stream            |   | - Task event stream            |
                    | - Task interrupt state         |   | - Task interrupt state         |
                    | - Task approval/resume channel |   | - Task approval/resume channel |
                    +--------------------------------+   +--------------------------------+

Execution Plane (per task): local Docker runtime OR managed runner runtime.

Isolation Rules (applies to all task features):
1) Read/Write scope is task-local (A cannot read/write B task state or runtime channels).
2) Control actions are task-bound (an action for A is resolved only in A context).
3) Interrupt/approval flow is task-bound (A interrupt lifecycle is independent from B).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [quareth/drowAI](https://github.com/quareth/drowAI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
