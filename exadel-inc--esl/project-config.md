---
trigger: always_on
description: This file is the **AI-oriented entry point** for working in the ESL monorepo.
---

# AGENTS.md — ESL (Exadel Smart Library) Codebase Guide

This file is the **AI-oriented entry point** for working in the ESL monorepo.
Keep it short, repo-specific, and focused on decision-making. Detailed ESL usage patterns belong in the skills and docs linked below.

## Repository Overview

**ESL** is an Nx-managed monorepo of TypeScript + LESS packages:

| Package | Path | Purpose | Published |
|---|---|---|---|
| `@exadel/esl` | `packages/esl` | Core web-components library | ✓ |
| `@exadel/ui-playground` | `packages/ui-playground` | Real-time code preview widget | ✓ |
| `@exadel/eslint-config-esl` | `packages/eslint-config` | Shared ESLint config | ✓ |
| `@exadel/stylelint-config-esl` | `packages/stylelint-config` | Shared Stylelint config | ✓ |
| `esl-website` | `packages/esl-website` | 11ty + webpack demo site | — internal |
| `esl-website-e2e` | `packages/esl-website-e2e` | Playwright snapshot tests | — internal |
| `esl-website-monitoring` | `packages/esl-website-monitoring` | Website monitoring scripts/actions | — internal |

### Core library shape (`packages/esl/src/<component>/`)
Typical component layout:

```text
esl-<name>/
  core.ts          # only public re-export barrel
  core.less        # styles for custom tags
  core/            # implementation
  test/            # unit tests
```

Rules:
- `core.ts` is the only public component entry.
- Do **not** import from `core/` implementation files as public API.
- Source code lives in `src/`; published build output goes to `modules/`.

## Design Philosophy

ESL is a **base library** — it ships in downstream applications. Each addition is weighed against bundle size, runtime cost, and long-term readability for consumers who will extend and debug it.

When making changes, prefer:

1. **Composition over inheritance**  
   `ESLMixinElement` exists because not every behavior warrants a new custom tag. Before adding a new component, ask whether the behavior should be a mixin on an existing element.

2. **Decorators over boilerplate**  
   Attribute mapping, event subscription, and lifecycle guarding already have first-class primitives such as `@attr`, `@listen`, `@ready`, `@memoize`. Manual `addEventListener`, `removeEventListener`, `getAttribute`, or `setAttribute` inside component code usually means an ESL primitive or utility was missed.

3. **No magic at the call site**  
   Public API should stay readable without internal knowledge. Helpers like `$$find`, `$$fire`, and `$$cls` are intentionally short but explicit. Avoid abstractions that require reading implementation details to understand usage.

4. **Utilities first**  
   Check `packages/esl/src/esl-utils/` before writing helpers. The library already contains traversal, class, async, attribute, focus, and event utilities. Duplication inside component code is a bug, not a style preference. Before implementing any DOM, Async, or Array helper, you must list the contents of esl-utils to check for an existing solution.

5. **Small, targeted diffs**  
   Preserve existing public API, file layout, and naming style unless the task explicitly requires broader refactoring.

## AI Editing Rules

Use these rules when modifying this repository:

### 1. Choose the correct host model
- New custom tag → `ESLBaseElement`
- Attribute-driven behavior on an existing node → `ESLMixinElement`
- In mixins, the real DOM host is `this.$host`, not `this`

For details, see [`packages/esl/skills/esl/references/esl-core.md`](./packages/esl/skills/esl/references/esl-core.md).

### 2. Prefer ESL primitives over raw DOM code
Reach for the existing ESL APIs first:
- decorators: `@attr`, `@boolAttr`, `@jsonAttr`, `@prop`, `@listen`, `@ready`, `@memoize`
- host shortcuts: `$$find`, `$$findAll`, `$$cls`, `$$attr`, `$$fire`, `$$on`, `$$off`
- responsive/event utilities: `ESLTraversingQuery`, `ESLMediaQuery`, `ESLMediaRuleList`, `esl-event-listener` targets

If code looks like generic DOM code with an ESL wrapper, ask which ESL primitive should own it instead.

### 3. Respect public boundaries
- In library source, keep public entrypoints at `core.ts`
- Prefer named exports
- Avoid importing repository internals when a public barrel already exists
- Consumer-facing import patterns are described in [`packages/esl/skills/esl/references/esl-core.md`](./packages/esl/skills/esl/references/esl-core.md)

### 4. Preserve registration and lifecycle contracts
- Set `static is` before `register()`
- Do **not** mutate `is` on built-in ESL components after registration
- Preserve `super.connectedCallback()` / `super.disconnectedCallback()`
- Stable listeners should normally use `@listen`

### 5. Reuse existing event infrastructure
Prefer ESL event abstractions over manual wiring:
- `@listen` for stable class-owned listeners
- `$$on` / `$$off` for dynamic subscriptions
- existing adapters such as resize, intersection, swipe, wheel, or decorated event targets when applicable

### 6. Follow repo conventions via docs, not duplication
- Naming rules: [`docs/CODE_CONVENTIONS.md`](./docs/CODE_CONVENTIONS.md)
- Commit format: [`docs/COMMIT_CONVENTION.md`](./docs/COMMIT_CONVENTION.md)
- Dev workflow and setup: [`docs/DEVELOPMENT.md`](./docs/DEVELOPMENT.md)

### 7. Prefer existing npm scripts and Nx targets

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [exadel-inc/esl](https://github.com/exadel-inc/esl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
