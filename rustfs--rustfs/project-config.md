---
trigger: always_on
description: This root file keeps repository-wide rules only.
---

# RustFS Agent Instructions (Global)

This root file keeps repository-wide rules only.
Use the nearest subdirectory `AGENTS.md` for path-specific guidance.

## Rule Precedence

1. System/developer instructions.
2. Current user/task instructions.
3. The nearest `AGENTS.md` in the current path.
4. This file (global defaults).

If repo-level instructions conflict, follow the nearest file and keep behavior aligned with CI.

## Execution Discipline

- Read the relevant existing code, tests, and local guidance before changing behavior. For new helpers or test setup, that read includes `crates/utils`, `crates/common`, and the touched crate's own `test_util`/fixtures (see Reuse Before You Write).
- State assumptions when they affect the implementation or verification path.
- If a task has multiple plausible interpretations, list the options briefly and choose the narrowest reasonable path; ask when the ambiguity would make the change risky.
- For multi-step work, keep the plan minimal and tied to verifiable outcomes.
- Avoid redundant file reads, repeated commands, and unnecessary exploratory work once enough context is available.
- A good result is a minimal diff with clear assumptions, no over-engineering, and independent verification that survives Adversarial Validation (below).

## Communication and Language

- Respond in the same language used by the requester.
- Keep source code, comments, commit messages, and PR title/body in English.
- Be concise. Avoid sycophantic openers, closing fluff, and verbose status reporting.

## Change Style for Existing Logic

- Prefer direct, local code over extracting one-off helpers.
- Extract a helper only when logic is reused or the extraction materially clarifies a non-trivial flow.
- Use Rust's default module file layout (`mod foo;` with `foo.rs` or `foo/mod.rs`/`foo/*.rs`).
  Avoid `#[path = "..."]` for module inclusion; move files into the canonical module tree instead.
  If an unavoidable generated-code, FFI, or test-fixture exception remains, keep it local and document why the canonical layout cannot work.
- Solve only the requested problem; do not add speculative features, configurability, or adjacent improvements.
- Prefer editing existing code over rewriting files or reshaping unrelated logic.
- Modify only what is required and remove only artifacts introduced by your own changes.
- Preserve the existing control-flow and logic shape when fixing bugs or addressing review comments, especially in init, distributed coordination, locking, metadata, and concurrency paths.
- Do not refactor existing code only to make it easier to unit test.
- Keep fixes narrowly aligned with the requested behavior; avoid semantic-adjacent rewrites while touching sensitive paths.
- Keep code elegant, concise, and direct. Prefer minimal, readable implementations over over-engineering and excessive abstraction. Use comments to clarify non-obvious intent and invariants, not to compensate for unclear code.
- Do not write comments that narrate what the next line does, restate a signature, or describe the change you just made — that commentary belongs in the PR description, not the code. Required invariant comments — lock ordering, `SAFETY`, unwrap justification, `#[allow(dead_code)]` rationale, `RUSTFS_COMPAT_TODO` — are never narration.
- Mention unrelated issues when useful, but do not fix them as part of a narrow task.

## Reuse Before You Write

Search for an existing implementation before writing a new one; extend what exists instead of duplicating it:

- **Helpers and utilities** (path/string handling, hashing, retry, env parsing, IO wrappers): check `ls crates/utils/src` first — file names map to operations (`retry.rs`, `envs.rs`, `hash.rs`, `path.rs`, `string.rs`, `io.rs`) — plus `crates/common` (shared structures/globals), then `rg -i 'fn \w*<term>' crates/utils/src crates/common/src <touched-crate>/src` for signatures. Helpers are snake_case: a full-text single-word grep over a large crate drowns you and a multi-word phrase returns nothing. Reimplementing an existing workspace helper — or hand-rolling what `std`, `tokio`, or an existing workspace dependency already provides — is a review finding, not a style preference.
- **Reuse requires matching semantics, not a matching name**: before adopting a helper, check its normalization (`clean` resolves `.`/`..` — never apply it to raw S3 object keys), error type, backoff/deadline behavior, and durability gating against the call site. When semantics differ, a new narrowly-named helper with a comment naming the rejected lookalike is the correct outcome. The inverse also holds: workspace wrappers exist because raw `std`/`tokio` semantics were insufficient (durability gates, retries) — prefer the wrapper over the raw call.
- **Constants and fixed tokens** (protocol labels, error identifiers, header keys, event names, metric names, command tags): search for existing constants/enums that already represent the same semantic value and reuse them. If a value is truly new, define one local constant near related logic; never scatter the literal across sites. When changing existing behavior, align naming and format with the established constants.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rustfs/rustfs](https://github.com/rustfs/rustfs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
