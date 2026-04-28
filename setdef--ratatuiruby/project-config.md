---
trigger: always_on
description: SPDX-FileCopyrightText: 2026 Kerrick Long <me@kerricklong.com>
---

<!--
  SPDX-FileCopyrightText: 2026 Kerrick Long <me@kerricklong.com>
  SPDX-License-Identifier: CC-BY-SA-4.0
-->

# AGENTS.md

## Project Identity

Project Name: ratatui_ruby

Description: A high-performance Ruby wrapper for the Ratatui TUI library.

Architecture:

- **Frontend (Ruby):** Pure `Data` objects (Ruby 3.2+) used in Frames and/or defining the View Tree. Immediate mode.
- **Backend (Rust):** A generic renderer using `ratatui` and `magnus` that traverses the Ruby `Data` tree and renders to the terminal buffer.

## Stability & Compatibility

- **Project Status:** Pre-1.0. 
- **User Base:** First external users (as of 2026-01-03).
- **Breaking Changes:** Backward compatibility is **NOT YET** a priority. Since there are few external users, you are encouraged to refactor APIs for better ergonomics and performance even if it breaks existing code. This will when we ship v1.0.0.
- **Requirement:** All breaking changes **MUST** be explicitly documented in the [CHANGELOG.md](CHANGELOG.md)'s **Unreleased** section to ensure transparency as the project evolves toward 1.0. Migration guides **MUST** be included in `doc/` when we release a new minor version.

## 1. Standards

### STRICT REQUIREMENTS

- **Check Before Implementing:** FIRST check tests for existing coverage. If it works, say so and point to the test.
- Every file MUST begin with an SPDX-compliant header. License by directory: `LGPL-3.0-or-later` for library source (`lib/`, `ext/`, `test/`, `sig/`); `MIT-0` for widget and verify examples (`examples/widget_*`, `examples/verify_*`); `AGPL-3.0-or-later` for app examples, tasks, and tooling (`examples/app_*`, `tasks/`, `bin/`); `CC-BY-SA-4.0` for documentation. `reuse annotate` can help you generate the header. **For Ruby files**, wrap SPDX comments in `#--` / `#++` to hide them from RDoc output.
- Every line of Ruby MUST be covered by tests that would stand up to mutation testing.
  - Tests must be meaningful and verify specific behavior or rendering output; simply verifying that code "doesn't crash" is insufficient and unacceptable.
  - **Prefer snapshot tests** (`assert_snapshots`, plural) over manual `buffer_content` assertions for UI widgets. Snapshots are self-documenting and easier to maintain.
  - For UI widgets, use `with_test_terminal` and snapshot assertions to verify terminal buffer content.
- Every line of Rust MUST be covered by tests that would stand up to mutation testing.
  - Tests must be meaningful; simply verifying that code "doesn't crash" or "compiles" is insufficient and unacceptable.
  - Each widget implementation must have a `tests` module with unit tests verifying basic rendering.
- **Pre-commit:** Use `bin/agent_rake` to ensure commit-readiness. See Tools for detailed instructions.
- **Git Pager:** ALWAYS set `PAGER=cat` for ALL `git` commands (e.g., `PAGER=cat git diff`). This is mandatory.

### Tools

- **NEVER** run `bundle exec rake` directly. **NEVER** run `bundle exec ruby -Ilib:test ...` directly.
- **ALWAYS use `bin/agent_rake`** for running tests, linting, or checking compilation.
  - **Usage:**
    - Runs default task (compile + test + lint): `bin/agent_rake`
    - Runs specific task: `bin/agent_rake test:ruby` (for example)
- **Snapshot Testing:** When tests fail due to intentional behavior changes (not bugs), update snapshots:
  - **Command:** `UPDATE_SNAPSHOTS=1 bin/agent_rake test:ruby`
  - This regenerates all `snapshot/*{txt,ansi}` files to match current output
  - **When to use:** After modifying widget rendering, changing default values, or updating UI behavior
  - **When NOT to use:** For actual test failures that indicate bugs in your code
  - After updating, verify the changes make sense by reviewing the diff of `.snapshot` files
- **Setup:** `bin/setup` must handle both Bundler and Cargo dependencies.
- **Git:** ALWAYS set `PAGER=cat` with `git`, `git`, etc.. **THIS IS CRITICAL!**
- **Rake:** Our rake tasks use `git ls-files`, so errors happen when you move or delete files. In this case, ask the user to stage changes for you.

### Ruby Standards

- Use `Data.define` for all value objects (UI Nodes). (Prefer `class Foo < Data.define()` over `Foo = Data.define() do`).
- Define types in `.rbs` files. Don't use `untyped` just because it's easy; be comprehensive and accurate. Do not include `initialize` in `.rbs` files; use `self.new` for constructors instead.
- Every public Ruby class/method must be documented for humans in RDoc (preferred)--**not** YARD--or markdown files (fallback), and must have `*.rbs` types defined.
- Every significant architectural and design decision must be documented for contributors in markdown files. Mermaid is allowed.
- **Rust-backed methods:** For methods implemented in Rust (magnus bindings), use RDoc directives instead of empty method bodies. Use `##` followed by `:method:`, `:call-seq:`, and prose. End with `(Native method implemented in Rust)`. See `lib/ratatui_ruby.rb` for examples.
- Refer to [doc/contributors/design/ruby_frontend.md](doc/contributors/design/ruby_frontend.md) for detailed design philosophy regarding the Immediate Mode paradigm including Data-Driven UI and Frames.

### Rust Standards

- **Crate Type:** `cdylib`.
- **Bindings:** Use [magnus](https://github.com/matsadler/magnus).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [setdef/RatatuiRuby](https://github.com/setdef/RatatuiRuby) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
