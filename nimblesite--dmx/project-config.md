---
trigger: always_on
description: <!-- agent-pmo:a72c926 -->
---

<!-- agent-pmo:a72c926 -->
# dmx — Agent Instructions

⚠️ **TOKEN DISCIPLINE.** Check file size first. `Grep` over `Read`. Use `offset`/`limit`. Smallest diff that solves the problem. Delete dead code, unused imports, stale comments. Call out irrelevant context before proceeding. Bloat degrades reasoning. ⚠️

⚠️ **ACT AUTONOMOUSLY. DO NOT STOP TO ASK QUESTIONS.** When something is ambiguous, pick the most reasonable default, note the assumption, and work to completion. Deliver finished work plus a short list of assumptions. ⚠️

## What dmx Is

**Fast, reliable Dart code generation on every save. No generated `part` files. Fully customizable.**

Annotate a Dart class, save, and the generated members appear **inside the same file**, below a divider — no `part` directive, no `.g.dart`, no mixin, no delegating factory. The VS Code extension runs the watcher; other editors run `dmx watch` once. Built-ins cover the familiar Freezed / dart_mappable jobs; Mustache templates let teams reshape the output; custom Dart macros generate members or whole files from any source of truth.

Pipeline: **parse → context → render → validate → emit inline.** tree-sitter reads the Dart CST, a Rust context builder finishes every decode/encode/equals/hash/copy expression, Mustache renders dumb strings, the emitter splices between the dividers byte-exactly.

**Languages:** Rust (the CLI). Dart is the *output* plus the `src/dart_packages/dmx` runtime.

## Messaging — [docs/messaging.md](docs/messaging.md) is the ONLY authority

Any user-facing words — README, website, docs, blog, release notes, CLI help, extension copy, PR descriptions — must match [docs/messaging.md](docs/messaging.md). Read it before writing prose.

- **Never restate the pitch from memory.** Reuse its ready-to-use copy verbatim where one fits.
- **Obey its Guardrails section literally.** No numeric speed claims without a reproducible benchmark; no "drop-in Freezed replacement"; no compiler jargon in the lead.
- **Lead with the workflow:** *save and keep coding*, *no `build_runner` dance*, *no generated `part` files*, *one complete Dart file*.
- Changing the message means editing `docs/messaging.md` in the same commit as the copy quoting it.

# HARD RULES — ALL LANGUAGES, NO EXCEPTIONS

- ZERO DUPLICATION. Run Deslop often.
- ZERO DEAD CODE. Check often.
- THROWING EXCEPTIONS IS ⛔️ ILLEGAL.
- FP STYLE ALL THE WAY: `Result<T,E>`, immutable.
- Pattern matching only. NO CASTING.
- FILES OVER 500 LOC ARE ILLEGAL.
- NO REGEX on structured data — real parsers for JSON/YAML/TOML/Dart.
- NO PLACEHOLDERS that silently no-op. No linter suppressions — fix the code.
- **GENERATED CODE OBEYS EVERY RULE ABOVE.** No `throw`, no `as`, no `!`.
- **PROPER NAMES** in generated code: `json`, `other`, `value` — never hash-mangled. Suffix only on a real collision.
- **TYPEDIAGRAM MACRO** [typediagram]: the built-in `typeDiagram` macro parses Markdown and the typeDiagram DSL natively in Rust. typeDiagram definitions are the model source; bound Mustache templates are the sole authority for generated code shape. Never invoke typeDiagram's language emitters in the production path.

## Hard Rules — Rust

- No `unwrap()`/`expect()` in production (tests may `expect`). No `panic!`/`todo!`/`unimplemented!`/`unreachable!`.
- No `unsafe {}` or `allow(clippy::...)` without documented justification.
- Every public item has `///` docs; every module opens with `//!` citing the spec ID it implements.
- `thiserror` for library errors; `anyhow` only in the CLI binary.
- Lints live in `Cargo.toml` `[lints.rust]` / `[lints.clippy]` — every rule on, at ERROR. [LINT-RUST]

## Hard Rules — Dart (emitted code and `src/dart_packages/dmx`)

- No `late`, `!`, `dynamic`, `as` casts (use `is` + smart casts), `.then()` (use `async`/`await`), or `throw`.
- Generated Dart must satisfy `dart analyze --fatal-infos` **and** every rule above.
- `examples/storefront/lib` and `src/dmx/tests/golden/*.dart` are generator OUTPUT. Never hand-edit between the dividers — change the template or context builder and regenerate (`make golden`, `make example`).

## Logging

- `tracing` + `tracing-subscriber` only. Never `println!`/`eprintln!`/`dbg!` for internal state. The CLI's stdout/stderr (progress lines, `DMX####` diagnostics) is the **output contract**, not logging — leave it alone; CLI tests assert on it.
- Log entry/exit of significant operations (`error|warn|info|debug|trace`). Silent failures are forbidden.
- Structured fields, never interpolation: `{ path: "…", generation: 4 }`.
- NEVER log PII or secrets. Log `"key: present"` or a truncated hash.

# Bugs / Correctness

When fixing a bug, you MUST write a failing test that fails because of the bug before moving forward. Follow this [text](.agents/skills/fix-bug/SKILL.md)

Placeholder code is always illegal. If you encounter code that produces incorrect results or is not complete:

- Immediately DELETE the offending code
- Replace it with a panic or exception
- Report the problem to the user explaining why the code is wrong
- 🛑 STOP 

# Testing

- LOAD THIS REPO UP WITH E2E TESTS. NO MOCKS. Cover the corner cases.
- MANY user interactions per test; MANY assertions per user interaction; MANY input permutations per test

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Nimblesite/dmx](https://github.com/Nimblesite/dmx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
