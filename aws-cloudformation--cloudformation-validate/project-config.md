---
trigger: always_on
description: `cloudformation-validate` is a fast, offline validator for
---

# CLAUDE.md

`cloudformation-validate` is a fast, offline validator for
AWS CloudFormation templates: parse JSON/YAML → structured diagnostics (schema, semantic, security,
best-practice). Rules + schemas compile into the binary - no network, no credentials. Ships as a Rust CLI,
Rust library, Node WASM package, Python package, Go module, and JVM (Kotlin/Java) library over one shared core.

Deeper architecture notes live in `.kiro/steering/` (`product.md`, `structure.md`, `tech.md`, `private-context.md`,
and `version-control.md`).

## Confidential agent context and skills

Before starting any task, if `.kiro/steering/private/` exists, use a direct filesystem directory read that does not
apply `.gitignore` to recursively discover and read every readable regular file in it before planning or making changes.
Do not rely on a gitignore-aware glob or search as the sole discovery mechanism, and do not follow symlinks that resolve
outside `.kiro/steering/private/`. Use applicable content as supplemental agent context, and follow any task-relevant
skill instructions found there, resolving conflicts according to the normal instruction priority. Treat both filenames
and contents as confidential: do not quote, summarize, or copy them into tracked files, logs, commit messages, review
descriptions, or responses unless the user explicitly asks for that specific disclosure. The directory and its contents
must remain untracked and must never be added to version control. If the directory is absent or empty, continue normally.

## Version control

Never run `git add` or `git commit` in this repository for any path. Do not stage or commit changes through another
tool. Leave all changes unstaged for the user to review and manage.

## Commands and validation selection

All `cargo` commands run from `src/`. The toolchain is pinned by `src/rust-toolchain.toml`. Run only checks that
exercise the files and behavior changed; if no files changed, run no tests or validation commands.

```bash
cd src

# Build
cargo build                                   # whole workspace (debug)
cargo build -p cfn-validate                   # CLI -> target/debug/cfn-validate (add --release for optimized)

# Core Rust tests - only when they cover the changed behavior
cargo test -p cloudformation-validate-cel-engine <name>               # single crate / filtered test - preferred while iterating
cargo test --workspace 2>&1 | tee ../tmp/test-output.txt   # broad core changes only; at most once at completion
# CI runs coverage, not plain test: cargo llvm-cov --locked --profile ci --workspace --no-fail-fast

# Required after every Rust source change
cargo fmt --all
cargo clippy --locked --all-targets --workspace -- -D warnings

# Run the CLI
cargo run -p cfn-validate -- <template|dir> --engine rego|cel|composite --format standard|detailed --level fatal|error|warn|info|debug
cargo run -p cfn-validate -- --list-rules
```

Validation depends on the changed surface:

- Core Rust changes: run format, clippy, and the narrowest Cargo tests that exercise the change. Use the full workspace
  suite only for broad or cross-crate core changes for which it provides meaningful coverage.
- Binding-layer Rust changes: run format and clippy, then the affected binding's `build.sh` and `tests/run.sh`. Do not
  use `cargo test` as a substitute; it does not exercise the packaged Node.js, JVM, Python, or Go API. Generated binding
  artifacts are committed only by the `build-artifacts` workflow. If generation is needed for local verification,
  generate, test, and then revert every generated binding artifact.
- Non-Rust-only changes such as documentation, GitHub workflows, scripts, or binding-language code: do not run Cargo
  format, clippy, or tests unless the file is a Cargo/build input and the command actually exercises it. Use the
  artifact-specific syntax checker, build, test runner, or dry-run instead.
- Rule, schema-data, or template changes still require the focused validator, engine-parity, corpus, and snapshot
  checks that exercise the changed diagnostics; they do not justify unrelated Cargo tests.

### Debugging tools (use these, not `println!`)

```bash
# Dump the full SemanticModel - ALWAYS start here. If the model is wrong, fix template-model.
cargo run -p cloudformation-validate-template-model --example inspect -- <template>

# Accuracy vs cfn-lint. Requires a local cfn-lint checkout - first check whether cfn-lint is available on the
# machine (`cfn-lint --version`), then ask the user for the checkout path; never assume or hardcode a location.
CFN_LINT_ROOT=<path> python3 scripts/compare_cfnlint.py --engine rego|cel|composite
```

Scratch files, debug output, and tool artifacts go in `./tmp/` at the project root - never scatter them in
the tree.

## Architecture (the non-obvious rules)

- **The two built-in rule engines must stay at parity, but parity must preserve correctness.** `EngineType::Rego` and
  `EngineType::Cel` are independent implementations of the built-in rules and must produce identical diagnostics
  (ID, severity, location, message) for any template - divergence
  is a bug. A mismatch is a signal to investigate, not permission to make the outputs agree mechanically. Establish

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aws-cloudformation/cloudformation-validate](https://github.com/aws-cloudformation/cloudformation-validate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
