---
trigger: always_on
description: Generates:    AGENTS.md
---

<!-- ───────────────────────────────
  Template:     AGENTS.md
  Template-ID:  agents
  Generates:    AGENTS.md
  Description:  Agent entry contract — first file every AI agent reads (commands, rules, boundaries, routing).
  Library ver:  0.2.2
  Last updated: 2026-07-22
─────────────────────────────── -->

<!-- sdd-generated-metadata
doc_kind: agent-entry
generated_from: agents
generator_plugin: sdd-bootstrap@0.4.3
generated_by: cursor-agent
approved_by: pending PR approval
updated_at: 2026-09-08
validation_status: pass
-->


# AGENTS.md — react-widgets

> Read first. Next: [`ai-docs/SPEC_INDEX.md`](ai-docs/SPEC_INDEX.md) for routing and [`ai-docs/ARCHITECTURE.md`](ai-docs/ARCHITECTURE.md) for system shape. Load only the module specs needed for the task.

## Repo Overview

**react-widgets** is a browser-focused React package repository that provides Webex widgets, reusable UI components, Redux modules, host-integration helpers, build/release tooling, and browser journey tests.

**What it is:**

- A single root build that produces many `@webex/*` npm packages and selected CDN widget bundles.
- A client library that consumes Webex JavaScript SDK services and exposes React, browser-global, data-attribute, and event surfaces.

**What it is NOT:**

- It is not the Webex backend or the Webex JavaScript SDK implementation.
- It does not own a server datastore or API schema.
- It is not a native mobile or desktop application.

## Tech Stack

- JavaScript and TypeScript; React 16, Redux 3, Immutable.js, React-Redux, and recompose.
- Babel 7, Rollup 2, Webpack 4, npm, Jest 24, WebdriverIO 7, Sauce Labs, and axe-core.
- CI currently runs Node 22.22; `.nvmrc` selects the Node 22 LTS line (`lts/jod`).

## Architecture

```text
host application / HTML data attributes
        ↓
webex-widget-base (mount, auth/SDK, Redux provider, events, teardown)
        ↓
capability widgets → containers/HOCs → Redux modules → Webex JS SDK/services
        ↓                    ↘ shared React components
host callbacks + DOM/ampersand events
```

Full component responsibilities and interaction diagrams: [`ai-docs/ARCHITECTURE.md`](ai-docs/ARCHITECTURE.md).

## Module / Package Structure

```text
packages/node_modules/@webex/
├── widget-*                 # capability widgets and demos
├── react-component-*        # reusable UI components
├── react-container-*        # state/SDK-connected components
├── react-hoc-*              # reusable higher-order behavior
├── redux-module-*           # actions, reducers, constants, helpers, and/or thunks; view selectors live in widget/container packages
├── react-redux-*            # SDK/store integration, fixtures, metrics
└── webex-widget-base/       # common host/runtime composition
scripts/                     # build, start, publish, deploy, and tooling commands
test/journeys/               # browser integration and accessibility suites
```

Module routing: [`ai-docs/SPEC_INDEX.md`](ai-docs/SPEC_INDEX.md).

## Critical Rules

1. **Code and tests describe current behavior.** Never invent a package export, prop, event, command, destination type, or SDK capability.
2. **Plan and confirm behavior changes.** This bootstrap is docs-only; future public API, event, auth, build, release, or test-policy changes require an approved spec delta.
3. **Preserve package entrypoints.** `src/index.js` / `src/index.ts`, package `main`/`module`, widget browser names, data-toggle names, and event strings are compatibility surfaces.
4. **Preserve enhancer ordering.** Data API and browser globals wrap Redux setup, widget removal, SDK injection, current-user loading, display name, and version metadata in `webex-widget-base`.
5. **Keep Redux state immutable.** Follow existing action/reducer/thunk patterns and never mutate stored records in place.
6. **Treat tokens and SDK instances as host-supplied credentials.** Never commit or log access tokens, guest tokens, client secrets, Sauce keys, npm tokens, Netlify tokens, or AWS credentials.
7. **Keep accessibility behavior testable.** Maintain roles, labels, keyboard navigation, focus transitions, and axe journey coverage when changing UI.
8. **Update SDD docs with behavior.** Change the owning module spec, contracts/state indexes, and manifest in the same change.

## Essential Commands

| Role | Command |
|---|---|
| Install | `npm install --legacy-peer-deps` |
| Build | `npm run build:all` |
| Unit test | `npm run jest` |
| Lint/format | `npm run static-analysis` |

→ Full command surface, toolchain, and registries: **[GETTING_STARTED.md](./ai-docs/GETTING_STARTED.md)** · test tiers and coverage gate: **[TEST_INDEX.md](./ai-docs/TEST_INDEX.md)**

## Common Gotchas

1. Package source is intentionally tracked under `packages/node_modules/@webex/`; do not treat it as installed third-party output.
2. Jest remaps `@webex/*` imports to package `src/` entrypoints, so tests exercise source rather than built `cjs/` or `es/` output.
3. Space and Recents use encrypted Mercury flows by default; Recents `basicMode` uses Webex REST and changes encryption behavior.
4. Widget teardown must call the browser widget's `remove()` path so React unmounts and `window.webex.widgetStore` is cleared.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [webex/react-widgets](https://github.com/webex/react-widgets) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
