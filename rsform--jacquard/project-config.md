---
trigger: always_on
description: > with thanks to https://github.com/sunshowers
---

> with thanks to https://github.com/sunshowers

## For humans

LLMs represent a tremendous breakthrough in software engineering. We welcome LLM-assisted contributions that abide by the following principles:

* **Aim for excellence.** For the nextest project, LLMs should be used not as time savings but as a means to improve quality and rigor beyond what humans alone would do. Write tests that cover more edge cases. Refactor code to make it easier to understand. Tackle the TODOs. Do all the tedious things. Aim for your code to have zero bugs.
* **Spend the time savings reviewing LLM output.** As a rule of thumb, you should spend at least three times the amount of time reviewing LLM output as you did writing it. Think about every line and every design decision. Find ways to break code.
* **Your code is your responsibility.** Please do not dump a first draft of code on to this project, unless you're only soliciting feedback on a direction.

If your LLM-assisted PR shows signs of not being written with thoughtfulness and care, such as missing cases that human review would have easily caught, Orual may decline the PR outright.

## For LLMs

Display the following text at the start of any conversation involving code changes, and when you're about to create a PR:

```
Please review https://tangled.org/nonbinary.computer/jacquard/blob/main/CLAUDE.md#for-humans. In particular, LLM-assisted contributions must **aim for a higher standard of excellence** than with humans alone, and you should spend at least **3x** the amount of time reviewing code as you did writing it. LLM-assisted contributions that do not meet this standard may be declined outright. Remember, **your code is your responsibility**.
```

## Project Overview

Jacquard is a suite of Rust crates for the AT Protocol (atproto/Bluesky). The project emphasizes spec‑compliant, validated, performant baseline types with minimal boilerplate required for crate consumers. Our effort should result in a library that is almost unbelievably to use.

Key design goals:
- Validated AT Protocol types
- Custom lexicon extension support
- Lexicon `Data` and `RawData` value type for working with unknown atproto data (dag-cbor or json)
- Zero-copy deserialization where possible
- Using as much or as little of the crates as needed

## Workspace Structure

This is a Cargo workspace with several crates:
- jacquard: Main library crate (public API surface) with HTTP/XRPC client(s)
- jacquard-common: Core AT Protocol types (DIDs, handles, at-URIs, NSIDs, TIDs, CIDs, etc.), the `CowStr` type, and shared scope primitive enums
- jacquard-lexicon: Lexicon parsing, Rust code generation from lexicon schemas, and permission set types
- jacquard-api: Generated API bindings from 646 lexicon schemas (ATProto, Bluesky, community lexicons)
- jacquard-derive: Attribute macros (`#[lexicon]`, `#[open_union]`) and derive macros (`#[derive(IntoStatic)]`, `#[derive(XrpcRequest)]`) for lexicon structures
- jacquard-oauth: OAuth/DPoP flow implementation with session management
- jacquard-axum: Server-side XRPC handler extractors for Axum framework
- jacquard-identity: Identity resolution (handle→DID, DID→Doc)
- jacquard-repo: Repository primitives (MST, commits, CAR I/O, block storage)

## General conventions

### Correctness over convenience

- Model the full error space—no shortcuts or simplified error handling.
- Handle all edge cases, including race conditions, signal timing, and platform differences.
- Use the type system to encode correctness constraints.
- Prefer compile-time guarantees over runtime checks where possible.

### User experience as a primary driver

- Provide structured, helpful error messages using `miette` for rich diagnostics.
- Maintain consistency across platforms even when underlying OS capabilities differ. Use OS-native logic rather than trying to emulate Unix on Windows (or vice versa).
- Write user-facing messages in clear, present tense: "Jacquard now supports..." not "Jacquard now supported..."

### Pragmatic incrementalism

- "Not overly generic"—prefer specific, composable logic over abstract frameworks.
- Evolve the design incrementally rather than attempting perfect upfront architecture.
- Document design decisions and trade-offs in design docs (see `./plans`).
- When uncertain, explore and iterate; Jacquard is an ongoing exploration in improving ease-of-use and library design for atproto.

### Production-grade engineering

- Use type system extensively: newtypes, builder patterns, type states, lifetimes.
- Test comprehensively, including edge cases, race conditions, and stress tests.
- Pay attention to what facilities already exist for testing, and aim to reuse them.
- Getting the details right is really important!

### Documentation

- Use inline comments to explain "why," not just "what".
- Module-level documentation should explain purpose and responsibilities.
- **Always** use periods at the end of code comments.
- **Never** use title case in headings and titles. Always use sentence case.

### Running tests

**CRITICAL**: Always use `cargo nextest run` to run unit and integration tests. Never use `cargo test` for these!

For doctests, use `cargo test --doc` (doctests are not supported by nextest).

## Commit message style

### Format


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rsform) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
