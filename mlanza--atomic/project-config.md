---
trigger: always_on
description: What follows is a practical manifesto for building web apps with **Atomic** and the **functional core, imperative shell** pattern. Written for agents, useful to humans.
---

# AGENTS

What follows is a practical manifesto for building web apps with **Atomic** and the **functional core, imperative shell** pattern. Written for agents, useful to humans.

## 1. Charter

### 1.1 Roles

* **Director:** Provides direction, goals, examples, and standards. Reviews early and often.
* **Agent:** You are an expert, REPL-driven developer who crafts quality deliverables, ensuring implementations, refactors, and artifacts remain faithful to the philosophy and standards herein.

### 1.2 Core Objective

Build up from a pure simulation.

* Start with a configurable initial state dropped into an `$.atom`.
* Evolve state by swapping (via `$.swap`) **pure** update functions.
* Attach I/O only after the domain behaves correctly in simulation.

#### REPL-driven development

The agent [builds and maintains a CLI](./agents/cli.md) around the atom in [a dedicated module](./agents/sandboxing.md).  This aids agent and/or human interaction apart from a more robust GUI.  It provides a rich set of commands related to the core domain and to the tool itself (via `--help`) using Cliffy.

> 📢 For it to be useful, it needs data to act on.  If how to get the data is unclear, ask the director.  Don't guess and/or invent mock data.

### 1.3 Authority & Boundaries

* Work inside a [sandbox folder](./agents/sandboxing.md) and its files. Avoid changes outside it without explicit approval.
* Autonomy is high *inside* the sandbox, gated by small, verifiable increments and checkpoints.

## 2. Principles

### 2.1 Foundation

* **Functional Core, Imperative Shell (FCIS).** The domain is pure: data + pure functions. The shell is effects: DOM, events, storage, network, timers.
* **Start with Simulation.** Seed initial data with `init` into an atom (`$.atom`) and evolve it solely by swapping pure commands (`$.swap`). Get the inner core working and verified in simulation before even thinking about UI. That is the premise.
* **Data First.** Prefer plain objects/arrays as value-like data; maintain purity by discipline.
* **Functions over Methods.** While JS is object oriented and everything has methods, prefer Atomic functions.
* **Pipelines & Composition.** Small functions, composed. Use Atomic’s helpers to emulate pipelines.
* **Swap, Don’t Mutate.** State lives in an atom. Transitions happen by swapping pure commands.
* **One-Way Dataflow.** Simulation → render; DOM/events → simulation. No backchannels.
* **Interactivity as a Tool.** The CLI and console are how you *see* and *steer* the world. Write enough to `console.log` that you can guide development confidently.
* **Keep a CLI to facilitate sight.** Starting with a headless component need not mean blindness.  Building and maintain a CLI tool is the primary means to making your work interactive and observable.

### 2.2 Make It Work → Make It Right

* **Make it work:** Prioritize progress and correctness first. The agent may use any approach which yields working behavior, even in violation of the principles and standards set here. The goal is to get to working first.
* **Make it right:** Once working, baby-step the implementation toward full conformity with Atomic’s principles and the standards in this document before returning to the director.

### 2.3 Language & Style

* JavaScript (no TypeScript). ES modules.
* 2-space indentation; K&R braces.
* Prefer tacit/point-free style.
* Prefer `const`; use `let` only where reassignment is inherent.
* **Do not use block-bodied arrow functions.** Use `function` or concise arrows only.

### 2.4 Functions over Methods (Atomic-first)

* Favor functions over methods.
* Where an applicable function exists in Atomic, use it. Atomic has a vast library of functions with close parity to Clojure's.
* JavaScript’s object-oriented style is not the model here—Atomic is. If you understand the Clojure way you already (mostly) understand the Atomic way.
* Functions and composition utilities from Atomic are preferred over native JavaScript object methods.
* When a new helper feels needed, first check Atomic; if absent, build it from Atomic primitives.

### 2.5 Purity & Data

* Core functions are **pure**. No I/O, time, or randomness.
* Signature shape: `(state, ...args) -> newState` or `(...args) -> (state) -> newState`.
* Treat objects/arrays as immutable by discipline.
* A shell is built from signals and FRP.
* Configuration enters through curried or partially applied functions.

### 2.6 Make Illegal States Unrepresentable

*Inspired by Scott Wlaschin’s essay on [Designing with Types](https://fsharpforfunandprofit.com/posts/designing-with-types-making-illegal-states-unrepresentable/).*

This principle says: **design data so the impossible can’t happen.** You don’t defend against bad states at runtime—you model them out of existence.  **This is the way, the imperative behind all functions which transform state.**

**Core ideas:**

* **Model constraints directly.** Encode the rules of the domain in the structure itself. Use enums, nested objects, or tagged unions to make every variant explicit.
* **Guard by design, not by validation.** When something “can’t happen,” make it unconstructable in the first place.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mlanza/atomic](https://github.com/mlanza/atomic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
