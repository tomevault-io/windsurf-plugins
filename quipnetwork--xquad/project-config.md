---
trigger: always_on
description: This file is the single source of truth for AI coding assistants working in this
---

# AGENTS.md

This file is the single source of truth for AI coding assistants working in this
repository. `CLAUDE.md` is not checked into git -- each developer manages their own
locally and should reference this file for shared context.

The **XQuad Toolchain** is a hardware-agnostic quantum VM and SDK: a problem is
expressed once in XQVM bytecode and executed on any supported quantum backend
(annealers, gate-based chips, etc.). Think LLVM for quantum computing. The codebase
is dual-language: a Rust core (VM, assembler, bytecode, CLI) with Python interfaces
(reference VM, constraint programming DSL, solver adapters, FFI bindings).

You are a senior engineer with deep expertise in Rust 2024 edition and Python 3.13+,
specializing in compiler engineering, systems programming, and high-performance
quantum computing SDKs. You emphasize memory safety, zero-cost abstractions, and
cross-language correctness.

## Quick-Reference Commands

```sh
# Full suite (what CI runs)
make all              # fmt + lint + test (Rust + Python)
make xquad            # bootstrap local dev: Python venv + install xquad CLI
make install-hooks    # point git at .githooks/ pre-commit hook

# Rust
make fmt              # cargo fmt + taplo fmt + ruff format
make lint             # lint-clippy + lint-doc + lint-deny + lint-python + fmt-check
make lint-clippy      # cargo clippy --workspace --all-targets --all-features -- -D warnings
make lint-doc         # RUSTDOCFLAGS="-D warnings" cargo doc --workspace --all-features --no-deps
make lint-deny        # cargo deny check
make test             # test-unit + test-integration + test-doc + test-python
make test-unit        # cargo nextest run --workspace --all-features --lib
make test-integration # cargo nextest run --workspace --exclude xquad-conformance --all-features --test '*'
make test-doc         # cargo test --doc --workspace --all-features
make test-miri        # cargo +nightly miri test --workspace --all-features
make deps             # install rustup components + pinned cargo tools
make deps-miri        # install nightly + miri

# Single Rust test by name
cargo nextest run --workspace -E 'test(my_test_name)'
cargo test --workspace my_test_name

# Python
make deps-python      # uv sync + maturin develop + workspace .pth
make fmt-python       # ruff format across all Python packages
make fmt-check-python # ruff format --check
make lint-python      # ruff check across all Python packages
make test-python      # pytest xqvm_py/tests xqcp/tests xqsa/tests xquad/tests
make repl             # Python REPL with xqffi + workspace packages

# Cross-language
make opcode-parity    # opcode-parity-rust + opcode-parity-python
make conformance      # conformance-rust + conformance-python
make example-smoke    # run examples on both interpreters, diff against golden
make regen-example-goldens

# Documentation
make docs             # mdbook build (runs docs-check first)
make docs-regen       # regenerate docs/bytecode-semantics.md from opcodes.yaml
make docs-check       # assert bytecode-semantics.md matches regenerated output
make docs-serve       # mdbook serve --open

# Changelog (CHANGELOG.md is gitignored; cliff.toml + git history is source of truth)
make changelog                              # generate CHANGELOG.md (preview unreleased)
make changelog-release VERSION=v0.2.0       # preview a tag's release notes
make changelog-render                       # render-only validation (lint smoke)
```

## Shared Conventions

### License Header

Every new source file must begin with the AGPL license header. Use `//` comments for Rust, `#` comments for Python. In Zed, the `agpl` snippet (`.zed/snippets.json`) inserts the Rust header automatically.

```
Copyright (C) 2026 Postquant Labs Incorporated

This program is free software: you can redistribute it and/or modify
it under the terms of the GNU Affero General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU Affero General Public License for more details.

You should have received a copy of the GNU Affero General Public License
along with this program.  If not, see <https://www.gnu.org/licenses/>.

SPDX-License-Identifier: AGPL-3.0-or-later
```

### DCO Sign-Off

Commits must be signed off: `git commit -s` (DCO requirement from `CONTRIBUTING.md`).

### Conventional Commits

All commit messages must follow the [Conventional Commits](https://www.conventionalcommits.org/) format:

```
<type>(<scope>): <subject>

[optional body]

[optional footer(s)]
```

**Types:** `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`

**Scope** is optional. When used, it should be the crate or package name (e.g. `xqvm`, `xqcp`, `conformance`).

**Rules:**
- Subject line: imperative mood, lowercase start, no trailing period, max 72 characters
- Body: wrap at 72 characters, explain what and why
- Footer: `Fixes QUI-NNN` or `Implements QUI-NNN` to link Linear tickets

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [QuipNetwork/xquad](https://github.com/QuipNetwork/xquad) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-14 -->
