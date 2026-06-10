---
trigger: always_on
description: Terminal Agent is a CLI-first AI assistant that lets you collaborate with large language models directly from your shell. It wraps multiple provider APIs (OpenAI, Anthropic, Bedrock, Google, Ollama, etc.), and also supports direct local `llama.cpp`-based GGUF inference through the `llama` provider. It renders markdown responses with Glamour, and exposes commands like `agent ask` for general questions and `agent task` for planning and executing terminal workflows. Configuration lives in `~/.confi
---

# Terminal Agent Overview

Terminal Agent is a CLI-first AI assistant that lets you collaborate with large language models directly from your shell. It wraps multiple provider APIs (OpenAI, Anthropic, Bedrock, Google, Ollama, etc.), and also supports direct local `llama.cpp`-based GGUF inference through the `llama` provider. It renders markdown responses with Glamour, and exposes commands like `agent ask` for general questions and `agent task` for planning and executing terminal workflows. Configuration lives in `~/.config/terminal-agent/config.json`, can be overridden via CLI flags, and supports provider/model switching as well as MCP tool definitions.

Tool execution permissions can be set in `~/.config/terminal-agent/config.json` and in project-local `.terminal-agent.json` files. Local configs are discovered by walking from the current working directory up to the filesystem root; the closest file wins when priorities overlap. Permissions use the same action expression format shown in confirmations (for example `unix("aws sso login", profile="dev")`) and support `allow`, `deny`, and `ask` lists. When prompted, `yes!` or `no!` will remember a decision by writing to the nearest `.terminal-agent.json`, falling back to the global config when no local file exists.

Whether a tool prompts by default (when no `allow`/`deny`/`ask` rule matches) is driven by a per-tool **permission category** rather than a hardcoded list. Tools declare their category via the optional `tools.CategorizedTool` interface (`PermissionCategory()` returning `read`, `write`, or `execute`):

- **read** (`read`, `file_search`, `websearch`, `ask_user`, `final_answer`) — never prompts.
- **write** (`file_edit`) — does not prompt when the target path resolves inside the task workspace root; prompts otherwise. `file_edit` is already physically confined to the root by `ensureWithinRoot`, so the prompt branch is a backstop for future write tools.
- **execute** (`unix`, `python`) — always prompts; arbitrary effect.
- **undeclared** (MCP tools, any new tool not implementing the interface) — treated as `execute` and gated by default. This is deliberate: a tool with no declared category must not run silently.

The `allow`/`deny`/`ask` rule engine is unchanged and remains the override layer on top of these category defaults: `ask` always prompts, `deny` beats `allow` at equal-or-higher priority, and matched decisions are cached per run. The category only decides the fallback when no rule matches. See `permissionCategoryFor`/`confirmTool` in `internal/agent/task.go` and `ConfirmWithDefault` in `internal/agent/confirmation.go`.

The repo is structured around the Go implementation of the binary (see `cmd/` and `internal/`), plus documentation in `docs/` that backs the published site. Installation can happen via Homebrew, downloading release archives, `go install`, or building from source with Taskfile tasks such as `task build`/`task install`.

## Contribution Conventions

- Do **not** include design specs or implementation plans in pull requests or git commits. Keep them local only (e.g. under `docs/superpowers/`, which is git-ignored); a PR should contain only the production code, tests, and user-facing docs for the change.
- Do **not** commit build artifacts (the `agent` / `agent-gui` binaries produced by `task build`). They are git-ignored; never `git add` them into a commit.
- Prefer named constants over magic strings and numbers. If a value has product meaning, is reused, or is part of configuration/default behavior, define it as a const and reference the const in production code and tests.
- Keep source files focused. Do not let files grow into catch-all buckets; when a file starts mixing separate responsibilities, split it along durable product or implementation boundaries.
- Periodically revisit filenames as code evolves. A file name should still describe what the file holds today, not only the original intent from when it was created.

## Development Philosophy

- Optimize for the best durable solution, not the smallest or quickest implementation path. Prefer designs that make the idea clear, robust, and future-proof even when they require more code.
- Avoid shortcuts, temporary workarounds, and narrowly scoped fixes unless the user explicitly asks for them. If a shortcut is unavoidable, call it out clearly and explain the tradeoff.
- Treat coding as the easy part. Spend the necessary effort on understanding the product idea, long-term architecture, edge cases, and future maintenance before choosing an implementation.

## Architecture Notes

`internal/app` is the shared application-service layer used by both the CLI and the GUI.

- CLI commands in `internal/commands/` call `internal/app` service methods such as `Ask`, `AskEvents`, `Chat`, and `TaskEvents`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [laszukdawid/terminal-agent](https://github.com/laszukdawid/terminal-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-10 -->
