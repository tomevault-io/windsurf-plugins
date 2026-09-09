---
trigger: always_on
description: Guidance for working in this repository.
---

# CLAUDE.md

Guidance for working in this repository.

## Project

`vscode-forge-tools` — a VS Code extension (published as **Autodesk Platform Services**) that surfaces
APS services (OSS, Model Derivative, Data Management/Hubs, Webhooks, Design Automation, Secure Service
Accounts) as tree views, commands, and React webview panels. TypeScript, bundled with esbuild.

## Build & run

```bash
yarn install       # install dependencies
yarn build         # esbuild -> out/ (extension + webviews)
```

- Debug in VS Code with the **Launch Extension** task (opens an Extension Development Host window).
- `vscode:prepublish` runs a production build (`NODE_ENV=production`): minified, no sourcemaps.
- Requires at least one APS environment configured (`autodesk.forge.environments`); on first run the
  extension guides you through creating one.

## Architecture

The source under `src/` is organized into five layers with a strict, one-directional dependency rule.

### Layers

```
                 ┌───────────────┐     ┌───────────────┐
                 │  src/commands │     │ src/providers │      VS Code glue (thin)
                 └──────┬────────┘     └──────┬────────┘
                        │                     │
        ┌───────────────┼─────────────────────┼───────────────┐
        ▼               ▼                     ▼                ▼
  ┌───────────┐   ┌──────────────┐      ┌──────────────┐  ┌───────────┐
  │src/webviews│  │ src/services │─────▶│  src/models  │◀─│ (all      │
  └─────┬─────┘   └──────┬───────┘      └──────────────┘  │  layers)  │
        │                │                                 └───────────┘
        └────────────────┴──────────────▶ src/models
```

- **`src/models`** — Types and interfaces only. **No `vscode`. No logic.** Categorized by service
  (`oss.ts`, `model-derivative.ts`, `hubs.ts`, `design-automation.ts`, `design-automation-api.ts`,
  `secure-service-accounts.ts`, `webhooks.ts`, `authentication.ts`), plus `environment.ts`
  (`IEnvironment`, `DesignAutomationRegion`). This is the single place every other layer gets its types.
  When a non-service layer needs an `@aps_sdk/*` type, that type is **re-exported** from the matching
  `src/models/*` file (e.g. `export type { Bucket, ObjectFullDetails } from '@aps_sdk/oss';`) so that no
  layer other than `src/services` ever imports an npm SDK package directly.

- **`src/services`** — Domain logic. **No `vscode`.** This is the *only* layer allowed to import
  `@aps_sdk/*` and to use `fetch`/`fs`. Categorized by service, one `Service` class each:
  `OssService`, `ModelDerivativeService`, `HubsService`, `DesignAutomationService`,
  `SecureServiceAccountsService`, `WebhooksService`, `AuthenticationService`. Each wraps its underlying
  `@aps_sdk/*` client (or, for Design Automation, the hand-written `fetch`-based `DesignAutomationClient`)
  and exposes plain domain methods; SDK enums/values (`Access`, `PolicyKey`, `Region`, `Scopes`, `Utils`)
  stay *inside* the service. Shared infra also lives here: `aps-sdk-manager.ts`,
  `client-credentials-authentication-provider.ts`, `static-token-authentication-provider.ts`, and the
  `createServices()` factory in `index.ts`.

- **`src/webviews`** — React apps (`*.tsx`, each exporting `render(container, props)`), bundled
  separately for the browser. They import **only** types from `src/models` (and their own UI toolkit) —
  never `vscode`, never `@aps_sdk/*`, never a service. Communication with the extension host is via
  `postMessage`.

- **`src/commands`** — Thin `vscode` command wrappers, one registry class per service exposing
  `registerCommands(): vscode.Disposable[]`. They gather input, drive quick-picks/dialogs/progress, open
  webview panels, and call `context.<name>Service.*` for all domain work.

- **`src/providers`** — Thin `vscode.TreeDataProvider`s, one per view. They map results from
  `context.<name>Service.*` into `vscode.TreeItem`s.

#### Dependency rule

Dependencies point one way only:

- `commands`, `providers` → `services` + `models`
- `services` → `models`
- `webviews` → `models`

`src/services/**` and `src/models/**` must never `import 'vscode'`. Commands, providers, and webviews
must never `import` from `@aps_sdk/*` — types come from `src/models`, behavior from `src/services`.
These invariants are checked with:

```bash
grep -rn "from 'vscode'\|import \* as vscode" src/services src/models   # must be empty
grep -rn "@aps_sdk" src/commands src/providers src/webviews             # must be empty
```

### The context object

`activate()` (`src/extension.ts`) builds a single `IContext` (`src/common.ts`). `IContext extends
IServices` — so `context` carries every service instance (`context.ossService`,
`context.modelDerivativeService`, …) plus VS Code state (`extensionContext`, `environment`,
`previewSettings`, `threeLeggedToken`, `log`). Every provider and command receives this `context`.

Service construction happens in exactly one place — `createServices(env, threeLeggedToken?)` in
`src/services/index.ts`. Activation, environment switching (`EnvironmentCommands.switchEnvironment()`),
and login/logout (`AuthenticationCommands`) all call it again and `Object.assign` the fresh services
onto `context`, rather than mutating existing instances.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [petrbroz/vscode-forge-tools](https://github.com/petrbroz/vscode-forge-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
