---
trigger: always_on
description: This file is the short contract for code agents generating or migrating
---

# Agent Guide for cbor2

This file is the short contract for code agents generating or migrating
`cbor2` integrations. Prefer these rules over guessing from generic serde or
CBOR examples.

## Cargo Setup

The default build (`cbor2 = "1"`) is `std` and already covers `to_vec`,
`to_writer`, `from_slice`, `from_reader`, `validate`, `validate_slice`,
`Value`, `Simple`, `RawValue`, `cbor!`, the canonical encoders, diagnostics
and the synchronous `async_io` helpers. Add a feature only for the rows that
need one:

| Need | Manifest line |
| --- | --- |
| `#[derive(cbor2::Cbor)]` | `cbor2 = { version = "1", features = ["derive"] }` |
| tokio async adapters | `cbor2 = { version = "1", features = ["tokio"] }` |
| futures async adapters | `cbor2 = { version = "1", features = ["futures"] }` |
| `no_std` + heap | `cbor2 = { version = "1", default-features = false, features = ["alloc"] }` |
| `no_std`, no heap | `cbor2 = { version = "1", default-features = false }` |

Plain serde structs (`#[derive(Serialize, Deserialize)]`, used by most recipes
here) also need `serde = { version = "1", features = ["derive"] }` in the
manifest. Generating code that uses a feature-gated API without enabling the
feature is the most common compile failure — set the manifest first.

## API Selection

| Task | Use | Do not use |
| --- | --- | --- |
| Encode a serde value to memory | `cbor2::to_vec` | Manual `Value` construction unless the shape is dynamic |
| Encode a serde value to a writer | `cbor2::to_writer` | Building a `Vec` first when streaming is enough |
| Decode from an in-memory buffer | `cbor2::from_slice` | `from_reader` if borrowed fields are expected |
| Decode from `Read` | `cbor2::from_reader` | Borrowed output types |
| Require exactly one item in a buffer | `cbor2::validate_slice` before/after decode (`validate` for readers) | Assuming `from_slice` rejects trailing bytes |
| Decode a CBOR sequence | `cbor2::de::Deserializer::into_iter` | Repeated `from_slice` on the same buffer |
| Preserve exact encoded bytes | `cbor2::RawValue` | Decode/re-encode through typed structs |
| Dynamic or unknown shape | `cbor2::Value` or `cbor2::cbor!` | Untyped maps of JSON strings |
| Preserve CBOR simple values | `cbor2::Simple` or `Value::Simple` | Collapsing registered simple values into ad hoc integers |
| Deterministic bytes for signatures | `to_canonical_vec` / `to_canonical_writer` | Plain `to_vec` on maps with unspecified order |
| COSE integer keys, arrays, or tags | `#[derive(cbor2::Cbor)]` (feature `derive`) | `serde(rename = "1")` for integer keys |
| Async read/write of a typed value | `cbor2::async_io::{read_value, write_value}` | Treating serde itself as async |
| Async read from an untrusted stream | `cbor2::async_io::read_value_with_limit` / `read_item_with_limit` | Unbounded helpers without an outer message limit |
| Async item when you must borrow or inspect raw bytes | `cbor2::async_io::read_item` then `from_slice` | `read_value` when the buffer must outlive the call |

## CLI Selection

The `cbor2-cli` crate installs a `cbor` binary. Prefer text-safe commands when
you are an AI/code agent working in a terminal transcript:

| Task | Command |
| --- | --- |
| Inspect pasted CBOR hex/base64 or a file | `cbor <INPUT>` |
| Convert CBOR to JSON for jq-like tools | `cbor decode --json <INPUT>` |
| Preserve wire details while pretty-printing | `cbor decode <INPUT>` or bare `cbor <INPUT>` |
| Convert JSON to copyable CBOR bytes | `echo '{"a":1}' \| cbor encode --json --hex` |
| Convert CDN to copyable CBOR bytes | `printf "{1: h'dead'}" \| cbor encode --cdn --hex` |
| Pipe raw CBOR bytes to another binary command | `cbor encode` |
| Check one or more complete CBOR items | `cbor validate <INPUT>` |

For agent-generated examples, prefer `cbor encode --hex` over raw
`cbor encode`; raw binary stdout is hard to quote, diff and paste reliably.
Use `--json` when the input is intended to be strict JSON, or `--diag`/`--cdn`
when it is intended to be CDN.

## Non-Negotiable Semantics

- `from_slice` and `from_reader` deserialize one leading CBOR item. They are
  not exact-buffer validators. Use `validate_slice` (or `validate` for
  readers) when trailing data must fail.
- `from_slice` is the borrowed path. Definite-length text and byte strings can
  deserialize as borrowed `&str` and `serde_bytes` values from the input.
- `from_reader` copies because it cannot borrow from a generic stream.
- Indefinite-length text and byte strings can decode into owned targets, but
  cannot be borrowed as one contiguous slice.
- Plain `Vec<u8>` and `&[u8]` are serde sequences and encode as CBOR arrays.
  Use `serde_bytes::ByteBuf`, `serde_bytes::Bytes`, or
  `#[serde(with = "serde_bytes")]` for CBOR byte strings.
- `#[derive(cbor2::Cbor)]` generates serde `Serialize` and `Deserialize`.
  Do not also derive serde's `Serialize` or `Deserialize` on the same type.
- `#[cbor(key = 1)]` creates an integer map key. `#[serde(rename = "1")]`
  creates the text key `"1"`.
- `#[cbor(array)]` is for named structs whose CBOR wire shape is a field-order
  array. Do not combine it with per-field `#[cbor(key = ...)]`.
- `#[cbor(tag = N)]` wraps the container in CBOR tag `N` on encode and treats

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ldclabs/cbor2](https://github.com/ldclabs/cbor2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-25 -->
