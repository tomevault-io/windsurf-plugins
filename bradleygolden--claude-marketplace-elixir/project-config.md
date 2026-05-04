---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is a **Claude Code plugin marketplace** for Elixir and BEAM ecosystem development. It provides automated development workflows through hooks that trigger on file edits and git operations.

## Architecture

### Plugin Marketplace Structure

```
.claude-plugin/
└── marketplace.json          # Marketplace metadata and plugin registry

plugins/
├── elixir/                   # Combined Elixir plugin (recommended)
│   ├── .claude-plugin/
│   │   └── plugin.json       # Plugin metadata
│   ├── hooks/
│   │   └── hooks.json        # Hook definitions
│   ├── lib/
│   │   └── utils.sh          # Shared utilities
│   ├── scripts/
│   │   ├── post-edit.sh      # PostToolUse hook
│   │   └── pre-commit.sh     # PreToolUse hook
│   ├── skills/               # Inherited from core
│   └── README.md
├── core/                     # Legacy: Core Elixir development
├── credo/                    # Legacy: Credo static analysis
├── ash/                      # Legacy: Ash Framework codegen
├── dialyzer/                 # Legacy: Dialyzer type analysis
├── ex_doc/                   # Legacy: ExDoc documentation
├── ex_unit/                  # Legacy: ExUnit testing
├── mix_audit/                # Legacy: Dependency security
├── precommit/                # Legacy: Phoenix precommit alias
└── sobelow/                  # Legacy: Security analysis

test/plugins/
├── elixir/                   # Elixir combined plugin tests
│   ├── README.md
│   ├── postedit-test/
│   ├── precommit-test/
│   └── test-elixir-hooks.sh
├── core/                     # Core plugin tests
├── credo/                    # Credo plugin tests
├── ash/                      # Ash plugin tests
└── ...                       # Other legacy plugin tests
```

### Key Concepts

**Marketplace (`marketplace.json`)**: Top-level descriptor that defines the marketplace namespace ("elixir"), version, and lists available plugins. The `pluginRoot` points to the plugins directory.

**Plugin (`plugin.json`)**: Each plugin has metadata (name, version, description, author) and a `hooks` field pointing to its hook definitions.

**Hooks (`hooks.json`)**: Define automated commands that execute in response to Claude Code events:
- `PostToolUse`: Runs after Edit/Write tools (e.g., auto-format, compile check)
- `PreToolUse`: Runs before tools execute (e.g., pre-commit validation before git commands)

### Hook Implementation Details

**Elixir plugin** (recommended) - Comprehensive Elixir development:

1. **Post-edit** (non-blocking, PostToolUse, 30s timeout): After editing `.ex`/`.exs` files:
   - Auto-formats the file
   - Checks for compilation errors
   - Runs Credo (if dependency present)
   - Checks Ash codegen (if dependency present)
   - Runs Sobelow security check (if dependency present)

2. **Pre-commit** (blocking, PreToolUse, 180s timeout): Before `git commit`:
   - Defers to `mix precommit` alias if present (Phoenix 1.8+)
   - Otherwise runs: format check, compile, unused deps, Credo, Ash codegen, Dialyzer, ExDoc, tests, mix_audit, Sobelow
   - Blocks commit on any failure

**Legacy plugins** - Individual tool plugins (deprecated):
- **Core**: Auto-format, compile check, pre-commit validation
- **Credo**: Static code analysis
- **Ash**: Ash Framework code generation
- **Dialyzer**: Static type analysis (120s timeout)
- **ExDoc**: Documentation quality validation (with locking for concurrent execution)
- **ExUnit**: Test running
- **mix_audit**: Dependency security
- **Precommit**: Phoenix precommit alias runner
- **Sobelow**: Security-focused static analysis

Hooks use `jq` to extract tool parameters and bash conditionals to match file patterns or commands. Output is sent to Claude (the LLM) via JSON with either `additionalContext` (non-blocking) or `permissionDecision: "deny"` (blocking).

### Skills

Skills provide specialized capabilities for Claude to use on demand, complementing automated hooks with user-invoked research and guidance.

**Core plugin** - Research and best practices skills:
1. **hex-docs-search** (core@elixir): Searches Hex package documentation with progressive fetch strategy
   - Searches local deps → fetched cache → fetches if needed → HexDocs API → web search
   - Stores fetched docs in `.hex-docs/` and source in `.hex-packages/`
   - Provides API documentation, function signatures, and usage examples
   - See `plugins/core/skills/hex-docs-search/SKILL.md`

2. **usage-rules** (core@elixir): Searches package-specific usage rules and best practices
   - Searches local deps → fetched cache → fetches if needed
   - Stores fetched rules in `.usage-rules/<package>-<version>/`
   - Provides coding conventions, patterns, and good/bad examples
   - Context-aware section extraction based on coding context
   - See `plugins/core/skills/usage-rules/SKILL.md`

**Skill Composition**:
Skills are designed to be **single-purpose** and **composed by agents/commands**:
- `usage-rules` provides conventions and patterns (how to use correctly)
- `hex-docs-search` provides API documentation (what's available)
- Agents can invoke both for comprehensive guidance ("best practices + API")

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bradleygolden/claude-marketplace-elixir](https://github.com/bradleygolden/claude-marketplace-elixir) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
