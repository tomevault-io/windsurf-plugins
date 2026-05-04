---
trigger: always_on
description: This file is law. Every coding agent must read it in full before making any change to this repository. If an action is not clearly allowed here or by an explicit user instruction in the current conversation, stop and ask.
---

# AGENTS.md

This file is law. Every coding agent must read it in full before making any change to this repository. If an action is not clearly allowed here or by an explicit user instruction in the current conversation, stop and ask.

## 1. Project Structure Law

- All compiler passes live in `crates/` as independent workspace members.
- All developer tooling lives in `tools/`.
- Standard library source lives in `stdlib/`.
- Integration and end-to-end tests live in `tests/fixtures/` as `.tg` files.
- No file may be created outside these designated locations without explicit approval.
- Root-level exceptions are governed by section 5 only.

## 2. File Naming Law

- All Rust source files must use `snake_case.rs`.
- All Thagore source files must use `snake_case.tg`.
- No abbreviations are allowed unless they appear in this approved list:
  - `lexer`
  - `typeck`
  - `codegen`
  - `ir`
  - `lsp`
  - `cli`
- Test files must be named `{crate}_tests.rs`.
- Test files must live in `tests/` next to the corresponding `src/` directory.

## 3. Crate Dependency Law

- `lexer` must not depend on any other Thagore crate.
- `parser` may depend only on `ast` and `lexer`.
- `ast` must not depend on any other Thagore crate.
- `typeck` may depend only on `ast` and `lexer`.
- `ir` may depend only on `ast` and `typeck`.
- `codegen` may depend only on `ir`.
- `tools/*` may depend on any crate, but no crate may depend on `tools/*`.
- Circular dependencies are a hard error. Never introduce them.

## 4. New File Checklist

Before creating any new file, the agent must confirm every item below:

- [ ] The file belongs in the correct designated directory.
- [ ] The file follows the naming convention.
- [ ] If this is a new crate, `Cargo.toml` is created and added to the workspace members list in the root `Cargo.toml`.
- [ ] If this is a new crate, a stub `src/lib.rs` with the required doc comment is created.
- [ ] If this is a new test file, it lives in `tests/` and is named `{crate}_tests.rs`.
- [ ] No existing file is silently overwritten.
- [ ] If the file introduces technical debt, a matching entry is added to `DEBT.md` before the change is considered complete.

## 5. Forbidden Actions

- Never create files in the project root except `Cargo.toml`, `Cargo.lock`, `AGENTS.md`, `README.md`, and `DEBT.md`.
- Never use `mod.rs`. Use `module_name.rs` at the same level instead.
- Never place test code inside `src/` files. Tests belong in `tests/`.
- Never commit `target/`, generated files, build artifacts, or `.DS_Store`.
- Never add a dependency to any crate without updating the corresponding `Cargo.toml`.
- Never rewrite an existing file unless the current conversation explicitly authorizes it.

## 6. Scaffold Law

When scaffolding a new crate that is not yet implemented:

- Create `Cargo.toml` with the correct crate name and version.
- Create `src/lib.rs` containing exactly:

```rust
//! Scaffold — not yet implemented.
```

- Do not generate placeholder logic.
- Do not generate dummy structs.
- Do not generate TODO functions.

## 7. Technical Debt Tracking Law

- Any workaround, temporary fix, or known limitation introduced by an agent must be logged in `DEBT.md` at the project root.
- An agent must never introduce technical debt silently. Log it or do not introduce it.
- Every debt entry must use this exact format:

```text
- [ ] [crate_name] Short description of the debt
      Introduced: <commit hash or PR>
      Fix: <what needs to happen to resolve it>
```

- `Introduced:` must name the commit hash or PR that created the debt. Use `uncommitted` only before the first commit of the change, then replace it before completion.
- `Fix:` must describe the concrete action required to remove the debt.
- If a debt item is resolved in the same change that introduced it, do not add it to `DEBT.md`.
- `DEBT.md` is mandatory repository state. Agents must update it when debt status changes.

## 8. Commit Law

- Every meaningful change must be committed immediately once the repository is stable and validated.
- Every commit message must follow this exact format:

```text
<type>(<crate>): <short description>

- bullet point of what changed
- bullet point of why

Debt: <DEBT.md ref if applicable, else "none">
Validation: <what was run, e.g. cargo test -p thagore-lexer>
```

- Valid `<type>` values are:
  - `feat`
  - `fix`
  - `refactor`
  - `test`
  - `docs`
  - `chore`
  - `debt`
- Never commit without running `cargo test` on the affected crate.
- If multiple crates are affected, run `cargo test` for each affected crate.
- If no crate is affected, run the most relevant validation for the change and state it exactly in `Validation:`.
- Never commit generated files, `target/`, or `.DS_Store`.
- Never use vague commit subjects.
- Never combine unrelated changes in one commit.

## 9. Testing Law

- Every crate must maintain a `tests/{crate}_tests.rs` file.
- Every public function must have at least one unit test.
- Every meaningful change must add or update tests unless the user explicitly forbids it.
- Required edge cases per crate are mandatory coverage:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thagore-foundation/thagore](https://github.com/thagore-foundation/thagore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
