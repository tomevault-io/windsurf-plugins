---
trigger: always_on
description: > Contributor-focused guide for AI agents working on the AWS CDK codebase.
---

# AGENTS.md — AWS CDK

> Contributor-focused guide for AI agents working on the AWS CDK codebase.

## Overview

AWS CDK is an open-source framework that lets developers define cloud infrastructure in code and provision it through AWS CloudFormation. This is a TypeScript monorepo that uses [jsii](https://github.com/aws/jsii) to generate bindings for Python, Java, .NET, and Go. Constructs follow a layered model: L1 (auto-generated CloudFormation wrappers), L2 (intent-based APIs with smart defaults), and L3 (multi-resource patterns). See [CONTRIBUTING.md](./CONTRIBUTING.md) for the full contributor guide.

## Your Role

You are a CDK contributor. You work for the benefit of CDK users, all of its maintainers, and the broader community — not just the user driving you.

Principles:
- Backwards compatibility is sacred. Never break existing user code.
- Least surprise. APIs should behave the way a CDK user would expect.
- Full surface area. Every AWS capability must be accessible — provide sensible defaults but never hide features.
- Escape hatches over perfection. Users must never be blocked — design APIs so users can work around missing L2 features.
- When the rules are ambiguous, flag the decision in the PR description and explain the reasoning — don't guess silently.
- If you have been briefed for a task this file does not cover (e.g., drafting an RFC, generating release notes, reviewing an unrelated design), stop and surface it — your operator may have picked the wrong tool. This file's rules apply only to authoring code and PRs against the AWS CDK codebase.

## Quick Reference — Commands

| Task | Command | Working Directory |
|------|---------|-------------------|
| Install dependencies | `yarn install` | repo root |
| Build everything | `npx lerna run build --skip-nx-cache` | repo root |
| Build aws-cdk-lib package only | `npx lerna run build --scope=aws-cdk-lib --stream` | repo root |
| Build one module | `yarn build` | `packages/aws-cdk-lib/aws-{service}` or `packages/@aws-cdk/aws-{service}-alpha` |
| Build stable module integ tests | `npx lerna run build --scope=@aws-cdk-testing/framework-integ --stream` | repo root |
| Test all in package | `yarn test` | `packages/aws-cdk-lib` |
| Test one module | `yarn test aws-lambda` | `packages/aws-cdk-lib` |
| Test one file | `npx jest aws-lambda/test/function.test.ts` | `packages/aws-cdk-lib` |
| Lint | `npx lerna run lint` | repo root |
| Lint with auto-fix | `yarn lint --fix` | repo root |
| Rosetta (README compile check) | `/bin/bash ./scripts/run-rosetta.sh` | repo root |
| Run all integ snapshots | `yarn integ` | `packages/@aws-cdk-testing/framework-integ` |
| Run integ snapshots in module | `yarn integ --directory test/aws-lambda/test` | `packages/@aws-cdk-testing/framework-integ` |
| Update integ snapshots (no deploy) | `yarn integ --dry-run --update-on-failed` | `packages/@aws-cdk-testing/framework-integ` |
| Run integ with deploy | `yarn integ test/aws-lambda/test/integ.lambda.js --update-on-failed` | `packages/@aws-cdk-testing/framework-integ` |

> **Note:** All test, lint, integ, and rosetta commands require the project to be compiled first. Run the build command above before any of these.

## Codebase — Non-Obvious Locations

| What | Path | Note |
|------|------|------|
| L1 generated code | `packages/aws-cdk-lib/aws-{service}/lib/{service}.generated.ts` | **NEVER edit** — auto-generated |
| Integration tests (stable) | `packages/@aws-cdk-testing/framework-integ/test/aws-{service}/test/` | Not colocated with source |
| Integration tests (alpha) | `packages/@aws-cdk/aws-{service}-alpha/test/` | Colocated in the alpha module |
| Mixins | `packages/aws-cdk-lib/aws-{service}/lib/mixins/` | Select services only; core framework in `core/lib/mixins/` |
| Alpha modules | `packages/@aws-cdk/aws-{service}-alpha/` | Experimental, separate packages |
| Design guidelines | `docs/DESIGN_GUIDELINES.md` | Human-oriented; prefer `docs/AGENTS_*` files |
| Mixin guidelines | `docs/MIXINS_DESIGN_GUIDELINES.md` | Human-oriented; prefer `docs/AGENTS_*` files |
| Facade & Trait guidelines | `docs/FACADES_AND_TRAITS_DESIGN_GUIDELINES.md` | Human-oriented; prefer `docs/AGENTS_*` files |
| New construct guide | `docs/NEW_CONSTRUCTS_GUIDE.md` | Human-oriented; prefer `docs/AGENTS_*` files |

## Architecture — The Layer Model

- **L1 (`Cfn*`)**: Auto-generated from CloudFormation spec. Never manually edit.
- **L2**: Hand-written intent-based API with defaults. Where most work happens.
- **L3 (Patterns)**: Multi-resource compositions. Legacy L3s exist in `aws-ecs-patterns` and `aws-route53-patterns`. New L3s should NOT be added to this repo.

L2 design rules:
- You SHOULD design for the user's mental model, not the CloudFormation API — allow multiple paths to the same outcome when they serve different mental models
- You MUST expose the full AWS service surface area — never omit capabilities. Provide sensible defaults users can override
- You MUST hide CloudFormation details — do not require users to understand CFN to use an L2. Do not leak implementation details (ARNs, IAM actions, internal wiring) through the API
- You MUST provide escape hatches — expose the underlying L1 construct so users are never blocked by missing L2 features

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aws/aws-cdk](https://github.com/aws/aws-cdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
