---
trigger: always_on
description: This file applies to the entire repository. Keep changes scoped, preserve
---

# Repository Agent Guide

This file applies to the entire repository. Keep changes scoped, preserve
unrelated work, and use `gofmt` for every modified Go file.

## Build and test

- Build: `go build ./cmd`
- Full test suite: `DWS_PACKAGE_VERSION=0.0.0-test go test ./...`
- Generate Schema assets: `go generate ./internal/cli`
- Check generated drift: `./scripts/policy/check-generated-drift.sh`
- Check the Schema contract: `./scripts/policy/check-schema-catalog.sh`

Generated Schema JSON is committed. Change its source inputs and generators,
then regenerate; do not hand-edit generated Catalog or Agent metadata files.
`internal/cli/schema_command_registry.json` is different: it is a reviewed
`CommandRegistry` source, not a generated snapshot. It is the single reviewed
source of stable canonical identity,
primary paths, aliases, and navigation. Edit it only when reviewed exposure,
identity, primary path, or aliases change; parameter, Skill, and metadata-only
changes must not rewrite it mechanically.

## Agent Schema contract

The Schema data flow is one way:

```text
1. app.NewRootCommand()
   └─ builds the real Cobra command tree and flags

2. schema_command_registry.json
   + schema_hints/metadata/<product>.json tool parameters (+ cli_path)
   └─ forms EffectiveCommandRegistry
      └─ binds exactly to real Cobra leaves and aliases

3. Parameter resolution
   Cobra flags
   + schema_parameter_bindings.json
   + metadata tool parameters
   └─ produces ParameterSpec and constraints

4. Agent and interface semantics
   schema_hints/selection/<product>.json   (selection prose)
   + schema_hints/metadata/<product>.json  (safety/interface/runtime_gate)
   + pinned MCP metadata
   └─ resolves Agent metadata by source precedence
      Markdown is evidence only; it is not concatenated into final prose

5. One typed hub
   BoundCommandRegistry
   + ParameterSpec
   + Agent metadata
   + Interface metadata
   └─ resolves every command exactly once into ToolSpec
      └─ aggregates SchemaRegistry + SchemaIndex

6. One-way publication
   SchemaRegistry
   └─ internal/cli/schema_catalog.json
      └─ dws schema list/product/group/leaf/--all
```

Parameter overlays from metadata are merged into `EffectiveCommandRegistry`
*before* Cobra binding; after that point there is no second identity source and
no identity precedence winner. The binder must reject a missing/non-runnable
Cobra path, an alias collision, and any native identity annotation that
disagrees with the effective registry. A missing native identity annotation is
allowed because annotations are implementation-side assertions, not identity
fallbacks.

The assembler resolves every bound command exactly once into one `ToolSpec`.
Build-time gates and the snapshot serializer consume that source-resolved typed
registry/index. Runtime projections and delivery gates consume the typed
registry/index returned by the production snapshot loader. Neither path may
reopen annotations, merge source records, or use a previous Catalog or other
generated JSON as a source. `schema_catalog.json` is output-only in the
generation graph. The production loader decoding the embedded published
snapshot is a delivery boundary, not source resolution; it must never create or
repair a Cobra command, flag, registry entry, or later Catalog generation.

This split is architecturally isomorphic to Lark's typed metadata registry,
navigation catalog, and schema renderer. DWS intentionally preserves its
existing flat JSON wire contract for compatibility; do not treat architectural
alignment as permission to make an unversioned wire-format change.

The reviewed `CommandRegistry` is the sole source of stable command identity
and navigation. The executable Cobra tree remains the source of truth for
whether a CLI path exists, is runnable, and which flags it accepts. Schema
coverage is bidirectional:

1. Every final `SchemaRegistry` tool, including its serialized Catalog
   projection, must resolve to an executable Cobra command.
2. Every public runnable Cobra leaf must either resolve to Schema or appear as
   an exact, reviewed exclusion with a non-empty reason in
   `internal/cli/schema_command_exclusions.json`.

Do not use prefix or wildcard exclusions: they can silently hide future
commands. Remove an exclusion when its command enters Schema; stale, invalid,
or duplicate exclusions must fail generation and CI.

When adding or changing an Agent-visible command, review all relevant inputs:

- `internal/cli/schema_command_registry.json` for the reviewed
  `CommandRegistry`: canonical identity, primary CLI path, aliases, and stable
  navigation. It is the identity source and is not a generated artifact.
- `internal/cli/schema_command_registry.schema.json` is its closed,
  machine-readable editing contract. Preserve the local `$schema` reference;
  unknown fields, invalid visibility values, stale paths, and collisions fail
  Go validation and policy.
- `internal/cli/schema_hints/metadata/<product>.json` for safety, interface,
  `runtime_gate`, and optional parameter overlays (`parameters` / `cli_path`).
- `internal/cli/schema_hints/selection/<product>.json` for reviewed Agent
  selection prose (`agent_summary`, `use_when`, `avoid_when`, `examples`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DingTalk-Real-AI/dingtalk-workspace-cli](https://github.com/DingTalk-Real-AI/dingtalk-workspace-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
