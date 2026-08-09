---
trigger: always_on
description: The working contract for anyone — human or agent — modifying code in this
---

# Agents

The working contract for anyone — human or agent — modifying code in this
repo. This file is the canonical entry point: read it first, every time.

This is Dunky's state-machine monorepo: UI behavior authored once as
plain TypeScript state machines (`packages/core`), rendered anywhere
through thin per-substrate targets (`react`, `native`, `opentui`), with
a benchmark suite, per-substrate sandboxes, and the docs website
alongside.

## Preflight

Before touching any code, read these in order:

1. `AGENTS.md` (this file) — rules, boundaries, the flow.
2. `README.md` — what this repo is and where things are.
3. `ARCHITECTURE.md` — the deep reference: the layered model, where
   things live, testing, releases.

If a nested `AGENTS.md` exists next to a `package.json`, read it before
editing files in that scope — it overrides anything here for that scope
(see [Agents](#agents-1)).

## Scopes

| Scope     | Path          | What it is                                                                         |
| --------- | ------------- | ---------------------------------------------------------------------------------- |
| Packages  | `packages/**` | The core machine, substrate targets (react, native, opentui), and shared internals |
| Benchmark | `benchmark/`  | Perf suite comparing against competitor libraries                                  |
| Sandbox   | `sandbox/`    | Per-substrate demo apps for manual verification                                    |
| Website   | `website/`    | The docs site                                                                      |

Some changes are cross-scope: a change in `core/` may need follow-up in
the targets, sandboxes, and docs — and vice versa. Check what else your
change touches before calling it done.

## Boundaries

These are invariants, not preferences. Violating them breaks the
layered model:

- **Core never imports a substrate.** No React, no React Native, no
  DOM, no `window`, no `document`. `packages/core/*` is pure
  TypeScript. If you reach for a substrate API in `core/`, stop — the
  code belongs in a target.
- **Targets never reimplement state.** Targets read from the machine
  via its connector. They do not fork the state graph, mirror context,
  or shadow transitions. If a target needs new state, the state goes in
  `core/`.
- **Substrate quirks live in the target.** Focus traps, escape-key
  listeners, back-button handling, RN gesture quirks are
  prop-dependent, so they live in the target as `ComponentEffect`s.
  Props-free, platform-free effects belong in the core machine config's
  `effects` (authored via `setup()`).

## Workflow

`SPEC -> TEST -> IMPLEMENT -> RECONCILE`

Keep an eye on what's been done and what's going on so you don't make a
mess: know the state before you change it, and check your work still
fits after.

### SPEC

Describes the package's intent: the problem it solves, its expected
behavior, and the constraints and edge cases that define its scope. The
spec is a description of intent, not a checklist — the reconcile step
verifies the implementation against it, so state what the package is
meant to do rather than enumerating fields. Capture what carries design
and API meaning: the consumer-facing API, the behavior contract, and any
UX or reference material that anchors it. Its home is the package's
`SPEC.md`.

If the package has no `SPEC.md` yet, ask before creating one — don't
add it unprompted.

A design decision lives in one `SPEC.md` — the package it's about.
Don't copy it into every package it touches. When a decision affects
others, just check their specs don't now contradict it; only edit
another package's `SPEC.md` if it genuinely conflicts (and prefer a
short cross-reference over restating the decision).

### TEST

Write behavior tests first. Tests are the executable spec: they capture
what the code is meant to do before the code exists.

### IMPLEMENT

Write the code that faithfully realizes the SPEC and the behavior the
TEST phase captures, within the boundaries and the architecture. The
implementation must cover the intent, not merely satisfy the assertions.
Match the existing code patterns rather than inventing structure
whenever possible, otherwise open it up for discussion.

### RECONCILE

Check whether the SPEC still describes the code: loop back or ship it.
Before shipping: tests, lint, and type-check pass, and the change is
verified across all scopes. Use the `/code-review` skill before
reviewing code, and suggest improvements you noticed along the way —
don't just do the minimum. If something's off, loop back to SPEC or
TEST; if not, ship it!

## Code

### Naming

Descriptive names everywhere. Short names are fine for local variables
with a tight, obvious scope.

### Comments

If the code says what it does, the comment is noise. Write comments to
explain _why_: a hidden constraint, a subtle invariant, a workaround, a
hard decision. Keep them short — over-explaining is its own kind of
noise.

### Performance

Performance is a constraint, not a feature. Prefer mutation over
allocation on hot paths. Avoid spreading objects, chaining array
methods, or allocating closures inside loops.

### Testing

Unit tests cover behavior and logic only — test what the code does, not

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dunky-dev/state-machine](https://github.com/dunky-dev/state-machine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-08 -->
