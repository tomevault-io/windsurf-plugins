---
trigger: always_on
description: This file contains repository-specific instructions for coding agents. Keep product usage, CLI examples, and general feature descriptions in `README.md`; keep this file focused on how agents should safely modify the codebase.
---

# AGENTS.md — Open Dynamic Workflow Repository Guide

This file contains repository-specific instructions for coding agents. Keep product usage, CLI examples, and general feature descriptions in `README.md`; keep this file focused on how agents should safely modify the codebase.

---

## 1. Core Boundaries

Open Dynamic Workflow orchestrates external provider CLIs. Do not turn it into a coding-agent implementation.

Preserve these boundaries:

- Workflow runtime owns DSL behavior.
- Scheduler owns agent lifecycle, concurrency, timeout, and cancellation.
- Provider adapters only build provider commands and parse provider output.
- Process runner owns child-process execution.
- Structured output validation stays provider-independent unless a provider truly requires custom parsing.
- Reporters render events and final results; they do not control execution.
- Artifact store owns durable run state.

Do not place provider-specific behavior in workflow validation/runtime, and do not place workflow semantics inside provider adapters.

---

## 2. Code Placement

Prefer the existing layered structure:

```text
src/cli/            command parsing and CLI orchestration
src/config/         config loading, defaults, schema validation
src/workflow/       parser, validator, DSL runtime, sandbox constraints
src/pipeline/       pipeline stage orchestration, item flow, events, summaries
src/orchestration/  scheduler, event bus, concurrency limits, cancellation
src/agents/         provider registry, adapters, process runner abstractions
src/structured/     JSON extraction, normalization, schema validation
src/artifacts/      run store, manifests, logs, durable outputs
src/output/         pretty, JSON, and JSONL reporters
src/errors/         typed errors, serialization, exit-code mapping
src/security/       environment allowlist and redaction helpers
src/types/          public/shared contracts
```

Avoid cross-layer shortcuts. If a change needs a new boundary, design it explicitly before implementing it.

---

## 3. Workflow Runtime Rules

Workflow code runs in a constrained JavaScript / TypeScript-like environment.

Validation/runtime must continue to reject unsupported host access:

- arbitrary imports
- `require()`
- direct filesystem APIs
- direct process APIs
- shell execution
- unsupported globals
- direct access to adapters, event bus, artifact store, or process runner

The constrained runtime reduces accidental misuse, but do not describe it as a complete sandbox for malicious code.

---

## 4. DSL Implementation Rules

### `agent()`

- All provider-backed work must flow through the scheduler.
- Provider-specific flags belong in provider config, not workflow logic.
- Schema validation failure should produce a failed agent result, not an internal crash.
- Persist enough raw, normalized, stderr, schema, and validation-error artifacts for debugging.

### `parallel()`

- Accept task thunks, not already-started promises.
- Preserve global scheduler concurrency.
- Do not add dependency ordering semantics; use `pipeline()` for ordered per-item stages.

### `pipeline()`

`pipeline()` is exposed through the workflow DSL, with implementation support in `src/pipeline/`. It must not introduce a second execution engine.

Rules:

- Keep pipeline orchestration in `src/pipeline/`, with `src/workflow/` limited to DSL/runtime integration.
- Agent calls inside stages must use the existing `ctx.agent()` path.
- Provider adapters must not know whether an agent call came from top-level `agent()`, `parallel()`, or `pipeline()`.
- Pipeline owns item/stage progression, item concurrency, result ordering, and pipeline events/artifacts.
- Scheduler still owns agent lifecycle, provider concurrency, timeout, cancellation, and fail-fast propagation.
- Pipeline must not grant shell, filesystem, or import permissions.

---

## 5. Events, Reports, and Artifacts

- Event sequence numbers must be assigned centrally and remain monotonic per run.
- JSONL output must stay line-delimited machine-readable event JSON.
- JSON output must print only the final machine-readable report.
- Pretty output must not affect execution semantics.
- Unknown event types should be safe for consumers to ignore.
- Do not duplicate full provider logs into pipeline artifacts; reference agent artifacts instead.
- Redact and size-limit previews before writing events, reports, or artifacts.
- Treat artifacts as sensitive because they may contain prompts, code snippets, stdout, stderr, and model output.

---

## 6. Security-Sensitive Areas

Use extra care for changes touching:

- workflow capabilities
- import / require policy
- shell or process execution
- filesystem access
- environment propagation
- provider command construction
- path handling
- artifact/report redaction
- automatic patch application
- workspace write permissions
- worktree or container isolation

Security defaults should stay conservative:

- no arbitrary workflow imports
- no direct shell access
- allowlist-based environment passing
- secret redaction before logs, events, reports, previews, and serialized errors


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [travisliu/open-dynamic-workflow](https://github.com/travisliu/open-dynamic-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
