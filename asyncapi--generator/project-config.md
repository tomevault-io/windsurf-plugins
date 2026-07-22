---
trigger: always_on
description: This document is the source of truth for code review standards in the `asyncapi/generator` monorepo. It is consumed by CodeRabbit (`knowledge_base.code_guidelines` in `.coderabbit.yaml`) and is intended to be read by both human reviewers and AI agents.
---

# AGENT.md — AsyncAPI Generator Code Guidelines

This document is the source of truth for code review standards in the `asyncapi/generator` monorepo. It is consumed by CodeRabbit (`knowledge_base.code_guidelines` in `.coderabbit.yaml`) and is intended to be read by both human reviewers and AI agents.

The monorepo contains tightly coupled packages that together form the AsyncAPI code/documentation generation pipeline. Each package has a specific role and its own conventions; this file captures both the shared rules and the per-package deviations.

---

## Referenced documents

The guidelines below cross-reference the following authoritative docs. If a path or URL changes, update it here **and** at each inline mention.

- [Conventional Commits](CONTRIBUTING.md#conventional-commits) — commit and PR title format rules
- [Release process](Development.md#release-process) — changesets, release-triggering prefixes, full release flow
- [Packages README](packages/README.md) — template architectural principles and assumptions
- [Hooks guide](apps/generator/docs/hooks.md) — lifecycle hook order and signatures (`generate:before`, `generate:after`, `setFileTemplateName`)
- [Keeper README](apps/keeper/README.md) — `@asyncapi/keeper` public API surface
- [React SDK README](apps/react-sdk/README.md) — rendering architecture, restrictions, and component examples
- [Baked-in Templates guide](apps/generator/docs/baked-in-templates.md) — directory layout, required files, package-name convention
- [WebSocket test README](packages/templates/clients/websocket/test/README.md) — `TEST_CLIENT` scoping, snapshot layout, Microcks setup

---

## 1. Repository topology

```text
apps/
  generator/          # @asyncapi/generator — main CLI + Generator class (CJS, Jest)
  hooks/              # @asyncapi/generator-hooks — built-in lifecycle hooks (CJS, Jest)
  keeper/             # @asyncapi/keeper — AJV message validator (ESM → Babel → CJS)
  nunjucks-filters/   # legacy/stub — treat as frozen; do not add features here
  react-sdk/          # @asyncapi/generator-react-sdk — TS React renderer (TS → tsc; bundles Rollup as a runtime dep for its template transpiler)
packages/
  components/         # @asyncapi/generator-components — shared React template components (JSX)
  helpers/            # @asyncapi/generator-helpers — pure JS helpers over Parser API (CJS, no build)
  templates/          # Baked-in templates shipped inside the generator (grouped by type: clients, sdks, docs, configs). Currently: clients/kafka/java/quarkus, clients/websocket/{javascript,python,dart,java/quarkus}
```

Orchestration is Turborepo (`turbo.json`). Every package-level script (`test`, `lint`, `build`, `docs`) must be runnable through the root `turbo run <script> --filter=<pkg>` — do not introduce scripts that only work from inside a package directory when they need the full workspace graph.

---

## 2. Monorepo-wide rules

### 2.1 Runtime and tooling
- **Node**: `>=24.11`. **npm**: `>=11.5.1`. Do not regress `engines` in any `package.json`.
- **Module style** differs per package (see 4). Do not silently switch a package from CJS to ESM or vice versa; that is a structural change that must be called out in the PR description.
- **No Prettier.** Formatting is enforced entirely through ESLint. Do not add a `.prettierrc` or `prettier` devDependency.

### 2.2 Linting
Every package inherits the root `.eslintrc` — that file is the source of truth for lint rules. Package `lint` scripts must invoke the root config and ignore file via relative `--config` / `--ignore-path` flags — the exact number of `../` segments depends on the package's depth in the tree (e.g. `apps/*` uses `../../.eslintrc`; `packages/templates/clients/<protocol>/test/integration-test/` uses `../../../../../../.eslintrc`). Do not add a package-local `.eslintrc` to avoid the relative path.

### 2.3 Commits and PR titles
See the [Conventional Commits section in `CONTRIBUTING.md`](CONTRIBUTING.md#conventional-commits).

### 2.4 Documentation and comments

**Packages that require JSDoc on public functions:**

| jsdoc2md scans (entry passed to `--files`) | Published docs file | Build? |
|---|---|---|
| `apps/generator/lib/generator.js` only | `apps/generator/docs/api.md` | `jsdoc2md`, committed |
| `packages/components/src` | `apps/generator/docs/api_components.md` | `jsdoc2md`, committed |
| `apps/react-sdk/src` | `apps/react-sdk/API.md` | `jsdoc2md`, committed |

Required tags: `@param`, `@returns`, and `@throws` / `@async` where applicable.

"Public" means **exported from the package's main entry, or reachable via that package's jsdoc2md config** — not every file-internal helper. If a symbol shows up in one of the generated MD files, it is public by definition. Being under the source tree or `module.exports`-ed is *not* the trigger: for `apps/generator` the docs build scans only `lib/generator.js`, so internal modules it never reaches (`lib/logMessages.js`, `lib/utils.js`, `lib/parser.js`) need no JSDoc — demanding it there is a false positive.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [asyncapi/generator](https://github.com/asyncapi/generator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
