---
trigger: always_on
description: - Keep **cognitive complexity** below the threshold of **25** (see **Complexity and linting** below).
---

# Rust Development Rules for AI Agents

## When Creating New Code (Files, Functions, Methods, Enums)
- Keep **cognitive complexity** below the threshold of **25** (see **Complexity and linting** below).
- Keep functions under **150 lines** (enforced by Clippy `too_many_lines`).
- Prefer straightforward **data flow** (fewer threaded parameters, clearer state boundaries). This is a **design guideline**—there is no compiler lint for it; match patterns used in existing modules.
- Add `///` rustdoc to all new public **and** private items (`missing_docs` + `missing_docs_in_private_items` are both enforced; see **Lint configuration**).
- Use the **What / Inputs / Output / Details** rustdoc layout for non-trivial APIs (see **Documentation** for the template).
- Add focused **unit** tests for new logic.
- Add **integration** tests when behavior crosses modules or the CLI boundary.

## When Fixing Bugs/Issues
1. Identify the root cause before writing code.
2. Write or adjust a test that **fails** on the bug.
3. Run the test — it must fail. If it passes, the test does not reproduce the issue; adjust it.
4. Fix the bug.
5. Run the test again — it must pass. If not, iterate on the fix.
6. Add edge-case tests when they reduce future regressions.

## Always Run After Changes
Run from the repository root, in this order:
1. `cargo fmt --all`
2. `cargo clippy --all-targets --all-features -- -D warnings`
3. `cargo check`
4. `cargo test -- --test-threads=1`

**CI note:** `.github/workflows/rust.yml` runs `cargo build` and `cargo test` (with `--test-threads=1`). `.github/workflows/lint.yml` runs `cargo fmt --check` and `cargo clippy ... -D warnings` on pushes/PRs to `main`. `.github/workflows/security.yml` runs supply-chain and secret scans (no clippy). `cargo check` is still required locally before considering work done.

## Lint configuration (source of truth)

**`Cargo.toml` — `[lints.clippy]`** (excerpt; see file for full list):
```toml
[lints.clippy]
cognitive_complexity = "warn"
pedantic = { level = "deny", priority = -1 }
nursery = { level = "deny", priority = -1 }
unwrap_used = "deny"
missing_docs_in_private_items = "warn"
```

**`Cargo.toml` — `[lints.rust]`:**
```toml
[lints.rust]
missing_docs = "warn"
```

**`clippy.toml`:**
- `cognitive-complexity-threshold = 25` — used by Clippy's **`cognitive_complexity`** lint (not cyclomatic complexity; that is a different metric).
- `too-many-lines-threshold = 150` — used by Clippy's **`too_many_lines`** lint.

With `cargo clippy ... -- -D warnings`, **all warnings become errors**. That means `cognitive_complexity`, `missing_docs`, and `missing_docs_in_private_items` violations will **fail** the Clippy run.

## Code Quality Requirements

### Pre-commit checklist
Before completing any task, ensure all of the following pass:
1. **Format:** `cargo fmt --all` produces no diff.
2. **Clippy:** `cargo clippy --all-targets --all-features -- -D warnings` is clean.
3. **Compile:** `cargo check` succeeds.
4. **Tests:** `cargo test -- --test-threads=1` — all tests pass.
5. **Complexity:** New functions stay under cognitive-complexity threshold (25).
6. **Length:** New functions stay under too-many-lines threshold (150).
7. **Exceptions:** If a threshold cannot be met, add a **documented** `#[allow(...)]` with a justification comment. Use sparingly.

### Documentation
- **All** new public and private functions, methods, structs, enums, traits, and modules must have `///` rustdoc comments.
  - `missing_docs` fires on public items.
  - `missing_docs_in_private_items` fires on private items.
  - Both are **warn**, promoted to **error** by `-D warnings`.
- For non-trivial APIs, use the structured rustdoc layout with **What**, **Inputs**, **Output**, and **Details** sections:
  ```rust
  /// What: Brief description of what the function does.
  ///
  /// Inputs:
  /// - `param1`: Description of parameter 1
  /// - `param2`: Description of parameter 2
  ///
  /// Output:
  /// - Description of return value or side effects
  ///
  /// Details:
  /// - Additional context, edge cases, or important notes
  pub fn example_function(param1: Type1, param2: Type2) -> Result<Type3> {
      // implementation
  }
  ```
- Documentation must include all four sections: **What**, **Inputs**, **Output**, and **Details**.

### Testing

**For bug fixes:**
1. Create a failing test that reproduces the issue.
2. Fix the bug.
3. Verify the test passes.
4. Add additional edge-case tests if applicable.

**For new features:**
1. Add unit tests for new functions/methods.
2. Add integration tests for new workflows.
3. Test error cases and edge conditions.
4. Ensure tests are meaningful and cover the functionality.

**Test guidelines:**
- Tests must be deterministic and not rely on undeclared external machine state.
- For code paths that would mutate the system, exercise **dry-run** behavior via the `dry_run` bool field (wired from the CLI `--dry-run` flag through the app), or use equivalent test doubles. Do **not** blindly pass `--dry-run` to every shell command.
- Always run tests with `--test-threads=1` to avoid parallel interference.

## Code style conventions
- **Edition:** Rust 2024 (see `Cargo.toml`).
- **Naming:** Clear and descriptive; clarity over brevity.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Firstp1ck/Pacsea](https://github.com/Firstp1ck/Pacsea) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
