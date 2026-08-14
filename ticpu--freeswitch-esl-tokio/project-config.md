---
trigger: always_on
description: validates all user-supplied fields and rejects `\n`/`\r`. See
---

## Project Type

This is a **library-first** crate. There is an examples/ folder buildable binaries
`Cargo.lock` is gitignored per Cargo convention for libraries.

## Reference Documentation

@docs/

Before changing an area, read the doc covering it — the whole file, not the
section that looks relevant. These carry wire-format and FreeSWITCH behaviour
detail that is in neither the source nor `design-rationale.md`, which records
only the decisions and deliberately leaves the mechanics here. Citing a line
range out of one you have half-read is how a reference goes stale.

## `freeswitch-types` Version Requirement

While the workspace is on a beta, the root `Cargo.toml` requires the full
three-component prerelease — a caret only resolves prereleases when it carries
one itself, and one that does floats to later betas of that version and to any
later stable in the major. Move the floor to the version being released each
time; never `=`-pin the exact beta.

## Enum Variant Ordering — Append Only

New variants on public enums without `#[repr(...)]` **must be appended at
the end**. Inserting in the middle shifts implicit discriminant values,
which `cargo semver-checks` flags as a breaking change (callers using
`as isize` casts see different values). Grouping by category is fine
within a single commit that introduces the enum, but subsequent additions
always go at the tail.

## `#[non_exhaustive]` Policy

All public enums and public structs **with public fields** have
`#[non_exhaustive]`. Structs with all-private fields do **not** need
`#[non_exhaustive]` — privacy already prevents external construction and
destructuring, and omitting the attribute lets internal code benefit from
exhaustive compiler checks.

**Public fields split:** Types with invariants or builder APIs
(`ExecuteOptions`, `EslConnectOptions`, `Application`, `BridgeDialString`)
use private fields + accessor methods. Pure data/DTO structs with
`#[non_exhaustive]` and constructors (`SofiaGateway`, `SofiaEndpoint`,
`SofiaContact`, `LoopbackEndpoint`, `UserEndpoint`, `UriInfoEntry`,
`ConferenceInfo` and its children) keep public fields — no validation
constraints on individual fields.

**Exception:** single-field error newtypes (`pub struct ParseFooError(pub String)`)
are exempt. These will never grow additional fields, and adding `#[non_exhaustive]`
would break destructuring (`let ParseFooError(msg) = err`) for zero practical
semver benefit.

Because `#[non_exhaustive]` prevents struct literal construction from external
crates (including `examples/`), such a struct needs a constructor (`new()` or
named constructors) **only when something outside the crate has to build one** —
a test fixture, an example, or a downstream app with a legitimate reason to
construct the value. Types that only ever come out of a parser or a wire decode
need none; add one when a caller actually asks. Optional fields use builder
methods (`with_foo()`).

## SIP Modules Are Protocol-Agnostic

Modules under the `sip_*` namespace (`sip_header`, `sip_header_addr`,
`UriInfo`) are pure SIP standard types with no FreeSWITCH coupling.
Doc comments, module-level docs, and error messages in these modules must
not reference FreeSWITCH, mod_sofia, ESL, NOTIFY_IN, or any FS-specific
concepts. FreeSWITCH integration context belongs in `lookup.rs` (the
`HeaderLookup` trait methods) or `variables/` (e.g. `SipPassthroughHeader`
for `sip_h_*`/`sip_i_*` mappings).

## API Boundary Rules

- **Never expose dependency types in public signatures.** Return `impl Iterator`
  (not `indexmap::map::Iter`), wrap dependency errors (not `#[from] serde_json::Error`).
  A dependency major-version bump becomes a semver break if its types leak.
  **Exceptions:** `sip-uri` is an accepted public dependency of `freeswitch-types`
  (same author, narrow scope, stable). The `pub use sip_uri;` re-export and
  `SipHeaderAddr` returning `sip_uri::Uri` are intentional. `indexmap` is treated
  as a basic collection type (like `HashMap`) and may appear in public signatures.
- **`pub(crate)` modules can still leak types.** If a public function returns a
  type from a `pub(crate)` module, that type is visible but unnameable by callers.
  Either re-export the type or don't return it.
- **Struct fields that control behavior should be private.** Expose via accessor
  methods (e.g. `scope()` not `pub vars_type`). This prevents callers from
  mutating invariants after construction.
- **`constants` module is `pub(crate)`.** Only `DEFAULT_ESL_PORT` is re-exported.
  Internal protocol constants are implementation details.

## Method Signature Conventions

- **`FromStr` casing rules.** Wire protocol types use **strict canonical case**.
  User-facing config types use `eq_ignore_ascii_case`. `Display` always emits
  the canonical form.
- **Typed + raw method pairs.** `filter()` / `filter_raw()`,
  `subscribe_events()` / `subscribe_events_raw()`. Always provide the `_raw`
  escape hatch for values not yet in the enum.
- **Options structs for optional wire headers.** Keep the base method simple;
  add `_with_options()` variant. Never grow parameter lists.
- **Preserve wire context in error/status enums.** Disconnect notices, auth
  responses carry useful context — never discard it.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ticpu/freeswitch-esl-tokio](https://github.com/ticpu/freeswitch-esl-tokio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
