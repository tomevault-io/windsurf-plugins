---
trigger: always_on
description: `oxvif` is an async Rust client library for the ONVIF IP camera protocol.
---

# oxvif — Development Guidelines

## Project overview

`oxvif` is an async Rust client library for the ONVIF IP camera protocol.
Library crate (no binary). Published on crates.io.

## Before every commit

```
cargo fmt
cargo clippy --all-targets -- -D warnings
cargo test
```

All three must pass cleanly before committing.

## Before every publish (additional checks)

```
cargo test --doc          # verify all doc examples compile and run
cargo doc --no-deps       # verify HTML docs generate cleanly (mirrors docs.rs)
cargo audit               # zero vulnerabilities required
cargo outdated --depth 1  # review; upgrade direct deps if significantly behind
```

After `cargo outdated`, if any direct dependency was updated, re-check for
feature-unification footguns (a public API a sibling crate can flip off via
`#[cfg(not(feature = …))]`). See [docs/dependency-pitfalls.md](docs/dependency-pitfalls.md)
for the audit steps and the `quick-xml/encoding` case that motivated this.

## Coding rules

### Required fields must return `Result`

Every `from_xml` / `vec_from_xml` function that parses a required field
(especially `token` attributes) must return `Err` on missing input — never
silently default to an empty string.

```rust
// WRONG
token: node.attr("token").unwrap_or("").to_string()

// CORRECT
let token = node
    .attr("token")
    .filter(|t| !t.is_empty())
    .ok_or_else(|| SoapError::missing("Foo/@token"))?
    .to_string();
```

### XML escaping

All user-supplied strings or device-echoed strings interpolated into XML
bodies must be wrapped in `xml_escape()` (defined in `src/types/mod.rs`).

```rust
// WRONG
format!("<tt:Name>{name}</tt:Name>")

// CORRECT
format!("<tt:Name>{}</tt:Name>", xml_escape(name))
```

This applies to:
- `format!()` calls in `client.rs` that embed `&str` parameters
- `to_xml_body()` methods in `src/types/*.rs`

### No `unwrap()` in library code

Library code must not panic on malformed device responses.
Use `?`, `if let`, or `.ok_or_else()` instead of `.unwrap()`.

Test code may use `.unwrap()` / `.expect()` where appropriate.

### No panics in `vec_from_xml` closures

When using `.map(|node| ...)` to parse a collection, the closure must return
`Result<T, OnvifError>` and the final `.collect()` will propagate the first
error. Do not use `Ok(iter.map(|n| Self { ... }).collect())` when any field
can fail.

```rust
// WRONG — silently skips errors
Ok(resp.children_named("Foo").map(|n| Self { ... }).collect())

// CORRECT — propagates first error
resp.children_named("Foo").map(|n| {
    let token = ...?;
    Ok(Self { token, ... })
}).collect()
```

## Testing rules

- Every new client method needs at least one **positive test** (happy path)
  and one **negative test** (missing required field or SOAP Fault).
- Fixtures go in `src/tests/client_tests.rs`.
- Use `MockTransport` for happy-path tests and `ErrorTransport` for HTTP
  error tests.
- Negative SOAP Fault tests: use `make_soap_fault_xml(code, reason)`.

## Adding a new ONVIF service — step-by-step SOP

### Implementation

1. Create `src/types/<service>.rs` with all response structs.
   - All `from_xml` / `vec_from_xml` that parse required fields → `Result<Self, OnvifError>`
   - Token attributes → `.ok_or_else(|| SoapError::missing("Elem/@token"))?`
   - `to_xml_body()` string fields → `xml_escape(&self.field)`
2. Add `mod <service>;` and `pub use <service>::*;` to `src/types/mod.rs`.
3. Add methods to `src/client.rs`:
   - Add new types to the `use crate::types::{ ... }` import list
   - All `&str` params interpolated into XML → `xml_escape(param)`
4. Re-export all new public types from `src/lib.rs`.

### Testing

5. Append tests to `src/tests/client_tests.rs`:
   - At least one positive test per method (fixture XML + assert fields)
   - At least one negative test per method (missing token or SOAP Fault)
   - For write methods: use `RecordingTransport` and assert `c.action` + `c.body`

### Mock server coverage

5a. Add a handler for every new ONVIF action under `src/mock/` (the mock
    engine moved into the library in 0.9.6; `examples/mock_server/` is now
    a thin wrapper over `oxvif::mock::MockServer`). Including write/Set
    methods. This keeps both `MockTransport` and `MockServer` as full
    integration harnesses that run without a real device.
    - Add the action URI to the match block in `src/mock/dispatch.rs`.
    - Add a `resp_<operation>()` function in the right
      `src/mock/services/<service>.rs` returning a plausible response
      (or mutating `DeviceState` for write methods).
    - Write methods that return `void` may share the empty-body helper
      from `src/mock/helpers.rs`.
    - The behind-the-scenes example binary needs no change — it auto-picks
      up new handlers because they live in the library now.

### Quality gate (run before every commit)

```
cargo fmt
cargo clippy --all-targets -- -D warnings
cargo test
```

All three must pass cleanly.

### Documentation

6. Update `README.md`:
   - Architecture diagram (top of file) if a new service is added
   - Add a new `## <Service> methods` section with method table and code example
   - Update the `Implemented ONVIF operations` status table (— → ✓)
   - Update test count (`N unit tests`)
   - Update installation version number

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [smiti1642/oxvif](https://github.com/smiti1642/oxvif) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-12 -->
