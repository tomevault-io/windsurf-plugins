---
trigger: always_on
description: **CRITICAL**: Mandatory rules for AI agents working on XZatoma. Non-compliance
---

# AGENTS.md - AI Agent Development Guidelines

**CRITICAL**: Mandatory rules for AI agents working on XZatoma. Non-compliance
will result in rejected code.

---

## Critical Rules

## Rule 0: Use the Agent Harness Tools

Use `agent_harness` for all agent interactions

### Rule 1: File Extensions

- Use `.yaml` for ALL YAML files (NOT `.yml`)
- Use `.md` for ALL Markdown files (NOT `.MD`, `.markdown`)
- Use `.rs` for ALL Rust files

CI/CD pipelines expect `.yaml`. Using `.yml` causes build failures.

### Rule 2: Markdown File Naming

- Use `lowercase_with_underscores.md` for all documentation files
- `README.md` is the ONLY exception to the lowercase rule
- Never use CamelCase, kebab-case, spaces, or uppercase

Inconsistent naming breaks documentation links.

### Rule 3: No Emojis

- No emojis in code, comments, documentation, or commit messages
- Exception: This AGENTS.md file only

Emojis cause encoding issues and break tooling.

### Rule 4: Quality Gates (ALL Must Pass)

Run in this order before claiming any task complete:

```bash
cargo fmt --all
cargo check --all-targets --all-features
cargo clippy --all-targets --all-features -- -D warnings
cargo test -p xzatoma --lib -- --skip providers::copilot --skip mcp::auth
```

**NEVER run `cargo test --all-features` or `cargo test -p xzatoma --lib` bare.**
The `providers::copilot` and `mcp::auth` modules link against the macOS
`Security.framework` via the `keyring` crate. On macOS, any freshly compiled
binary that links that framework triggers an OS Keychain access dialog on first
execution, even when no `#[ignore]`-guarded test function runs. Always pass
`--skip providers::copilot --skip mcp::auth` to keep tests hermetic.

**Keyring access chain -- know this to avoid re-introducing the bug:**

```text
keyring::Entry::{get,set,delete}_password
  <- CopilotProvider::{get_cached_token, cache_token, clear_cached_token}
    <- is_authenticated(), authenticate(), fetch_copilot_models()
  <- TokenStore::{save,load,delete}_token
    <- AuthManager::get_token(), handle_401()
```

`CopilotProvider::new()` itself is safe -- it only stores strings. The keyring
is hit as soon as any method that checks or updates authentication is called.

Modules that are gated by `#[ignore = "requires system keyring"]` (not by
`--skip`) include tests in `src/acp/stdio.rs` that call `run_client_server_test`
or `create_session`. Those helpers initialize a session with the default config
(provider = copilot) which calls `provider.list_models()` -> `authenticate()` ->
`get_cached_token()` -> keyring. Never remove the `#[ignore]` annotation from
those tests.

To run the full suite including keyring round-trips (only in a trusted
environment where Keychain prompts are acceptable):

```bash
XZATOMA_RUN_KEYCHAIN_TESTS=1 cargo test -p xzatoma --lib -- --include-ignored
```

**MANDATORY**: All Markdown files must pass linting and formatting checks:

```bash
markdownlint --fix --config .markdownlint.json "${FILE}"
prettier --write --parser markdown --prose-wrap always "${FILE}"
```

Stop immediately and fix if any command fails.

### Rule 5: Documentation is Mandatory

- Create `docs/explanation/<feature_name>_implementation.md` for every feature
  or task
- Add `///` doc comments to every public function, struct, enum, and module
- Include runnable examples in doc comments (they are compiled by `cargo test`)
- Never skip documentation because "code is self-documenting"

### Rule 6: Use the Agent Harness Tools

Do not write custom scripts for tasks that can be accomplished with the agent
tools.

---

## Project Overview

- **Name**: XZatoma
- **Type**: Autonomous AI agent CLI
- **Language**: Rust (latest stable)
- **Purpose**: Execute tasks through conversation with AI providers using basic
  file system and terminal tools
- **Providers**: GitHub Copilot, Ollama

### Module Structure

```text
src/
├── main.rs              # Entry point
├── lib.rs               # Library root
├── cli.rs               # CLI parsing and user interface
├── config.rs            # Configuration management
├── error.rs             # Error types and conversions
├── agent/               # Autonomous execution loop
│   ├── mod.rs
│   ├── agent.rs         # Main agent logic
│   ├── conversation.rs  # Message history
│   └── executor.rs      # Tool execution
├── providers/           # AI provider abstraction
│   ├── mod.rs
│   ├── base.rs          # Provider trait
│   ├── copilot.rs       # GitHub Copilot
│   └── ollama.rs        # Ollama
└── tools/               # Basic file and terminal tools
    ├── mod.rs
    ├── file_ops.rs      # File operations
    ├── terminal.rs      # Terminal execution
    └── plan.rs          # Plan parsing
```

### Architecture Principles

XZatoma is intentionally simple. Do NOT over-engineer it.

- Separate concerns by technical responsibility: CLI, agent, providers, tools
- Avoid unnecessary abstraction layers
- Do not abstract prematurely - wait until you have 3 examples
- Do not add complex inheritance hierarchies
- Keep tools generic (file ops, terminal) - no specialized tools

### Module Dependencies

| Module       | Responsibility                 | Dependencies         |
| ------------ | ------------------------------ | -------------------- |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xbcsmith/xzatoma](https://github.com/xbcsmith/xzatoma) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
