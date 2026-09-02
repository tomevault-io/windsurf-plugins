---
trigger: always_on
description: This document describes the process for going from zero to full Alchemy coverage for a single AWS service:
---

# AWS Service Bring-Up Process

This document describes the process for going from zero to full Alchemy coverage for a single AWS service:

- all canonical resources
- all bindings
- all event sources
- all ergonomic helpers
- deterministic audit and test coverage checks

Use this process whenever adding a brand new AWS service or finishing an incomplete one.

## Goal

For a given AWS service, the end state should include:

1. Every canonical Alchemy resource for that service.
2. Every important AWS API operation represented either as:
   - a binding,
   - a resource lifecycle provider,
   - an event source surface,
   - or an intentional helper abstraction.
3. Runtime-specific event-source implementations where applicable.
4. End-to-end tests covering the implemented binding and event-source surface.
5. Deterministic audit checks that report what is still missing.

## Source Of Truth

Start from the distilled spec in:

- `.vendor/distilled/@distilled.cloud/aws/src/services/<service>.ts`

Never start from ad-hoc memory of the AWS service. The distilled spec is the source of truth for operations.

## Core Concepts

Every distilled operation must be classified into one of these buckets:

### 1. Binding

Use a binding when the operation is a runtime capability.

Examples:

- `GetItem(table)`
- `PutItem(table)`
- `ListTables()`
- `DescribeTable(table)`

Bindings are:

- one file per operation
- the combined `Binding.Service` form (`interface X extends Binding.Service<X, "id", Shape>` + `const X = Binding.Service<X>("id")`); the deploy-time IAM registration is inlined into the impl layer under `if (!globalThis.__ALCHEMY_RUNTIME__)`, resolving the host via `yield* Binding.host`
- usually named `alchemy/src/AWS/<Service>/<Operation>.ts` (callable + types) with the impl layer in `<Operation>Http.ts` (AWS runtime impls call the distilled HTTP API authenticated by the Lambda's IAM role; `Http`, not `Binding` — `Binding` is a Cloudflare native-worker concept)

### 2. Resource

Use a resource when the operation set implies lifecycle ownership of infrastructure.

Examples:

- `createTable` / `updateTable` / `deleteTable` -> `Table`
- `createBucket` / `deleteBucket` -> `Bucket`

Resources are:

- canonical Alchemy infrastructure entities
- implemented as `Resource` contract + provider in a single file

### 3. Event Source

Use an event source when the service can push records/events into a runtime.

This always has two layers:

1. Service-level abstraction in `alchemy/src/AWS/<Service>/...`
2. Runtime-specific implementation in places like:
   - `alchemy/src/AWS/Lambda/...`
   - `alchemy/src/Process/...`

Examples:

- `consumeBucketEvents(bucket, handler)`
- `consumeQueueMessages(queue, handler)`
- `consumeTableChanges(table, handler)` for DynamoDB-style change streams

### 4. Helper

Use a helper when multiple raw operations should collapse into a more ergonomic surface.

Examples:

- `consumeBucketEvents(bucket, handler)`
- `consumeQueueMessages(queue, handler)`
- batch or transaction wrappers

Helpers should not hide missing low-level primitives. Implement the primitives first.

## Resource Arity

Classify each binding by resource arity:

- `0`: service/account scoped
  - example: `ListTables`
- `1`: one resource
  - example: `GetItem(table)`
- `2+`: multiple resources
  - example: `RestoreTableToPointInTime(fromTable, toTable)`
  - example: copy, batch, or transaction style operations

This classification helps decide:

- binding shape
- helper shape
- policy shape
- whether the operation belongs on a resource or service surface

Arity should be modeled in terms of canonical resources whenever possible.

- good: a `2`-arity binding accepts `<From extends Table, To extends Table>`
- bad: a `2`-resource operation accepts one `Table` plus a raw `string` target name
- only fall back to raw identifiers when there is no real canonical resource to bind against
- when there's a missing canonical resource, that might suggest we need to add one

### Case Study: `ExecuteTransaction`

Use `ExecuteTransaction` as the reference pattern for bindings that touch `1..*` canonical resources.

The ambiguity we want to avoid is:

- bad: `ExecuteTransaction()` with IAM `Resource: ["*"]`
- bad: `ExecuteTransaction(tableNames: string[])`
- bad: a SID like `AWS.DynamoDB.ExecuteTransaction(2 table(s))` that hides which resources were bound

The required pattern is:

- good: `ExecuteTransaction(tableA, tableB, ...)`
- good: the binding type requires at least one table
- good: the policy enumerates exactly those table ARNs
- good: the SID is deterministic and names the participating resources

Runbook for any `1..*` resource-bound binding:

1. Model the binding arguments as a non-empty tuple of canonical resources.
2. Call `.bind(resourceA, resourceB, ...)`, never `.bind()` with hidden resource discovery.
3. Before constructing the SID, sort the resources by `LogicalId` so equivalent calls produce the same binding identity.
4. Pass the sorted resource array into the `host.bind` template so the SID renders each resource name explicitly, for example `AWS.DynamoDB.ExecuteTransaction(TableA, TableB)`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alchemy-run/alchemy-effect](https://github.com/alchemy-run/alchemy-effect) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-02 -->
