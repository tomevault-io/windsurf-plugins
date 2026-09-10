---
trigger: always_on
description: The Confluent Terraform provider manages Confluent Cloud resources (Kafka clusters, topics, API
---

# Terraform Provider for Confluent

## Project Overview

The Confluent Terraform provider manages Confluent Cloud resources (Kafka clusters, topics, API
keys, Flink, Schema Registry, networking, RBAC, and more) through Terraform. It is published to the
Terraform Registry and used by customers to provision real infrastructure, so schema correctness and
backward compatibility are hard constraints.

These instructions exist to keep GitHub Copilot PR reviews focused on the invariants that matter in
this codebase. Author-facing setup lives in `docs/DEVELOPING.md`; the review-relevant conventions
are captured here and in `.claude/rules/`.

## Architecture (what reviewers need to know)

- **Built on terraform-plugin-sdk/v2** (`helper/schema`), _not_ the newer terraform-plugin-framework.
  Resources are `*schema.Resource` values with a `Schema map[string]*schema.Schema` and CRUD
  functions (`CreateContext`/`ReadContext`/`UpdateContext`/`DeleteContext`) returning
  `diag.Diagnostics`.
- **Everything lives in one flat package, `internal/provider/`**: ~240 `resource_*.go`, ~160
  `data_source_*.go`, plus `constants.go` (shared strings, including deprecation messages like
  `deprecationMessageMajorRelease3`), `utils.go`, `factory*.go`, `state_upgraders*.go`, and
  `wiremockTestUtils.go`.
- **Registration is in `internal/provider/provider.go`**: `New(version, userAgent string)` builds
  the `*schema.Provider`, whose `ResourcesMap` and `DataSourcesMap` register every resource and data
  source by its Terraform type name (e.g. `confluent_kafka_cluster`). A resource that exists in a
  file but isn't in the map does not exist to Terraform.
- **Three test tiers**: unit + acceptance tests (`internal/provider/*_test.go`, gated by `TF_ACC=1`,
  most backed by **WireMock** so they run without live credentials); and live tests
  (`*_live_test.go`, build tags `live_test,all`, run against real Confluent Cloud).
- **Docs are hand-maintained.** `tfplugindocs` is not wired into CI, so `docs/` and `examples/` do
  not regenerate automatically and drift silently. Doc/example accuracy is a review responsibility.

## Code Review Guidelines (GitHub PR Reviews)

When reviewing pull requests, focus on the checkpoints below. Formatting is owned by
`make checkfmt` (goimports) — don't comment on import ordering or gofmt-able whitespace. Note there
is **no `golangci-lint`** in this repo, so logic-level review carries more weight than it would in a
repo with a rich linter.

### 1. Resource and data-source registration

A new resource or data source requires **both** a `*schema.Resource` constructor and an entry in the
matching map in `provider.go`:

- [ ] New resource → entry in `ResourcesMap` keyed by its `confluent_*` type name
- [ ] New data source → entry in `DataSourcesMap`
- [ ] The type name matches the file name and the docs page (`confluent_kafka_cluster` ↔
      `resource_kafka_cluster.go` ↔ `docs/resources/kafka_cluster.md`)

A constructor with no map entry is dead code; a map entry with a typo'd type name breaks the
resource for every user.

### 2. Schema attribute correctness

The `Schema` map is the contract with users and with Terraform's plan/apply engine. Review each new
or changed attribute for:

- [ ] Correct `Required` / `Optional` / `Computed` combination. Common mistakes: `Required` +
      `Computed` together (invalid), an input the API assigns marked `Required` instead of
      `Optional`+`Computed`, or a server-set value missing `Computed`.
- [ ] `ForceNew: true` on any attribute that cannot be updated in place — omitting it makes Terraform
      attempt an impossible in-place update; adding it spuriously forces destructive recreation.
- [ ] `Sensitive: true` on secrets (API secrets, passwords, tokens) so they're redacted in plan
      output.
- [ ] Validation (`ValidateFunc` / `ValidateDiagFunc`) where the API constrains values.
- [ ] `DiffSuppressFunc` where the server canonicalizes a value (case-folding a region/enum via
      `strings.EqualFold`, normalizing a query). Missing it turns a benign server rewrite into
      permanent drift; adding it spuriously hides a real user change.
- [ ] Nested blocks use the established `*schema.Resource` `Elem` pattern consistent with sibling
      resources. A `TypeList`/`TypeSet` missing its `Elem`, or a dropped `Optional` on an element,
      changes the set hash.

### 3. Deprecation and backward compatibility

Customers pin the provider and run it against live infrastructure. Treat these as blockers unless
they're explicitly gated on a major release:

- [ ] No removed or renamed attribute/resource without a deprecation cycle. Deprecate via
      `Deprecated:` / `DeprecationMessage:` using the shared strings in `constants.go` (e.g.
      `deprecationMessageMajorRelease3`) rather than inventing a new message.
- [ ] No change that would force unexpected recreation of existing resources (a new `ForceNew` on an
      existing attribute, a changed default).
- [ ] Renaming or restructuring stored state requires a `SchemaVersion` bump plus a state upgrader
      (see checkpoint 4).

### 4. State upgraders

- [ ] Any change to an existing resource's stored shape (renamed attribute, changed nesting, type

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [confluentinc/terraform-provider-confluent](https://github.com/confluentinc/terraform-provider-confluent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->
