---
trigger: always_on
description: This file is the operating contract for agentic coding assistants working in this repository. Keep it specific to Tenkit. Prefer concrete repo rules over generic advice.
---

# Agent Guidelines - tenkit

This file is the operating contract for agentic coding assistants working in this repository. Keep it specific to Tenkit. Prefer concrete repo rules over generic advice.

## Project Snapshot

Tenkit is an open source pnpm monorepo for helping people quickly start multi-tenant Expo projects.

- `apps/playground` is the runnable Expo Playground. It proves Setup Types, App Variants, Runtime Tenants, Scaffold, Build Preparation, native assets, and runtime bootstrap behavior in a real app.
- `packages/template-generator` is the open source Template generation package. It exposes explicit local proof and generated-app verification commands for generated Templates.
- `packages/cli` is the Public CLI implementation package. `packages/create-tenkit` is the thin package-manager create entrypoint.
- Web builder, npm publishing, trusted publishing, release automation, and changelogs are future product surfaces unless the current task explicitly scopes them.

## Non-Negotiables

- Use `pnpm` for package scripts and dependency management. Do not use npm, Yarn, Bun, or ad-hoc package manager commands.
- Expo has changed. Before writing Expo code, read the exact versioned docs at `https://docs.expo.dev/versions/v57.0.0/`.
- Preserve Tenkit domain language. Do not collapse App Variant, Runtime Tenant, Setup Type, Example, Starter Data, Scaffold, Template, Playground, Active Setup, and Build Preparation into generic "tenant/template/app" wording.
- Customer-facing marketing copy may translate domain terms into plain audience language when it improves immediate comprehension. It must stay conceptually accurate. Technical explanations, code, and domain documentation must use precise Tenkit terminology.
- Do not introduce additional public CLI surfaces, web builder, npm publishing, trusted publishing, release automation, or changelog automation unless explicitly requested.
- Do not mutate the Playground while proving Template generation, and do not treat the Playground as generated output.
- Do not add broad fallbacks to hide broken behavior. Prefer fixing the underlying behavior. Use fallbacks only at external/runtime boundaries where malformed input is expected and the fallback is explicit.

## Commands

Install dependencies:

```bash
pnpm install
```

Playground commands from the workspace root:

```bash
pnpm start
pnpm test
pnpm typecheck
pnpm lint
pnpm tenkit
```

Template generator commands:

```bash
pnpm -F @tenkit/template-generator test
pnpm -F @tenkit/template-generator typecheck
pnpm proof -- --setup-type white-label --target ../tenkit-white-label-proof
pnpm verify -- --setup-type white-label
pnpm test:proof
```

Expo config smoke checks:

```bash
APP_VARIANT_SLUG=first-tenant pnpm -F playground exec expo config --type public
```

Formatting:

```bash
pnpm format
npx prettier --write <files>
```

When a command is impractical because of local environment state, record the exact reason in the final response.

## Architecture Rules

When a request adopts, replaces, or follows an architecture, pattern, reference implementation, or durable boundary, treat the task as system design work before treating it as file editing.

Before editing, state:

- the boundary being changed,
- the invariants the new design must preserve,
- the scope where those invariants apply,
- and, when a reference is named, which responsibilities from the reference are being adopted, adapted, deferred, or rejected.

During implementation:

- apply chosen conventions consistently across the affected scope,
- do not introduce mixed conventions unless the split is intentional and explained,
- and ask before taking a local shortcut that weakens the adopted architecture.

Before finishing:

- verify behavior and consistency across the whole affected scope, not only touched files,
- record durable boundary decisions in the configured clone-local workflow documentation when it exists,
- and do not add or require tracked public documentation solely to satisfy agent workflow rules. Add public documentation only when the user explicitly requests it or the product itself requires a public document.

Tenkit-specific boundaries:

- The Playground proves behavior in a real Expo app.
- Scaffold changes the Playground's Active Setup.
- Template generation creates a separate generated project.
- Generated apps are the validation target for Template work.
- Build Preparation selects an App Variant and prepares native project state.
- Runtime Tenant selection remains runtime behavior unless intentionally projected into `extra.activeSetup`.

## TypeScript Rules

- Keep strict TypeScript clean. Do not silence errors with `any`, broad casts, or ignored diagnostics.
- Prefer `unknown` over `any` for untrusted or runtime-shaped values, then narrow explicitly.
- Validate unknown runtime/config data at module boundaries.
- Use `as const`, discriminated unions, and `satisfies` for domain literals and setup definitions.
- Prefer precise exported types where they encode a contract; keep local implementation types local.
- Throw `Error` objects with actionable messages. Do not throw strings.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [brilliantinsane/tenkit](https://github.com/brilliantinsane/tenkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
