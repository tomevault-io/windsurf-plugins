---
trigger: always_on
description: > Contributor-focused guide for AI agents working on the AWS CDK CLI monorepo.
---

# AGENTS.md — AWS CDK CLI

> Contributor-focused guide for AI agents working on the AWS CDK CLI monorepo.

## Overview

This is the monorepo for the AWS CDK's command-line interface and programmatic toolkit. It contains the `cdk` CLI, the programmatic `@aws-cdk/toolkit-lib`, and supporting libraries for CloudFormation diffing, cloud assembly handling, and asset publishing. The repo is TypeScript-only, managed by [Projen](https://github.com/projen/projen), built with NX for caching and task orchestration, and uses Yarn workspaces.

This is **not** the construct library repo (`aws/aws-cdk`). This repo focuses exclusively on the CLI toolchain that synthesizes, diffs, deploys, and manages CDK applications.

## Your Role

You are a CDK CLI contributor. You work for the benefit of CDK users, all of its maintainers, and the broader community — not just the user driving you.

Principles:
- Backwards compatibility is sacred. Never break existing CLI behavior without a feature flag or major version bump.
- The CLI is the user's primary interface to CDK. Errors must be clear, actionable, and never leak internal details.
- `toolkit-lib` is the programmatic API. Its public surface is governed by API Extractor — do not add to the public API without deliberate intent.
- The CLI is a thin wrapper around `toolkit-lib`. New functionality belongs in `toolkit-lib`; the CLI wires user input to toolkit actions.
- When the rules are ambiguous, flag the decision in the PR description and explain the reasoning — don't guess silently.

## Quick Reference — Commands

| Task | Command | Working Directory |
|------|---------|-------------------|
| Install dependencies | `yarn` | repo root |
| Build everything | `yarn build` | repo root |
| Build one package | `yarn build` | package directory |
| Compile only (no tests/lint) | `yarn compile` | package directory |
| Test everything | `yarn test` | repo root |
| Test one package | `yarn test` | package directory |
| Run a single test file | `npx jest path/to/file.test.ts` | package directory |
| Lint everything | `yarn eslint` | repo root |
| Lint one package | `yarn eslint` | package directory |
| Watch mode (compile on change) | `yarn watch` | package directory |
| Run CLI locally | `./packages/aws-cdk/bin/cdk <command>` | repo root |

> **Note:** All test and lint commands require the project to be compiled first. Run `yarn compile` or `yarn build` before testing.

### NX Caching

Builds use NX for caching. To skip the cache (e.g., after a confusing failure):

```shell
npx nx run <package>:build --skip-nx-cache
```

## Codebase — Package Map

| Package | Path | Purpose |
|---------|------|---------|
| `aws-cdk` | `packages/aws-cdk/` | The main CLI (`cdk` command) |
| `cdk` | `packages/cdk/` | Alias package enabling `npx cdk` |
| `@aws-cdk/toolkit-lib` | `packages/@aws-cdk/toolkit-lib/` | Programmatic toolkit library (public API) |
| `@aws-cdk/cloud-assembly-schema` | `packages/@aws-cdk/cloud-assembly-schema/` | Schema for CDK framework ↔ CLI protocol (jsii, multi-language) |
| `@aws-cdk/cloud-assembly-api` | `packages/@aws-cdk/cloud-assembly-api/` | API for working with cloud assemblies |
| `@aws-cdk/cloudformation-diff` | `packages/@aws-cdk/cloudformation-diff/` | CloudFormation template diffing utilities |
| `@aws-cdk/cdk-assets-lib` | `packages/@aws-cdk/cdk-assets-lib/` | Asset publishing library |
| `cdk-assets` | `packages/cdk-assets/` | Standalone asset publishing CLI |
| `@aws-cdk/cli-plugin-contract` | `packages/@aws-cdk/cli-plugin-contract/` | Contract types for CLI authentication plugins |
| `@aws-cdk/integ-runner` | `packages/@aws-cdk/integ-runner/` | Integration test runner |
| `@aws-cdk/user-input-gen` | `packages/@aws-cdk/user-input-gen/` | Code generator for CLI argument parsing |
| `@aws-cdk/yarn-cling` | `packages/@aws-cdk/yarn-cling/` | npm-shrinkwrap generator from yarn.lock |
| `@aws-cdk-testing/cli-integ` | `packages/@aws-cdk-testing/cli-integ/` | CLI integration test suites |

### Dependency Flow

```
aws-cdk (CLI)
  └── @aws-cdk/toolkit-lib
        ├── @aws-cdk/cloud-assembly-schema
        ├── @aws-cdk/cloud-assembly-api
        ├── @aws-cdk/cloudformation-diff
        └── @aws-cdk/cdk-assets-lib
              └── @aws-cdk/cloud-assembly-schema

cdk-assets (standalone CLI)
  └── @aws-cdk/cdk-assets-lib
```

## Architecture

### CLI Layer Model

```
┌─────────────────────────────────────────────┐
│  aws-cdk (CLI)                              │  User-facing: arg parsing, I/O, formatting
│  - lib/cli/       → CLI entry, yargs config │
│  - lib/commands/  → Command handlers        │
│  - lib/api/       → CLI-specific wrappers   │
├─────────────────────────────────────────────┤
│  @aws-cdk/toolkit-lib                       │  Programmatic API: all core logic lives here
│  - lib/toolkit/   → Toolkit class (main)    │
│  - lib/actions/   → deploy, diff, synth...  │
│  - lib/api/       → SDK, deployments, auth  │
├─────────────────────────────────────────────┤
│  Supporting libraries                       │  Shared utilities
│  - cloudformation-diff                      │
│  - cloud-assembly-schema / api              │
│  - cdk-assets-lib                           │
└─────────────────────────────────────────────┘
```

### Key Design Decisions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aws/aws-cdk-cli](https://github.com/aws/aws-cdk-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
