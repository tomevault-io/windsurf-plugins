---
trigger: always_on
description: This file provides guidance specific to **mnemonica/core** for AI agents
---

# AGENTS.md

This file provides guidance specific to **mnemonica/core** for AI agents
modifying the library itself. If you are *using* mnemonica in your own
project, start with [`README.md`](./README.md).

---

## Rule #1 — Non-negotiable: PAUSE AND ASK

**This is the highest-priority rule. It overrides everything else in this file.**

You MUST pause and ask the user before proceeding if any of these is true:

1. **An editing error occurred** — `Edit` failed, `Write` produced unexpected
   results, or any tool returned an error.
2. **You are uncertain** — about what change to make, how a function works,
   or what the user intended.
3. **You are filling gaps with assumption** — "probably", "likely",
   "I think", "it should work" are signals to stop.

When in doubt: STOP, ask a clear specific question, WAIT for the answer.
Do not invent workarounds (no `sed`, no `python -c`, no console hacks).

The reason this rule exists: wrong assumptions waste both your time and the
user's. The library encodes non-obvious design intent (data-flow vs control-flow,
`define()` semantics, the proxy architecture, the return-via-variable rule).
Confident guesses produce code that compiles but corrupts the design.

---

> **Note:** Framework-agnostic rules are also available in `.ai/`:
> [`AGENTS.md`](./.ai/AGENTS.md), [`CODE.md`](./.ai/CODE.md),
> [`ARCHITECT.md`](./.ai/ARCHITECT.md), [`DEBUG.md`](./.ai/DEBUG.md),
> [`async_init.md`](./.ai/async_init.md).
> These rules apply to all agent frameworks.

## What and why

For the library's thesis, the four data mistakes it eliminates, the Trie observation, the pipeline pattern, and the HoTT framing, read [`README.md`](./README.md). This file assumes you have.

The short version for editors: mnemonica is an instance inheritance system; `define()` declares types; subtypes are constructed from parent *instances* via `new instance.SubType(args)`; the prototype chain back to root IS the identity; construction context is stored in a WeakMap and exposed via `getProps()`.

## Agent Reading Guide

Load the docs that match your change type. The wrong context produces broken code; the right context is a short read.

| Change type | Read before starting |
|---|---|
| Any `src/` change | This file + [`.ai/ONBOARDING.md`](./.ai/ONBOARDING.md) |
| Involves `define()` / type graph | + [`.ai/rules-skill/define-patterns.md`](./.ai/rules-skill/define-patterns.md) |
| Involves hooks | + [`.ai/rules-skill/hooks.md`](./.ai/rules-skill/hooks.md) |
| Involves async constructors | + [`.ai/rules-skill/async-constructors.md`](./.ai/rules-skill/async-constructors.md) + [`.ai/async_init.md`](./.ai/async_init.md) |
| Involves TypeScript types | + [`.ai/rules-skill/type-system.md`](./.ai/rules-skill/type-system.md) |
| Involves proxy internals | + [`.ai/rules-skill/proxy-architecture.md`](./.ai/rules-skill/proxy-architecture.md) |
| Uses tactica / `lookup` | + [`.ai/TACTICA-RULES.md`](./.ai/TACTICA-RULES.md) |
| Docs-only change | README section you're touching only |

**This file + `.ai/ONBOARDING.md` are the always-required baseline for any `src/` edit.**

### Framework-specific rules

Mode-specific files in `.ai/rules/`:
- [`.ai/rules/CODING.md`](./.ai/rules/CODING.md) — universal coding rules
- [`.ai/rules/REMINDERS.md`](./.ai/rules/REMINDERS.md) — type vs interface, spacing reminders
- [`.ai/rules/CONTEXT-CONDENSING.md`](./.ai/rules/CONTEXT-CONDENSING.md) — context recovery protocol

## Build/Test Commands

See [`.ai/rules-skill/testing.md`](./.ai/rules-skill/testing.md) for the full command reference, dual-framework details, and coverage requirements. Summary:

```bash
npm run build          # full build with linting
npm run test:cov       # Mocha + coverage (runs build:all internally)
npm run test:jest:cov  # Jest on TypeScript source
npm run watch          # watch mode
```

**Must run `npm run test:cov` before completing any task.**

**Documentation changes:** When modifying any `.md` file, `npm run lint:md` is mandatory. It checks for broken links and anchors. Run it and fix any reported issues before finishing.

## Code Style (Project-Specific)

See [`.ai/rules-skill/code-style.md`](./.ai/rules-skill/code-style.md) for the full style reference. Key rules: tabs only, space before function parens, colons aligned in object literals, `strict: true`, **no `any`** (`no-explicit-any: error`).

## Architecture Patterns

### The `define()` Function
The core API is `define(TypeName, constructHandler, config?)` in `src/index.ts`. It returns a constructor with additional methods:
- `.define()` - define subtypes
- `.lookup()` - find types by path
- `.registerHook()` - register lifecycle hooks

### The `lookup()` Function

For user-facing semantics, see [`README.md`](./README.md) and [`.ai/TACTICA-RULES.md`](./.ai/TACTICA-RULES.md). The contributor-relevant detail is the implementation pattern: `TypeRegistry` starts empty, and `lookup()` uses overloads so augmented keys return the typed constructor while unaugmented keys fall back to `TypeClass | undefined`.

```typescript
// In mnemonica core (src/index.ts)
export interface TypeRegistry {
	// Intentionally empty. Augment via declaration merging.
}

export function lookup<const K extends keyof TypeRegistry>(
	this: unknown,
	TypeNestedPath: K

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wentout/mnemonica](https://github.com/wentout/mnemonica) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
