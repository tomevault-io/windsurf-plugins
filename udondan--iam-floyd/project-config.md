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

- `lib/generated/model/` - Service model per AWS service (JSON, committed). The single source of truth for all generated code
- `lib/generated/policy-statements/` - TypeScript class per AWS service, emitted from the model (not committed)
- `lib/generated/index.ts` - Re-exports all service classes (emitted, not committed)
- `lib/generated/aws-managed-policies/` - Generated AWS managed policies (committed)
- `lib/generated/aws-service-principals/` - List of AWS service principals (`principals.json`, committed) and the class `AwsServicePrincipal` emitted from it (`index.ts`, not committed)
- `lib/shared/` - Hand-written core: `PolicyStatement`, `PolicyDocument`, `All`, `Operator`, `AccessLevel`
- `lib/collection/` - Predefined policy collection utilities
- `lib/generator/` - Scrapes AWS docs with `cheerio` into the model (`model.ts`), and emits TypeScript from the model with `ts-morph` (`emit/typescript.ts`), and the index of the policy converter (`emit/converter.ts`)

### PolicyStatement Inheritance Chain

Built in 10 numbered layers (`lib/shared/policy-statement/`):

```text
1-base → 2-conditions → 3-actions → 4-resources → 5-effect
       → 6-arn-defaults → 8-principals → 10-final (PolicyStatement)
```

Each `*.CDK.ts` file is the CDK variant of that layer (swapped in by `bin/mkcdk.ts`).

The policy documents (`lib/shared/policy/`) build on `1-base`, which holds the statements: in the CDK variant (`1-base.CDK.ts`) `PolicyBase` extends `aws_iam.PolicyDocument` and reads its private `statements`. `PolicyDocument` in `2-final.ts` takes the maximum size and the statements, estimates the size of the policy like the AWS CDK (ARNs with tokens count as `arnSizeEstimate`, 150, actions with tokens as 20), plus the parts the AWS CDK leaves out (`Version`, braces, `Sid`, principal keys), validates it against the maximum size, and splits it first fit like `PolicyDocument._splitDocument` (the protected `splitDocument`). `3-documents.ts` has one subclass per type of policy (`ManagedPolicyDocument`, `InlineRolePolicyDocument`, `S3BucketPolicyDocument`, …), which takes only the statements; only `ManagedPolicyDocument`, `ServiceControlPolicyDocument` and `ResourceControlPolicyDocument` have a public `split()`.

### Dual Package Strategy

One codebase produces two npm packages:

- `iam-floyd` - Standalone (uses built-in base class)
- `cdk-iam-floyd` - Extends `aws_iam.PolicyStatement` from AWS CDK

`bin/mkcdk.ts` transforms between variants by swapping `*.CDK.ts` files and emitting the CDK variant of the service classes from the model.

### CDK Constructs in `on*()` Methods

In the CDK variant, the `on*()` method of a resource type with a single required placeholder also takes a construct, if aws-cdk-lib has a reference interface for it (`aws-cdk-lib/interfaces`, e.g. `onFunction(fn)` with `interfaces.aws_lambda.IFunctionRef`). The method then uses the ARN of the reference (`fn.functionRef.functionArn`), or, if the reference has no ARN, its identifier in place of the placeholder. `lib/generator/cdk-refs.ts` matches the resource types of the model with the reference interfaces of the installed aws-cdk-lib (service prefix ↔ module, resource type ↔ interface) and writes them as `cdkRef` into the model; it runs with `make generate` and alone with `make cdk-refs`. `lib/generated/cdk-refs.json` (committed) has the interfaces in use and the minimum version of aws-cdk-lib, the peer dependency of `cdk-iam-floyd`, which is raised to the installed version only when interfaces are added, and the version of constructs this aws-cdk-lib requires, the other peer dependency (lower, NuGet fails with NU1605). Wrong matches are fixed in `fixes.ts` (`cdkModule`, `resourceTypes.<name>.cdkRef`).

### Other Languages (jsii)

`cdk-iam-floyd` is also packaged for Python, Java, .NET and Go with `jsii-pacmak`. The jsii compiler is not used: `lib/generator/emit/jsii.ts` writes the `.jsii` assembly from the model, and `bin/jsii.ts` adds the `jsii` targets to package.json, writes the assembly and appends the jsii type info to `lib/index.js`. The same `.jsii` is what Construct Hub renders the API docs from. jsii-pacmak runs with `--no-runtime-type-checking` and `bin/jsii-pack.ts` as pack command, which embeds an npm tarball without docs and `.d.ts` files in the packages.

`test/jsii/` builds `floyd-consumer`, a jsii library that depends on `cdk-iam-floyd`, and runs the same scenarios in TypeScript (the baseline, without jsii), Python, Java, .NET and Go against `test/jsii/expected.json`. It also runs the examples of the docs in each language (`examples/<name>/<name>.{py,java,cs,go}`, through the runners in `test/jsii/examples/`) and compares them to the `.result` files. Every example needs a file in every language.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [udondan/iam-floyd](https://github.com/udondan/iam-floyd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
