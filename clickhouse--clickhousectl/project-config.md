---
trigger: always_on
description: Read with the root `AGENTS.md` (commands, workspace rules, CI gates). This file covers the published API library
---

# AGENTS.md — `clickhouse-cloud-api` and `clickhouse-openapi-analyzer`

Read with the root `AGENTS.md` (commands, workspace rules, CI gates). This file covers the published API library
and the private drift analyzer, which are always edited together.

## Layout

- `src/client.rs` — `Client` and shared HTTP machinery; endpoint methods live in private per-domain `src/client/*.rs`.
- `src/models.rs` — the public model facade and the shared `discriminated_union!` macro (its rustdoc owns the
  grammar). Request/response structs, enums, aliases and impls live in private per-domain `src/models/*.rs` and
  are re-exported without changing the crate-root or `models::*` paths.
- `src/convert.rs` — `MissingRequiredFields` and conversion documentation; explicit response→request conversions
  live in private per-domain `src/convert/*.rs`.
- `src/error.rs` — `Error` is the structural contract for failure modes: a failure a caller must tell apart gets its
  own variant, never a recognizable message. `Error::Sql` (the Query API rejecting a statement) exists precisely
  so callers stop sniffing a `SQL error ` prefix. Keep each variant's `Display` stable — it is what the user sees.
- `crates/clickhouse-openapi-analyzer/` — OpenAPI and Rust inventory, direction-aware comparison, policy config, and
  stable drift reports. Private (`publish = false`), a dev-dependency of this crate. Parser/tooling deps such as
  `syn` must not enter either published crate's normal dependency graph. It recursively traverses the private module
  trees rooted at `client.rs`, `models.rs`, `meta.rs`; model declarations must remain literal source in that tree,
  and declarations in conversion files do not count as models.

## Request and response models

A response must never fail to deserialize because the API dropped a field, sent it as `null`, or added one —
several teams evolve the Cloud API independently, so in a published crate every strict response field is a latent
outage. Tolerance lives in the **type system**, not in serde attributes:

- **Request types are strict.** A spec-required field is `T`; optional or nullable fields are `Option<T>` plus
  `#[serde(skip_serializing_if = "Option::is_none")]`. The compiler enforces "strict in what we send".
- **Response types are all-`Option`.** Every field of every type reachable from a `Client` return type is `Option<T>`
  plus `skip_serializing_if`; a missing key *and* an explicit `null` both land as `None` natively. Nothing is
  fabricated, so "the server sent `0`" and "the server dropped the field" stay distinguishable.
- **`#[serde(default)]` is banned in the model module tree.** On a required request field it invents `""`/`0`/
  `false` that a get → edit → write-back caller silently persists; on an `Option` field it is dead weight.
  Sweeping it across every field was the superseded policy of #312/#313 — do not reintroduce it.
- **Unknown fields are ignored.** Never `deny_unknown_fields`.
- **Typed `additionalProperties` maps preserve dynamic keys.** Use typed string-keyed map aliases instead of
  empty structs; keep parent response fields optional. The analyzer checks named map schemas in both directions.

### Naming and the split

- A schema used in one direction keeps its Rust name — most schemas are one-directional, so most models are
  simply all-`Option` in place (response) or strict in place (request body, orphan schema).
- A schema used in **both** directions becomes two types: the request variant keeps the schema's name; the
  response variant is exactly `{Name}Response`, matching spec-derived names like `ApiKeyPostResponse`. The
  analyzer resolves the pair from that convention, so the suffix is load-bearing, not cosmetic.
- Splitting propagates: a nested type reachable from both a strict and a tolerant parent splits too, and a field
  of a response type must point at the `*Response` variant of anything that has one. Re-point the element type of
  any shared alias (`pub type PgTagsResponse = Vec<ResourceTagsV1Response>`).
- Object unions follow the variant structs they hold. When those split, duplicate the whole
  `discriminated_union!` invocation for a `{Name}Response` enum over the `*Response` structs, keeping the
  discriminator and `none unless` guards identical. The macro emits only `Deserialize`, so the enum declaration,
  derives, `Display` and any `Default` stay literal source for the syn analyzer to inventory.
- Enums and unions are otherwise **unchanged** by the split: string enums keep `Unknown(String)`, unions keep the
  lossless `Unknown(serde_json::Value)` fallback, and dispatch reads raw JSON rather than struct fields, so
  all-`Option` variants cannot weaken it.

### Serialization and write-back

- Response types keep `derive(Serialize)` — `--json` and `print_human` serialize them directly.
  `skip_serializing_if` on every response field means an absent field is **omitted**, never `null`. A test pins it.
- A caller that fetches, edits and writes back must resolve absence explicitly. The owning `src/convert/<domain>.rs`
  holds those conversions: `TryFrom<{Name}Response> for {Name}` where a required request field can be absent,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ClickHouse/clickhousectl](https://github.com/ClickHouse/clickhousectl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
