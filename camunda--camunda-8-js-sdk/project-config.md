---
trigger: always_on
description: This file is the **canonical** instruction document for AI agents (Copilot, Claude, etc.) working on the Camunda 8 JavaScript SDK repository. It supersedes the previous `.github/copilot-instructions.md`.
---

# Agent Information for Camunda 8 JavaScript SDK

This file is the **canonical** instruction document for AI agents (Copilot, Claude, etc.) working on the Camunda 8 JavaScript SDK repository. It supersedes the previous `.github/copilot-instructions.md`.

See also:
- [CONTRIBUTING.md](CONTRIBUTING.md) — contributor onboarding, branch model, release summary
- [MAINTAINER.md](MAINTAINER.md) — operational guide (CI workflows, semantic-release, npm publishing)

---

## Project Overview

The Camunda 8 JavaScript SDK is the official TypeScript/Node.js client library for Camunda Platform 8. It provides unified access to all Camunda 8 APIs including Zeebe, Operate, Optimize, Tasklist, Modeler, and Admin. It targets Node.js (not browser) and ships TypeScript typings.

### Key Features

- Full-featured clients for all Camunda 8 services
- Support for both SaaS and Self-Managed deployments
- OAuth authentication with token management
- Specialized handling for int64 values through the `LosslessDto` system
- Comprehensive testing suite for unit and integration tests

---

## Commit Conventions (read this before committing)

The repo enforces [Conventional Commits](https://www.conventionalcommits.org/) via `commitlint` (see [commitlint.config.js](commitlint.config.js)). The CI job `lint-commits` will block PRs that violate these rules.

### Allowed commit types

Only these `type:` values are accepted (any other type — including `deps:` — will fail `lint-commits`):

| Type           | When to use                                                      | Release impact |
| -------------- | ---------------------------------------------------------------- | -------------- |
| `feat`         | New SDK feature                                                  | patch          |
| `fix`          | Bug fix                                                          | patch          |
| `perf`         | Performance improvement                                          | patch          |
| `revert`       | Revert a previous commit                                         | patch          |
| `release`      | Force a patch release with no other eligible commits             | patch          |
| `server`       | Bump to a new Camunda **minor** line (e.g. 8.8 → 8.9)            | minor          |
| `server-major` | Bump to a new Camunda **major** line (e.g. 8.x → 9.0)            | major          |
| `chore`        | Maintenance, dependency bumps (`chore(deps): ...`), tooling      | none           |
| `build`        | Build system / packaging changes                                 | none           |
| `ci`           | CI workflow / GitHub Actions changes                             | none           |
| `docs`         | Documentation only                                               | none           |
| `refactor`     | Code refactor with no behavior change                            | none           |
| `style`        | Whitespace / formatting only                                     | none           |
| `test`         | Test-only changes                                                | none           |

Note the **mutated semver**: `feat`/`fix` produce **patch** bumps, not minor/major. The SDK's version tracks the Camunda 8 server line, not its own API surface. See [MAINTAINER.md](MAINTAINER.md) for the full release pipeline.

### Scopes

`commitlint` does not enforce a scope-enum, so any scope is accepted. Use scopes when they clarify the change (e.g. `chore(deps): ...`, `fix(zeebe): ...`, `ci(release): ...`).

### Format constraints

- Body lines: max **500** characters per line (`body-max-line-length`).
- Footer lines: max **1000** characters per line (`footer-max-line-length`).
- Header (first line): conventional-commits default — keep concise.

### Common pitfalls

- **`deps:` is not valid.** Use `chore(deps): ...` for dependency pinning / bumps. (Renovate is configured to use `chore(deps):` as well.)
- The pre-commit Husky hook runs `npm run test`, which can fail on local-only files (e.g. nested worktrees). The release workflow bypasses this with `HUSKY=0`. For agent commits that are otherwise validated, prefix with `HUSKY=0` rather than `--no-verify`.

---

## Essential Commands

### Build & Compilation

```bash
npm run build          # Clean, compile TypeScript, copy proto files
npm run clean          # Remove dist folder and build artifacts
npm run compile        # Compile TypeScript to JavaScript
```

### Testing

The repo uses **vitest** (not jest, despite older docs).

```bash
npm test               # Run unit tests only (CAMUNDA_UNIT_TEST=true)
npm run test:smoketest # Build + smoke test + tsd typings check
npm run test:8.8:sm    # 8.8 self-managed integration
npm run test:8.8:saas  # 8.8 SaaS integration
npm run test:8.7:sm    # 8.7 self-managed integration
npm run test:8.7:mt    # 8.7 multi-tenancy integration
npm run test:8.7:saas  # 8.7 SaaS integration
npm run test:c8run     # C8Run integration
```

Run a single test file:

```bash
# Unit test — source unit env first
. env/unit-test.env && CAMUNDA_UNIT_TEST=true npx vitest run --testTimeout=30000 ${testFile}


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [camunda/camunda-8-js-sdk](https://github.com/camunda/camunda-8-js-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
