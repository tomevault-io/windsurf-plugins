---
trigger: always_on
description: Guide for AI coding agents (and humans) working in this repository.
---

# AGENTS.md

Guide for AI coding agents (and humans) working in this repository.

> This is the canonical instruction file. `CLAUDE.md` exists for Claude Code
> compatibility and defers to this document.

---

## What this package is

`@monetisebg/circuit-breaker` is an open-source npm package: a circuit breaker
for AI agents that stops a run before it burns tokens or spins in a loop.
Behaviour is selected by **mode**:

- `budget-guard` (default) — caps input and output tokens independently
  (`maxInputToken` / `maxOutputToken`, 10k each by default).
- `loop-killer` — hashes each step's state to detect repetition and trips
  when the same state recurs more than `maxRetries` times (default 3). With
  `detectRepeatedState: false` it falls back to a raw iteration cap.

- `worth-it` — a third `withCircuitBreaker` mode that works in **currency**, not
  token counts. It costs each step (`C_s = (I·P_in + O·P_out) / 1e6`, with prices
  quoted per 1M tokens in the currency's smallest unit, e.g. cents), smooths an
  EMA, projects the run's total spend (`C_proj = C_cum + EMA·R_s`) against a
  `budgetLimit`, and fires graduated events (`predictive_warning` at 0.70,
  `optimize_context` at 0.85, `tripped` + throw at 0.95). Progress (`R_s`) is
  developer-driven via a milestone API (the wrapper's `onWorthItStep` hook) — no
  LLM self-reporting. Trips are *checkpoint & pause* (the engine isn't latched).

The `worth-it` logic lives in a dedicated `WorthItEngine` (it has a different
shape from the token-counting modes: cost, pricing, milestones). The core
`CircuitBreaker` class is the single entry point for **all three** modes — in
`worth-it` mode it delegates to an internal `WorthItEngine`, exposing
`recordStep` / `completeMilestone` / `setCompletedMilestones` / `worthItMetrics`
(the token/loop methods like `addTokens` throw a `TypeError` in `worth-it` mode,
and vice versa). Each adapter branches on the mode: for `worth-it` it builds the
engine via `createWorthItRunner` and feeds it each step's `{ input, output,
model }`. The engine is also exported standalone at the `/worth-it` subpath for
frameworks without an adapter.

Decision logic lives in a framework-agnostic core; each agent framework gets
its own thin adapter exposed as a subpath import. The breaker emits
`CircuitBreakerEvent`s (`retry`, `stop` for the wrapper modes;
`predictive_warning`, `optimize_context`, `tripped` for worth-it) for
visibility into what it's doing.

Current adapters:

- `@monetisebg/circuit-breaker/langchain` — wraps any LangChain `Runnable`
  (e.g. `AgentExecutor`) via a `BaseCallbackHandler`.
- `@monetisebg/circuit-breaker/openai-agents` — wraps an `Agent` from the
  OpenAI Agents SDK using `Runner` events and `AbortSignal`.
- `@monetisebg/circuit-breaker/claude-agent-sdk` — wraps the Claude Agent
  SDK's `query` function; drives the breaker off the streamed `SDKMessage`s
  and aborts via the SDK's `abortController` option.
- `@monetisebg/circuit-breaker/langgraph-sdk` — wraps a LangGraph Platform
  `client.runs`; drives the breaker off the `events` stream mode (forced into
  `streamMode`), aborts the local stream and cancels the run server-side via
  `runs.cancel`. For an in-process `@langchain/langgraph` graph use the
  LangChain adapter instead — a compiled graph is a `Runnable`.
- `@monetisebg/circuit-breaker/vercel-ai-sdk` — wraps the Vercel AI SDK's
  `generateText`; drives the breaker off the injected `onStepFinish` (one step
  per LLM call) and aborts the tool-loop via the `abortSignal` option. Non-
  streaming only — `streamText` is not yet supported.
- `@monetisebg/circuit-breaker/worth-it` — the `WorthItEngine` + `ProgressTracker`
  + `createWorthItRunner` (cost-predictive mode). Used internally by every
  adapter for `mode: "worth-it"`; also usable directly for frameworks without an
  adapter (`recordStep` / `completeMilestone`).

Every adapter accepts `mode: "worth-it"` (with `budgetLimit`, `currency`,
`milestones`, `pricing`/`defaultPricing`, and an optional `onWorthItStep` hook)
in addition to the two token/loop modes. Pricing is per-1M-tokens in the
currency's smallest unit (`ModelPricing.inputPerMToken` / `outputPerMToken`).

The package root (`@monetisebg/circuit-breaker`) exports the core:
`CircuitBreaker`, `CircuitBreakerError`, `WorthItEngine`, `ProgressTracker`,
and the option/context types.

---

## Repository layout

```
src/
├── core/                  # Framework-agnostic decision logic.
│   ├── breaker.ts         #   CircuitBreaker class — the single source of truth.
│   ├── worth-it.ts        #   WorthItEngine + ProgressTracker (cost-predictive).
│   ├── errors.ts          #   CircuitBreakerError.
│   ├── logger.ts          #   defaultLogger (console.warn-based).
│   ├── types.ts           #   Public types (Options, Metrics, TripContext, …).
│   └── index.ts
├── langchain/             # LangChain.js adapter.
│   ├── callback.ts        #   CircuitBreakerCallback : BaseCallbackHandler.
│   ├── tokens.ts          #   Provider-shape-aware token extraction.
│   ├── wrapper.ts         #   withCircuitBreaker(runnable, options).
│   └── index.ts
├── openai-agents/         # @openai/agents adapter.
│   ├── wrapper.ts         #   withCircuitBreaker(agent, options) — uses

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MonetiseBG/circuit-breaker](https://github.com/MonetiseBG/circuit-breaker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
