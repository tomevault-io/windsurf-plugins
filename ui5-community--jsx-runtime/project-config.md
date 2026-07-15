---
trigger: always_on
description: Orientation for coding agents (Claude Code, Cursor, Copilot, aider, and friends) landing on this repo. Human contributors, see [CONTRIBUTING.md](CONTRIBUTING.md).
---

# AGENTS.md

Orientation for coding agents (Claude Code, Cursor, Copilot, aider, and friends) landing on this repo. Human contributors, see [CONTRIBUTING.md](CONTRIBUTING.md).

## What this repo is

A pnpm monorepo with three packages:

- **`@ui5-community/jsx-runtime`** — [packages/jsx-runtime/](packages/jsx-runtime/) — the publishable library. Core JSX runtime + one sample plugin (`<Switch>`).
- **`@ui5-community/jsx-runtime-showcase`** — [packages/jsx-runtime-showcase/](packages/jsx-runtime-showcase/) — a runnable UI5 app that doubles as living documentation. Not published.
- **`@ui5-community/jsx-runtime-helloworld`** — [packages/jsx-runtime-helloworld/](packages/jsx-runtime-helloworld/) — a minimal, plain-scaffolded UI5 TypeScript app that consumes the runtime the way an external project would (`workspace:^` dependency, `jsxImportSource` + `ui5.yaml` `transformJSX` wiring only). The reference for "how do I adopt this from scratch?" Not published.

The runtime is small on purpose: ~2600 LoC of source, 12 runtime exports, ~12 types, one plugin SPI with five extension points. Read the whole barrel in one sitting: [packages/jsx-runtime/src/jsx-runtime.ts](packages/jsx-runtime/src/jsx-runtime.ts).

## If you need to know X, read Y

| You want to answer… | Read |
| --- | --- |
| What does the runtime do at all? | [docs/jsx-runtime.md](docs/jsx-runtime.md) |
| Is behaviour X a requirement or an accident? | [docs/requirements.md](docs/requirements.md) — search for `FR-…` / `NFR-…` IDs, they're cited from source JSDoc |
| How do I write recipe X? (events, bindings, tables, plugin, embedding) | [docs/cookbook.md](docs/cookbook.md) — 15 canonical recipes |
| Which known pitfall is this? | [docs/gotchas.md](docs/gotchas.md) — 12 entries, cross-linked from source |
| How do I migrate an XMLView? | [docs/MIGRATION.md](docs/MIGRATION.md) — translation table + prerequisites; [docs/xmlview-to-tsx.md](docs/xmlview-to-tsx.md) — full step-by-step conversion guide |
| Why is there no sample for X (routing, OData, custom control)? | [docs/deferred-samples.md](docs/deferred-samples.md) |
| What changed in the current release? | [packages/jsx-runtime/CHANGELOG.md](packages/jsx-runtime/CHANGELOG.md) (changesets-generated) |
| How do I import / configure this from a UI5 app? | [packages/jsx-runtime/README.md](packages/jsx-runtime/README.md) + [webapp/docs/setup.md](packages/jsx-runtime-showcase/webapp/docs/setup.md), [webapp/docs/tsconfig.md](packages/jsx-runtime-showcase/webapp/docs/tsconfig.md), [webapp/docs/ui5-yaml.md](packages/jsx-runtime-showcase/webapp/docs/ui5-yaml.md) |
| How does `<Foo prop=…/>` become `new Foo({prop: …})`? | [src/runtime/runtime.ts](packages/jsx-runtime/src/runtime/runtime.ts) — start at `export function jsx`, which now carries an `@example` block |
| How do I write a plugin? | [src/runtime/plugin.ts](packages/jsx-runtime/src/runtime/plugin.ts) (SPI) + [src/plugins/switch/index.tsx](packages/jsx-runtime/src/plugins/switch/index.tsx) (worked example) + [webapp/docs/plugin-spi.md](packages/jsx-runtime-showcase/webapp/docs/plugin-spi.md) + [docs/cookbook.md](docs/cookbook.md) recipe 15 |
| What are `Scope` / `withScope` for? | [src/runtime/scope.ts](packages/jsx-runtime/src/runtime/scope.ts) — file header explains the design; see caveat below |
| What does a real sample look like? | Enumerate via [packages/jsx-runtime-showcase/samples.json](packages/jsx-runtime-showcase/samples.json), then read the paired `.tsx` under [webapp/view/showcases/](packages/jsx-runtime-showcase/webapp/view/showcases/) + `.md` under [webapp/docs/samples/](packages/jsx-runtime-showcase/webapp/docs/samples/) |
| How is behaviour X tested? | [test/qunit/runtime/](packages/jsx-runtime/test/qunit/runtime/) — one file per concern |
| Which controls work as JSX tags? | Any UI5 class you `import` — see the "no registry" note below |

## Non-obvious things to know before you edit

### 1. There is no control registry

`jsx(type, props)` calls `new type(settings)` on whatever class you pass. Consumers `import Button from "sap/m/Button"; <Button …/>`. The tag namespace is exactly the set of UI5 modules the app imports. Per-control prop typing comes for free from `@openui5/types` (`$XSettings` interfaces). Do **not** add a control allow-list, mapping table, or tag-name-to-class bridge; the design explicitly rejects it.

### 2. HTML intrinsic tags are typed but not runtime-supported

`JSX.IntrinsicElements` permissively types `<div>`, `<svg>`, etc. The **default** renderer throws on lowercase tags because a `<div>` cannot become a UI5 control. An HTML-aware renderer plugin (planned) will install an `htmlIntrinsic` in the scope. Type-clean code can still fail at runtime — surface this in error messages if a user reports "TypeScript accepted it but it throws."

### 3. `withScope` is synchronous only


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ui5-community/jsx-runtime](https://github.com/ui5-community/jsx-runtime) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-15 -->
