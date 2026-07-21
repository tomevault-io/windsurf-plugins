---
trigger: always_on
description: This file provides guidance to AI agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI agents when working with code in this repository.

## Project Overview

IAM Floyd is an AWS IAM policy statement generator with a fluent interface. It generates TypeScript classes for all AWS services and their actions, resources, and condition keys from AWS documentation. The project supports both standalone usage (`iam-floyd`) and AWS CDK integration (`cdk-iam-floyd`).

## Core Architecture

### Generated Code Structure

- `lib/generated/policy-statements/` - Generated TypeScript class per AWS service (460+ files)
- `lib/generated/index.ts` - Re-exports all service classes
- `lib/generated/aws-managed-policies/` - Generated AWS managed policies
- `lib/shared/` - Hand-written core: `PolicyStatement`, `All`, `Operator`, `AccessLevel`
- `lib/collection/` - Predefined policy collection utilities
- `lib/generator/` - Scrapes AWS docs and generates `lib/generated/` via `cheerio` + `ts-morph`

### PolicyStatement Inheritance Chain

Built in 10 numbered layers (`lib/shared/policy-statement/`):

```
1-base → 2-conditions → 3-actions → 4-resources → 5-effect
       → 6-arn-defaults → 8-principals → 10-final (PolicyStatement)
```

Each `*.CDK.ts` file is the CDK variant of that layer (swapped in by `bin/mkcdk.ts`).

### Dual Package Strategy

One codebase produces two npm packages:

- `iam-floyd` - Standalone (uses built-in base class)
- `cdk-iam-floyd` - Extends `aws_iam.PolicyStatement` from AWS CDK

`bin/mkcdk.ts` transforms between variants by swapping `*.CDK.ts` files and rewriting constructors.

## Development Commands

### Build

```bash
make build           # tsc --build --force tsconfig.main.json
make package         # build + npm pack
make clean           # remove node_modules, *.js, *.d.ts
make install         # clean + npm i
```

### Code Generation

```bash
make generate        # generate lib/generated/ from AWS docs (25hr cache)
make generate-force  # NOCACHE=1 - ignores time-based cache
make index-managed-policies  # regenerate AWS managed policies index
```

### Testing

The project has **no unit test framework**. Tests are integration-style: TypeScript examples are compiled, run, and their output is diffed against stored `.result` files.

```bash
make test            # compile examples/ + diff against *.result files (standalone)
make test-typescript # same, via Test.TypeScript.Makefile
make cdk-test        # CDK test: real deploy + destroy via AWS CDK
make cdk-all         # cdk + install + build + cdk-test
```

**Run a single example test manually:**

```bash
# 1. Compile a single example
npx tsc -p tsconfig.test-iam-floyd.json

# 2. Run and compare output
node examples/allow/allow.js > /tmp/out.txt
diff /tmp/out.txt examples/allow/allow.result

# Or run the integration test harness directly:
npx ts-node test/main.ts
```

**Regenerate expected results** (after intentional changes):

```bash
make regenerate-code-example-results
```

### Linting

```bash
make eslint          # npx eslint .
```

### CDK Variant

```bash
make cdk             # transforms codebase to CDK variant (modifies lib/shared, lib/generated, package.json)
make uncdk           # reverts via git stash (lib/generated, lib/shared, package.json)
```

## Fixing AWS Documentation Errors (`lib/generator/fixes.ts`)

The generator scrapes live AWS docs, which sometimes contain errors or inconsistencies. `fixes.ts` is the central place to patch these before code is generated. **When the generator produces wrong output, add a fix here rather than editing generated files.**

### `fixes` object (keyed by URL slug)

Each top-level key is the URL slug of a service's IAM docs page (e.g. `ec2`, `ssm`, `'neptune-db'`). Supported sub-keys:

| Sub-key                              | Effect                                                                                                                                                        |
| ------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ignore: true`                       | Skip generating this service entirely (used for EOL services)                                                                                                 |
| `name: 'slug'`                       | Override the generated filename and class name (needed when the same service prefix spans multiple doc pages, e.g. `pinpointemailservice` → `ses-pinpoint`)   |
| `service: 'prefix'`                  | Override the IAM service prefix used in the generated code                                                                                                    |
| `resourceTypes.<name>.arn`           | Replace the ARN template for a resource type with a corrected one                                                                                             |
| `conditions.<key>.key`               | Rewrite the condition key string (used when docs have a concrete example key like `RequestTag/tag-key` instead of the parametric form `RequestTag/${TagKey}`) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [udondan/iam-floyd](https://github.com/udondan/iam-floyd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
