---
trigger: always_on
description: Orientation for AI/agent sessions working in this repo. Read this before touching `crates/core` or
---

# AGENTS.md

Orientation for AI/agent sessions working in this repo. Read this before touching `crates/core` or
`crates/macros`.

## What this project is

Schematic is a layered configuration library. Users write a plain Rust struct/enum, add
`#[derive(Config)]`, and get back:

- A **partial type** (`PartialExample`) where every setting is optional, used for parsing and
  merging config layers (files, env vars, extends chains).
- A **full type** (`Example`) where every setting is populated, produced once all layers are merged
  and finalized.

Everything hard about this codebase comes from that split. If you understand Config↔Partial, the
rest follows.

## Workspace map

| Crate              | Package            | Role                                                                                                           |
| ------------------ | ------------------ | -------------------------------------------------------------------------------------------------------------- |
| `crates/schematic` | `schematic`        | The user-facing runtime: `Config`/`PartialConfig` traits, loader, merge/validate/env helpers, schema renderers. |
| `crates/types`     | `schematic_types`  | `Schema`, `SchemaBuilder`, `SchemaType`, the `Schematic` trait. No macro code.                                  |
| `crates/core`      | `schematic_core`   | **Where the derive lives.** Container/field/variant analysis and all codegen.                                   |
| `crates/macros`    | `schematic_macros` | Thin proc-macro shell over `core`. Exports `Config`, `ConfigEnum`, and `Schematic`.                             |
| `crates/test-app`  | `test_app`         | Manual smoke-test binary.                                                                                       |

### Migration status — read this first

The rewrite has landed. The old hand-written derive was deleted and `crates/macros-next` was renamed
to `crates/macros`, so `schematic_macros` is now a thin shell over `core` and there is only one
derive implementation. Anything describing `macros` as a separate production implementation is
stale.

All three derives declare `serde` as a helper attribute, so `#[serde(untagged)]`, `#[serde(tag)]`,
and friends are legal on a type that derives only `Config`. That is the replacement for the removed
`#[config(serde(...))]`.

Deliberately dropped in the rewrite, and not coming back:

- `#[config(serde(...))]` — write the native `#[serde(...)]` attribute instead.
- `#[variant(value)]` — `#[variant(rename)]` covers it.
- The `tracing` feature. Generated code is no longer wrapped in `#[tracing::instrument]`; the loader
  still is.
- Named-field enum variants panic, as they did in the old derive too.

Known remaining hole: **`#[setting(skip_deserializing_if)]`** parses but emits nothing, because
serde has no such attribute.

`ConfigEnum` has no implementation of its own. It is the same `Container` with
`macro_type = ContainerMacro::ConfigUnitEnum`, which swaps what `ToTokens` renders — the
`ConfigEnum`/`FromStr`/`TryFrom`/`Display` impls plus `impl_schematic_full`. The schema comes out of
the ordinary unit-enum path: `get_tag_format` reports `Unit` for a config enum (even when a
`fallback` variant makes it structurally unnamed), so `impl_schema_type` emits a literal per unit
variant and infers the fallback's inner type as a string, as the old derive did.

**Casing has no default.** The old derive applied `camelCase` to struct fields and `kebab-case` to
enum variants. Core applies nothing — a name is used exactly as written, so Rust's `snake_case` is
what ships unless `rename_all` says otherwise. This was the single largest source of churn when the
rewrite landed, and it is why so many fixtures and snapshots in `crates/schematic/tests` are
snake_case. When `rename_all`
is set, `Field::get_name` and `Variant::get_name` apply it via `format_case`, so schemas,
`settings()`, and validation paths agree with what serde accepts. Env keys deliberately skip the
casing and stay derived from the Rust name (or an explicit `rename`).

**All three derives support generics**, where the old derive supported them only on `Schematic`.
`Container` carries `input.generics`, and every emitted item threads them
through: the partial declaration, its `Default`/`Deserialize`, `PartialConfig`, `Config`, and both
`Schematic` impls. Three things fall out of that:

- No bounds are added for you. `PartialConfig` requires `Clone + Default + DeserializeOwned +
  Schematic + Serialize`, so a generic `Config` needs those on its own type parameters, and a
  generic `Schematic` needs `T: Schematic`. An under-bounded type fails to compile at the impl.
- The partial gets an explicit `#[serde(bound(deserialize = "T: DeserializeOwned"))]`. Without it
  serde infers `T: Deserialize<'de>`, which is ambiguous against the `DeserializeOwned` the where
  clause carries. `#[config(partial(serde(bound(...))))]` suppresses the generated one.
- A generic type's `schema_name` appends each type argument (`Wrapper<String>` becomes
  `WrapperString`, its partial `PartialWrapperString`), because schemas are keyed by name alone and
  every instantiation would otherwise claim the same one.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [moonrepo/schematic](https://github.com/moonrepo/schematic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
