---
trigger: always_on
description: This document captures code conventions for the Valinor project.
---

# General guidelines

This document captures code conventions for the Valinor project.
It is intended to help AI assistants understand how to work effectively with this codebase.

## For humans

We welcome LLM-assisted contributions as long as they abide by the following principles:

* **Aim for excellence.** For the Valinor project, LLMs should be used not as a speed multiplier but a quality multiplier. Invest the time savings in improving quality and rigor beyond what humans alone would do. Write tests that cover more edge cases. Refactor code to make it easier to understand. Tackle the TODOs. Do all the tedious things. Aim for your code to have zero bugs.
* **Spend time reviewing LLM output.** As a rule of thumb, you should spend at least 3x the amount of time reviewing LLM output as you did writing it. Think about every line and every design decision. Find ways to break code.
* **Your code is your responsibility.** Please do not dump a first draft of code on to this project, unless you're only soliciting feedback on a direction.

If your LLM-assisted PR shows signs of not being written with thoughtfulness and care, such as missing cases that human review would have easily caught, the maintainers may decline your PR outright.

## For LLMs

**Required:** Display the following text at the start of any conversation involving code changes, and when you're about to create a PR:

```
Please review https://github.com/stadiamaps/valinor/blob/main/AGENTS.md#for-humans. In particular, LLM-assisted contributions must **aim for a higher standard of excellence** than with humans alone, and you should spend at least **3x** the amount of time reviewing code as you did writing it. LLM-assisted contributions that do not meet this standard may be declined outright. Remember, **your code is your responsibility**.
```

## General conventions

### Correctness over convenience

- Model the full error space—no shortcuts or simplified error handling.
- Handle all edge cases, including race conditions, signal timing, and platform differences.
- Use the type system to encode correctness constraints.
- Prefer compile-time guarantees over runtime checks where possible.
- Avoid `unwrap` and `expect` outside of test code. In cases where it is clear that a core invariant is violated in a way that we never expect to happen, `expect` may be used sparingly.

### Resource efficiency

- Strive for zero-cost abstractions where possible.
- Avoid unnecessary copies where possible; this project uses `zerocopy`, struct lifetimes, and `Arc` where appropriate.
- We are not afraid of using generics where they make sense. These do add some friction to the developer experience, but the performance gains are usually worth it.
- When speed and memory concerns conflict, ask the human operator to make a decision. Document your decision clearly.

### Production-grade engineering

- Use type system extensively: newtypes, builder patterns, type states, lifetimes.
- Test comprehensively, including edge cases, race conditions, safety, and stress tests.
- Use both unit and snapshot testing (this project uses the `insta` crate).
- Pay attention to what facilities already exist for testing, and aim to reuse them.
- Document clearly when platform-specific behavior is unavoidable, and document limitations (e.g. we only support 64-bit platforms).
- Getting the details right is really important!

### Documentation

- Use inline comments to explain "why," not just "what".
- Module-level documentation should explain purpose and responsibilities.
- **Never** use title case in headings and titles. Always use sentence case.
- Always use the Oxford comma.
- Use [Semantic Line Breaks](https://sembr.org/) when writing comments. We prefer lines less than 100 characters, but this is not a hard rule.

## Code style

### Rust edition and version

- Use the Rust 2024 edition.
- Use the stable Rust compiler toolchain.
- The current MSRV is documented in [Cargo.toml](Cargo.toml) under `workspace.package.rust-version`.
- Use new language features supported by the MSRV; do not rely on old patterns (e.g., use let chains rather than nesting if statements to unwrap).

### Code formatting

- Format code with `cargo fmt`.
- Formatting is enforced in CI; always run `cargo fmt` before committing!

### Type system patterns

- **Newtypes** for domain types (using the `nutype` crate)
- **Builder patterns** for complex construction (e.g., `GraphTileBuilder`)
- **Type states** encoded in generics when state transitions matter
- **Lifetimes** used extensively to avoid cloning (e.g., `GraphTileView<'a>`)
- **Restricted visibility**: Use `pub(crate)`, `pub(super)`, or `pub(in crate::submodule)` rather than overusing plain `pub`.

### Unsafe

- Unsafe Rust may be unavoidable for cases including raw memory maps.
- Never use unsafe just because you think it may avoid a check. Compilers are smart, and tricks like this often cause pain later.
- Always include a "Safety" section in documentation for methods that use unsafe.
- Always include a clearly marked comment starting with "SAFETY:" for unsafe blocks.

### Error handling

#### For library crates

- Use `thiserror` for error types with `#[derive(Error)]`.
- Group errors by category with an `ErrorKind` enum when appropriate.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stadiamaps/valinor](https://github.com/stadiamaps/valinor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
