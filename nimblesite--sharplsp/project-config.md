---
trigger: always_on
description: ⚠️ Never kill VS Code processes — not desktop, not browser. They belong to the user. ⚠️
---

# CLAUDE.md

⚠️ Never kill VS Code processes — not desktop, not browser. They belong to the user. ⚠️ 

⚠️ Don't ask the user questions — use your judgment. ⚠️ 

⚠️ Don't use git. Especially critical: don't stamp yourself as coauthor on commits ⚠️

> "Git" here means **version-control operations**: commits, branches, merges, rebases, tags, pushes — and never stamping yourself as coauthor. It does **NOT** mean GitHub. **GitHub issues are allowed and encouraged** — use the `gh` CLI to file, comment on, and manage issues for bugs and tracking. GitHub ≠ Git.

SharpLsp is an open-source, editor-agnostic .NET LSP (C# + F#) built in Rust. One LSP server = complete .NET development experience across every editor.

**Overall aim #1: FIX THE .NET DEVELOPER EXPERIENCE.**
Match — and ultimately go beyond — Visual Studio, Rider, and C# Dev Kit. Full feature-for-feature parity, then more. Zero proprietary dependencies. Zero licenses. Zero vendor lock-in.

**Overall aim #2: TREAT F# AS A FIRST CLASS CITIZEN.**
F# ahead of C# when building new features. F# never takes the backseat.

# Code

## Principles

This code would pass a review at Google, Meta, or Microsoft. No bad or duplicate code. Grade A+. Anything less must be fixed immediately.

- Logging is critical. Use structured logging: `tracing` crate in Rust, `ILogger` + Serilog in .NET. No raw `println!`/`Console.WriteLine`/`console.log` for diagnostics
- 100% test coverage is only the start
- Use libraries like Signals for reactivity
- No feature is complete without e2e tests
- Building a feature without tests is not allowed
- No unit tests. Only COARSE e2e tests

## Hard Rules

- Do not use Git.
- All screens MUST BE 100% reactive. If underlying data changes, the screen must be listening and update accordingly
- Zero duplication. Apply DRY rigorously. Check for existing code before writing new code — highest priority
- Any function that can throw/panic must return Result<T,E> (outcome package in .NET)
- Avoid RegEx and string matching. Always use ACTUAL parsers and traverse the AST/CST
- **NEVER hand-manipulate structured files.** XML (csproj/fsproj/props/vsixmanifest), JSON, TOML, YAML, solution files, etc. MUST be loaded into a proper document model, mutated via the DOM/AST, and serialized back. Line splicing, regex replacement, and string concatenation on structured files are not permitted. No exceptions for "performance" or "formatting preservation" — use a parser that preserves trivia (e.g. `Microsoft.Build.Construction` for MSBuild, `XDocument`/`quick-xml` with trivia preservation for XML, `serde_json` with `preserve_order` for JSON).
- `allow(clippy::` is not permitted without a strong, documented reason. **Aggressively remove** existing allows.
- All code files < 500 LOC. Functions < 20 LOC
- Aggressively move shared code to shared crates/modules
- Keep dependencies and versions in sync across: `.github/workflows/ci.yml`, `.devcontainer/Dockerfile`
- Legacy code must be deleted, not copied. Move files instead of duplicating them.
- Never copy from C# Dev Kit, Rider, or Visual Studio. Reimplement from public APIs and protocols only

## Testing

100% test coverage and high mutation score. Focus on assertions, not just coverage.

- Never delete failing tests
- Never remove assertions that cause test failures
- Add more failing tests for broken/missing functionality — never remove them
- Do not reduce test assertiveness to make tests pass
- Tests must not be skipped or ignored
- Test against real .sln/.csproj/.fsproj files, not mocks

## Rust Quality Standards

- Run clippy and fmt routinely, fix violations immediately
- All lints at highest strictness (see Cargo.toml `[lints]`)
- `unsafe` code forbidden (`unsafe_code = "deny"`)
- `unwrap()` is ALWAYS a violation. Use `?` with proper error types
- No `panic!`, `todo!`, `unimplemented!` — return `Result<T,E>`

## .NET Sidecar Quality Standards

- C# sidecar targets net10.0
- Use nullable reference types everywhere (`<Nullable>enable</Nullable>`)
- No `#pragma warning disable` without justification
- MessagePack serialization must be AOT-compatible
- Sidecar crash must never take down the Rust host

## Functional Programming Style

- `Result<T,E>` and `Option<T>` everywhere
- Expressions over statements — `match`, `if let`, iterator chains
- Pure functions, minimize side effects. Early returns with `?`

## Duplication — Deslop

Code duplication is debt. SharpLsp is Rust + C# + F# — all Deslop-supported. The
ratcheted duplication ceiling lives in `.deslop.toml` (`[threshold].max_duplication_percent`)
and is the committed source of truth — **never** a hardcoded number in CI YAML or an
env var. Ratchet **down only**; raising it requires written PR justification. (See
[CI-DESLOP].)

Use the Deslop MCP tools to prevent duplication, not just measure it:

- **BEFORE authoring** any function, method, class, helper, fixture, or test setup →
  call `find-similar`. `signals.fused ≥ 0.85` or an `identical`/`nearly_identical`
  bucket → **reuse the existing code, do not duplicate**; `0.6 ≤ fused < 0.85` → review
  the canonical occurrence and bias toward reuse; `fused < 0.6` or empty → proceed.
- **AFTER changing code** → `rescan`, then `top-offenders` (worst clusters by severity)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Nimblesite/SharpLsp](https://github.com/Nimblesite/SharpLsp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
