---
trigger: always_on
description: You are a senior TypeScript engineer working in this pnpm monorepo. Your expertise covers strict type systems, dependency injection, agent runtime lifecycle, and multi-provider AI integration. Follow every rule in this file without exception.
---

# AGENTS.md — Robota Monorepo Agent Guidelines

You are a senior TypeScript engineer working in this pnpm monorepo. Your expertise covers strict type systems, dependency injection, agent runtime lifecycle, and multi-provider AI integration. Follow every rule in this file without exception.

This file is the entry point for all agent guidance in the Robota monorepo.

## Document Discovery Policy

This file contains only domain-free rules and routing. It does not contain package-specific knowledge, domain logic, or implementation details.

**Progressive discovery model:**

1. **Start here.** Read this file for non-negotiable rules and document routing.
2. **Follow links.** For domain details, follow the references to rules, skills, specs, or structure documents.
3. **Dig into packages.** For package-specific contracts, read `packages/<name>/docs/SPEC.md`.

**Principles:**

- This file must remain domain-free. It must not reference individual package names, classes, or domain concepts.
- Domain-specific rules belong in skills (`.agents/skills/`) or package specs (`docs/SPEC.md`).
- Never duplicate content across levels. Each fact has exactly one owner document.
- When a rule is needed repeatedly, prefer a mechanical check over adding more prose.

**Document tree:**

| Document                                                                 | Purpose                                                                                                           |
| ------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------- |
| [.agents/rules/index.md](.agents/rules/index.md)                         | Rule group listing and routing                                                                                    |
| [.agents/rules/code-quality.md](.agents/rules/code-quality.md)           | Type system, imports, dev patterns                                                                                |
| [.agents/rules/process.md](.agents/rules/process.md)                     | Routing file → spec-workflow, tdd-and-planning, verification, publish, backlog-execution (done gate), operational |
| [.agents/rules/api-boundary.md](.agents/rules/api-boundary.md)           | API specs and application lifecycle                                                                               |
| [.agents/rules/naming-style.md](.agents/rules/naming-style.md)           | Language policy, agent identity, styling                                                                          |
| [.agents/rules/git-branch.md](.agents/rules/git-branch.md)               | Git ops and branch policy                                                                                         |
| [.agents/rules/common-mistakes.md](.agents/rules/common-mistakes.md)     | Observed failure patterns                                                                                         |
| [.agents/project-structure.md](.agents/project-structure.md)             | Package listing and dependency rules                                                                              |
| [.agents/skills/index.md](.agents/skills/index.md)                       | All procedural workflow skills                                                                                    |
| [.agents/backlog/README.md](.agents/backlog/README.md)                   | Future work items and backlog process                                                                             |
| [.agents/templates/spec-template.md](.agents/templates/spec-template.md) | SPEC.md authoring template                                                                                        |
| [.agents/specs/README.md](.agents/specs/README.md)                       | Cross-cutting specs that span multiple packages                                                                   |
| [.agents/evals/README.md](.agents/evals/README.md)                       | Agent quality evaluation datasets and metrics                                                                     |
| `packages/*/docs/SPEC.md`                                                | Package-level contracts (SSOT) — one per package                                                                  |

## Project Overview

TypeScript/JavaScript monorepo for building AI agents with multi-provider support. Uses a pnpm workspace with strict TypeScript and ESLint.

- Package manager: `pnpm` 8.15.4
- Node.js: 22.14.0 (Volta), minimum 18.0.0
- Module system: ES modules only (`"type": "module"`)
- Repository: <https://github.com/woojubb/robota.git>

## Project Structure

See [`.agents/project-structure.md`](.agents/project-structure.md) for the full package and app listing, including dependency direction rules.

## Common Commands

```bash
pnpm install
pnpm build
pnpm build:deps
pnpm --filter @robota-sdk/<pkg> build
pnpm test
pnpm test:coverage
pnpm typecheck
pnpm lint
pnpm docs:build
```

## Harness Entrypoints

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [woojubb/robota](https://github.com/woojubb/robota) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
