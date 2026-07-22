---
trigger: always_on
description: provides tooling for merging multiple OpenAPI 3.0 specification files into a single,
---

# AGENTS.md

> Generated: Thursday, 2026-05-21 08:46:03
>
> This file is a guide for AI coding agents (and humans) working in this repository.
> It describes the project structure, functionality, build/test/deployment workflows,
> coding conventions, and helpful commands.

---

## 1. Project Overview

This repository — **`openapi-merge`** — is a TypeScript multi-package mono-repo that
provides tooling for merging multiple OpenAPI 3.0 specification files into a single,
deterministic OpenAPI specification.

The primary motivating use case is:

> "I have multiple microservices, each with their own OpenAPI spec, and I want to
> expose them through one API Gateway with a single combined spec."

While the merging logic is generic enough for many use cases, many design decisions
favor the API-gateway scenario. The merge is deterministic: the first input listed
takes precedence over later inputs for any element-level conflicts (e.g., `info`,
`servers`, `security`, `externalDocs`).

The repository is owned and maintained by **Robert Massaioli** and published under
the **MIT License**. Source is hosted at
<https://github.com/robertmassaioli/openapi-merge>.

---

## 2. Repository Layout

```
.
├── .github/workflows/        # CI workflows: branch-test, npm-publish, codeql-analysis
├── .husky/pre-commit         # Husky pre-commit hook (runs `yarn lint`)
├── .nvmrc                    # Pinned Node major version (14)
├── LICENSE                   # MIT
├── README.md                 # Repository-level README
├── package.json              # Root package — orchestrates the mono-repo via `bolt`
└── packages/
    ├── openapi-merge/        # Library package (published as `openapi-merge` on npm)
    │   ├── src/              # Library source
    │   ├── src/__tests__/    # Jest test suites for the library
    │   ├── jest.config.js
    │   ├── tsconfig.json
    │   └── package.json
    └── openapi-merge-cli/    # CLI package (published as `openapi-merge-cli` on npm)
        ├── src/              # CLI source (entrypoint: `cli.ts` → `index.ts`)
        ├── confluence.swagger.yaml   # Example OpenAPI input used for manual testing
        ├── openapi-merge.test.json   # Example merge configuration
        ├── tsconfig.json
        └── package.json
```

### Package manager / workspace tooling

The mono-repo uses **[bolt](https://github.com/boltpkg/bolt)** (a thin wrapper over
Yarn) for workspace orchestration. The root `package.json` declares:

```json
"bolt": { "workspaces": ["packages/*"] }
```

Most root-level scripts are bolt commands that fan out into each package:

| Script             | What it does                                                       |
| ------------------ | ------------------------------------------------------------------ |
| `yarn lint`        | `bolt ws lint` — runs `lint` in every workspace                    |
| `yarn test`        | `bolt ws test` — runs `test` in every workspace                    |
| `yarn cli`         | `bolt w openapi-merge-cli run start` — runs the CLI in dev mode   |
| `yarn prepare`     | `husky install` — installs the git hooks                           |

---

## 3. The `openapi-merge` Library (`packages/openapi-merge`)

### Purpose

Provides a single `merge(inputs)` function that takes an array of OpenAPI 3.0
documents (plus per-input options) and produces a single combined OpenAPI 3.0.3
document, or an error describing why the merge failed.

### Public API (re-exported from `src/index.ts`)

```ts
import {
  merge,
  isErrorResult,
  MergeInput,
  MergeResult,
  PathModification,
  OperationSelection,
} from 'openapi-merge';
```

- `merge(inputs: MergeInput): MergeResult`
- `isErrorResult(result): result is ErrorMergeResult`

### Key types (`src/data.ts`)

- **`SingleMergeInput`** — a single input. Two backwards-compatible shapes:
  - `SingleMergeInputV1` (deprecated): uses `disputePrefix?: string`.
  - `SingleMergeInputV2`: uses `dispute?: Dispute` (a `DisputePrefix | DisputeSuffix`,
    each with an optional `alwaysApply` flag).
- **`PathModification`** — `{ stripStart?: string; prepend?: string }` applied to
  every path imported from the input. `stripStart` runs before `prepend`.
- **`OperationSelection`** — `{ includeTags?: string[]; excludeTags?: string[] }`.
  Exclusion takes precedence when both apply to the same operation.
- **`DescriptionMergeBehaviour`** — `{ append: boolean; title?: DescriptionTitle }`
  controls how `info.description` from each input contributes to the merged
  `info.description` field (with optional Markdown heading).
- **`MergeResult`** — `SuccessfulMergeResult { output } | ErrorMergeResult { type, message }`.
- **`ErrorType`** — `'no-inputs' | 'duplicate-paths' | 'component-definition-conflict' | 'operation-id-conflict'`.

### Source modules (one responsibility each)

| File                          | Responsibility                                                                                  |
| ----------------------------- | ----------------------------------------------------------------------------------------------- |
| `index.ts`                    | The top-level `merge` orchestrator.                                                             |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [robertmassaioli/openapi-merge](https://github.com/robertmassaioli/openapi-merge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
