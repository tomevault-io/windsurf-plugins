---
trigger: always_on
description: This is a Rust-based PHP language server. Performance and memory
---

# AI Agent Guidelines for PHPantom

This is a Rust-based PHP language server. Performance and memory
efficiency are critical -- PHPantom is one of the fastest language
servers available and it must stay that way.

## Before You Start

Read these to orient yourself:

- `src/types/` — Core data structures (`ClassInfo`, `MethodInfo`, `FunctionInfo`, `PropertyInfo`, etc.)
- `src/lib.rs` — `Backend` struct definition and all module declarations
- `docs/ARCHITECTURE.md` — Symbol resolution pipelines and design decisions
- `docs/todo.md` — Current backlog of known gaps and missing features

## Project Structure

Run `ls src/` for the current layout, and see the **Module Layout**
and pipeline sections of [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md)
for how the pieces fit together. A few durable landmarks, since the
module tree moves around:

- **The shared type engine lives under `src/type_engine/`.**
  `type_engine/resolver/` resolves a subject expression to a
  `ClassInfo`, and `type_engine/variable/forward_walk/` is the forward
  walker that answers "what is the type of this expression here?" — it
  is shared by diagnostics, hover, go-to-definition, and signature help,
  not just completion. `completion/` holds completion-specific code only.
  Do not build a second type-resolution path (see the anti-patterns
  below).
- **Parsing** is in `parser/` (PHP source → `ClassInfo`/`FunctionInfo`)
  and `docblock/` (PHPDoc tags, templates, conditional types).
- **The data model** is in `types/` and `php_type/`.
- **Cross-file symbol resolution** is `resolution.rs` (class/function
  lookup), with `composer.rs`/`classmap_scanner/` for autoloading,
  `inheritance/` for merging parent/trait/mixin members, and
  `virtual_members/` for synthesized members (`@method`/`@property`,
  Laravel Eloquent).
- **Each LSP feature** is its own module (`hover/`, `definition/`,
  `diagnostics/`, `code_actions/`, `references/`, `rename/`, …).
- **Embedded stubs** (`stubs.rs`, `stub_patches.rs`) supply the standard
  library; the `analyse` and `fix` CLI subcommands live in `analyse/`
  and `fix.rs`.

Tests live in `tests/`: `tests/integration/` has one file per feature
area (`completion_*.rs`, `definition_*.rs`, `code_action_*.rs`, …) with
shared helpers in `tests/integration/common/mod.rs`; `tests/unit/`,
`tests/fixture_runner.rs`, and the ported Psalm/PHPStan assertion suites
round it out.

## Before Committing

Always run these checks before considering any change complete:

```bash
cargo clippy --fix --allow-dirty -- -D warnings
cargo fmt
```

Run `cargo fmt` after clippy, not before -- clippy fixes can affect
formatting.

## Contributing Guidelines

Read and follow [docs/CONTRIBUTING.md](docs/CONTRIBUTING.md) for the
full set of CI checks, testing conventions, and code style rules.

## Key Rules

- **Performance is critical.** Every allocation, clone, and lock
  matters. Avoid unnecessary heap allocations, prefer `&str` over
  `String` where possible, and be mindful of hot paths. Do not
  introduce regressions in startup time or memory usage.
- **Run the full lint pipeline.** `cargo clippy` and `cargo fmt` must
  pass with zero warnings before every commit. Do not skip this.
- **Update the changelog.** Add an entry under `## [Unreleased]` in
  `docs/CHANGELOG.md` for bug fixes and new features. Skip purely
  internal refactors that don't change observable behaviour. Write
  for end users, not developers. Include `Contributed by @username`
  with the GitHub username of the author.
- **Reference issues in commits.** When fixing a GitHub issue, include
  `Closes #123` in the commit message body.
- **Prefer single tests.** Run individual tests (`cargo test test_name`)
  rather than the full suite during development for faster feedback.
- **Debug root causes.** When investigating a bug, determine the root
  cause rather than patching symptoms.
- **Comments only where they add value.** Don't add obvious or
  boilerplate comments. Do comment tricky logic, non-obvious design
  decisions, and workarounds. Follow existing conventions. Don't leave
  tombstone comments (e.g. "this was moved to `foo.rs`", "previously
  did X") — they stop being helpful the moment the commit lands and
  just rot; git history is the record of what moved where.
  All files must end with a newline.

## Working on examples/php/

The [CONTRIBUTING guide](docs/CONTRIBUTING.md) has the CI checklist; the
notes here are the agent-specific pitfalls when editing the demo files.

`examples/php/` is the user-facing playground people open to verify
PHPantom works on plain PHP (no framework). It is a standalone project
with no external dependencies — `autoload.php` is a list of hardcoded
`require_once` lines, not a `composer.json` — split into:

- **One demo file per LSP feature area** (all in namespace `Demo`):
  `completion.php`, `diagnostics.php`, `definition.php`,
  `code_actions.php`, `hover.php`, `signature_help.php`,
  `inlay_hints.php`, `code_lens.php`, `semantic_tokens.php`. These hold
  the demo classes themselves, including top-level "Try:" comments and
  simple top-level expressions users can trigger completion on directly,
  plus classes whose _methods_ contain completion triggers (e.g.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PHPantom-dev/phpantom_lsp](https://github.com/PHPantom-dev/phpantom_lsp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
