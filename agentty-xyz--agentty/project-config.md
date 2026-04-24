---
trigger: always_on
description: TUI tool to manage agents.
---

# Agentty

TUI tool to manage agents.

# MANDATORY

> **STOP! Read this section before proceeding.**
> These rules are absolute and take precedence over all others.

- **Document Code:** Document all added or updated code using docstrings. When touching existing code, add or refresh docstrings so the changed behavior is clearly described.
- **Update AGENTS.md:** Update the relevant `AGENTS.md` file only when a user instruction establishes a critical, persistent preference, convention, or workflow rule. Do not update it for one-off tasks.
- **Semantic Guidance Only:** Keep `AGENTS.md` files focused on purpose, entry points, invariants, change routing, and docs-sync notes. Do not maintain exhaustive per-directory file inventories.
- **Local Paths Only:** In `AGENTS.md`, do not use parent-directory relative paths. Each file should describe only its own directory or module boundary.
- **Context First:** Before broad exploration, read the nearest available `AGENTS.md`. If the current directory does not have one, fall back to the closest ancestor guide and the architecture docs in `docs/site/content/docs/architecture/`.
- **Context7 First:** If Context7 is connected as an MCP server, use it to retrieve the latest documentation and API details for the tools and libraries used in the task.
- **Test Isolation for External Commands:** Keep isolated single-command tests real when they validate one external command call, but for higher-level flows that involve multiple external command calls, always extract trait boundaries and mock them with `mockall` (`#[cfg_attr(test, mockall::automock)]`) to reduce runtime and flakiness.

## Project Facts

- Project is a Rust workspace.
- The `crates/` directory contains all workspace members.
- All workspace crates use the `ag-` prefix (e.g., `ag-xtask`).
- `agentty`: A binary crate providing the CLI interface using Ratatui.
- **Workflow**: Agents are run in isolated git worktrees.
- **Review**: Users review changes using the Diff view (`d` key in chat) which shows the output of `git diff` in the session's worktree.
- **Output**: Agent `stdout` and `stderr` are captured in parallel using `tokio` tasks to ensure prompts and errors are visible.

## Product-vs-Repository Prompt Scope

Agentty is developed with Agentty, so prompts often describe developer workflows
that should become Agentty product behavior rather than instructions to operate on
this repository's current session.

- Treat workflow prompts about branches, worktrees, commits, reviews, sessions,
  agent prompts, or agent behavior as requirements for Agentty's user-facing
  functionality by default.
- Apply those prompts to the projects that Agentty manages for its users, not to
  the local Agentty development checkout, unless the user explicitly names this
  repository, this worktree, or the current branch/session.
- If a prompt could require a mutating action against the local repository, first
  reinterpret it as a product requirement. Ask for clarification only when the
  requested scope still remains ambiguous after that reinterpretation.
- Example: `make sure main branch is always up-to-date` means Agentty should
  provide behavior that keeps users' project `main` branches current for managed
  workflows; it does not mean this repository's local `main` branch should be
  updated.

## Rust Project Style Guide

- **Dependency Management:** ALL dependencies (including `dev-dependencies` and `build-dependencies`) must be defined in the root `Cargo.toml` under `[workspace.dependencies]`.
- All workspace crates must use `workspace = true` for shared package metadata and dependencies. Never define a version number inside a crate's `Cargo.toml`.
- **Release Profile:** Maintain optimized release settings in `Cargo.toml` (`codegen-units=1`, `lto=true`, `opt-level="s"`, `strip=true`) to minimize binary size.
- Use `ratatui` for terminal UI development.
- **Constructors:** Only add `new()` and `Default` when there is actual initialization logic or fields with meaningful defaults. For unit structs or zero-field structs, construct directly (e.g., `MyStruct`) — do not add boilerplate `new()` / `Default` impls.
- **Constructors:** Prefer `Type::new(...)` associated constructors over standalone helper functions when constructing that type.
- **Function Ordering:** Order functions to allow reading from top to bottom (caller before callee):
  - Public functions first.
  - Followed by less public functions (e.g., `pub(crate)`).
  - Private functions last.
  - If a function has multiple callees, they should appear in the order they are first called within that function.
- **File Naming:** Use **singular** names for Rust source files (e.g., `model.rs`, `icon.rs`, `agent.rs`). Do not use plural forms.
- **Module Layout:** Prefer `module.rs` paired with `module/` for modules that have child modules. Do not introduce new `mod.rs` files.
- **Parent Module Router Rule:** For every `module.rs` file paired with a `module/` directory, keep `module.rs` router-only. It may declare child modules and re-export child APIs, but must not define runtime logic, functions, structs, enums, traits, impl blocks, or constants.
- **Imports:** Always place imports at the top of the file. Do not use local `use` statements within functions or other blocks.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agentty-xyz/agentty](https://github.com/agentty-xyz/agentty) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
