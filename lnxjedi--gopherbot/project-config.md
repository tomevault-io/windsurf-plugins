---
trigger: always_on
description: This file is the authoritative operating procedure for AI agents working in this repository.
---

# AGENTS.md — Gopherbot

This file is the authoritative operating procedure for AI agents working in this repository.

If any ad-hoc instruction conflicts with this file, this file wins.

## Authority Boundary

- `AGENTS.md` is the single source of policy and required process.
- Skills may provide workflows/templates, but must not redefine repository policy.

## Local Tool Availability

- If a task requires a local tool, command, interpreter, or development utility that is not installed or not available on `PATH`, pause and ask the project owner to install it.
- Do not install missing local tools, vendor replacements, or silently substitute a different workflow unless the project owner explicitly approves that change.
- For `.lua`, `.js`, `.gsh`, and interpreted `.go` extension work, standalone language runtimes are not required. Use `./gopherbot syntax` and `./gopherbot script`; see `aidocs/TESTING_CURRENT.md` and `aidocs/INTERPRETERS.md`.

## Phase 0 — Orientation (Two-Tier Model)

### Tier A: Default Orientation (required for all tasks)

Before proposing or implementing changes, read:
1. `aidocs/README.md`
2. `aidocs/COMPONENT_MAP.md`

Then load only the canonical docs needed for the task scope.

### Tier B: Escalated Orientation (hard requirement when triggered)

In addition you must run full architecture preflight before coding when **any** trigger applies.

Read in order:
1. `aidocs/STARTUP_FLOW.md`
2. `aidocs/TESTING_CURRENT.md`

Then summarize in your own words:
- core architectural invariants
- startup ordering constraints
- connector assumptions
- message routing model
- identity model

### Escalation Triggers (hard)

Escalated orientation is mandatory if a change touches or may affect:
- startup/config load order (`bot/start.go`, `bot/bot_process.go`, `bot/config_load.go`, `bot/conf.go`)
- message routing/pipeline ordering (`bot/dispatch.go`, `bot/run_pipelines.go`, scheduler flow)
- connector runtime/behavior (`connectors/*`, connector runtime orchestration)
- identity/authz semantics (username mapping, roster gates, authorization/elevation)
- root/default robot config structure (`conf/robot.yaml`, `robot.skel/conf/robot.yaml`)
- cross-protocol behavior/contracts
- privilege separation or task execution (`bot/privsep.go`, `bot/calltask.go`, `bot/task_execution.go`)
- user permission checks, admin/auth/elevation logic (`bot/available.go`, `bot/authorize.go`, `bot/elevate.go`)
- pre-pipeline user filtering or message context (`bot/handler.go`)

If uncertain, escalate.

## Phase 1 — Impact Analysis (Required for Cross-Cutting Changes)

For changes affecting connectors, routing, startup, configuration, identity, or compatibility:
- produce an Impact Surface Report before modifying code
- include subsystems, invariants, cross-cutting concerns, concurrency, compatibility, docs updates
- do not implement until report is shared, unless explicitly waived by user

## Architectural Invariants

Unless explicitly updated in canonical docs, these must hold:
- startup sequence is deterministic and traceable
- control flow is explicit, not implicit
- shared authorization/business policy remains in engine flows, not connectors
- permission/policy decisions are username-authoritative
- message routing order is preserved within a connector
- configuration precedence is explicit and documented
- engine-shipped extension defaults remain authoritative; custom robot extension config stays delta-only unless behavior is intentionally redefined
- multi-connector isolation prevents cascading failure
- secret access is explicit and scope-based: unprivileged extensions must not discover shared secrets through generic robot methods

## Connector Rules (Critical for Multi-Protocol)

- connectors own transport concerns and protocol-local behavior
- connectors must not bypass shared engine policy/authorization logic
- connectors map transport identity to canonical username deterministically
- cross-protocol identity equivalence is canonical username, not heuristic transport-ID matching
- connector failure isolation must be preserved when multiple connectors are enabled

## Extension Secret Boundary

- secrets may be exposed to an extension only through explicit administrator configuration for that extension, or through memory/brain state owned by that extension's authorized namespace
- unprivileged robot methods must not reveal shared secret-bearing configuration, nor provide indirect discovery of secrets outside the caller's granted scope
- do not add or document extension APIs that return provider registries, parameter-set contents, or other broad configuration objects containing secrets

## Security Model Invariants — Privilege Separation (setuid nobody)

These apply to `bot/privsep.go`, `bot/calltask.go`, `bot/task_execution.go`, `bot/run_pipelines.go`, and `bot/robot_pipecmd.go`. All are hard escalation triggers.

**Process privilege invariants:**
- There are no normal mid-process privilege transitions. Do not reintroduce `raiseThreadPriv`, `raiseThreadPrivExternal`, `dropThreadPriv`, or thread-pinned credential switching.
- The parent engine runs as the invoking robot user after privsep startup initialization. If it does not, startup/privsep initialization is broken.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lnxjedi/gopherbot](https://github.com/lnxjedi/gopherbot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
