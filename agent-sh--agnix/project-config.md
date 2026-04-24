---
trigger: always_on
description: > Linter for agent configurations. Validates Skills, Hooks, MCP, Memory, Plugins.
---

# Project Memory: agnix

> Linter for agent configurations. Validates Skills, Hooks, MCP, Memory, Plugins.

**Repository**: https://github.com/agent-sh/agnix

## Project Instruction Files

- `CLAUDE.md` is the project memory entrypoint for Claude Code.
- `AGENTS.md` is a byte-for-byte copy of `CLAUDE.md` for tools that read `AGENTS.md` (Codex CLI, OpenCode, Cursor, Cline, Copilot).
- Keep them identical (tests enforce this).

## Critical Rules

1. **Rust workspace** - agnix-rules (data), agnix-core (lib), agnix-cli/agnix-lsp/agnix-mcp (binaries), agnix-wasm (WASM bindings)
2. **rules.json is source of truth** - `knowledge-base/rules.json` is the machine-readable source of truth. When adding a new rule, add it to BOTH `rules.json` AND `VALIDATION-RULES.md`. CI parity tests enforce this.
3. **Plain text output** - No emojis, no ASCII art
4. **Certainty filtering** - HIGH (>95%), MEDIUM (75-95%), LOW (<75%)
5. **Release binaries** - Compile with LTO, strip symbols
6. **Track work in GitHub issues** - All tasks tracked there
7. **Task is not done until tests added** - Every feature/fix must have quality tests
8. **Documentation** - Keep long-form docs in `README.md`, `SPEC.md`, and `knowledge-base/` (especially `knowledge-base/VALIDATION-RULES.md`). Keep `CLAUDE.md`/`AGENTS.md` for agent instructions only.
9. **Always follow the skill/command flow as instructed** - No deviations
10. **No unnecessary files** - Don't create summary files, plan files, or temp docs unless specifically required
11. **Never merge without waiting for claude workflow to end successfully** - It might take time, but this is the major quality gate, and most thorough review.
12. **You MUST follow the flow phases one by one** - If they state to use subagents, tools, or any specific method, you must follow it exactly as described.
13. **You MUST address all comments and reviews** - If reviewers leave comments, even minor ones, and even if not a requested change, you must address them all before merging. If you disagree, respond in the review comments. Minor comments must still be addressed.
14. **Use single dash for em-dashes** - In prose, use ` - ` (single dash with spaces), never ` -- ` (double dash). This does not apply to CLI flags like `--help` or `--fix`.

## Architecture

### Crate Dependency Graph

```
agnix-rules (data-only, generated from rules.json)
    ↓
agnix-core (validation engine)
    ↓
├── agnix-cli (command-line interface)
├── agnix-lsp (language server protocol)
├── agnix-mcp (MCP server)
└── agnix-wasm (WebAssembly bindings)
```

### Project Layout

```
crates/
├── agnix-rules/    # Rule definitions (build-time generated)
├── agnix-core/     # Core: parsers, schemas, validators, diagnostics
├── agnix-cli/      # CLI binary (clap)
├── agnix-lsp/      # LSP server (tower-lsp, tokio)
├── agnix-mcp/      # MCP server (rmcp)
└── agnix-wasm/     # WASM bindings for browser/runtime integrations
editors/
├── neovim/         # Neovim plugin
├── vscode/         # VS Code extension
├── jetbrains/      # JetBrains IDE plugin
└── zed/            # Zed extension
knowledge-base/     # 399 rules, 75+ sources, rules.json

tests/fixtures/     # Test cases by category
```

### Core Modules (agnix-core)

- `parsers/` - Frontmatter, JSON, Markdown parsing
- `schemas/` - Type definitions (13 schemas: skill, hooks, agent, mcp, cline, roo, etc.)
- `rules/` - Validators implementing Validator trait (39 validators)
- `config.rs` - LintConfig, LintConfigBuilder, ConfigError, ToolVersions, SpecRevisions
- `diagnostics.rs` - Diagnostic, Fix, DiagnosticLevel, ValidationOutcome, LintError (= CoreError), LintResult
- `eval.rs` - Rule efficacy evaluation (precision/recall/F1)
- `file_types/` - FileType enum, detect_file_type(), FileTypeDetector trait, FileTypeDetectorChain
- `file_utils.rs` - Safe file I/O (symlink rejection, size limits)
- `fixes.rs` - Auto-fix application engine
- `fs.rs` - FileSystem trait abstraction (RealFileSystem, MockFileSystem)
- `pipeline.rs` - `ValidationResult`, `validate_project()`, `validate_file()` -> `LintResult<ValidationOutcome>`
- `registry.rs` - ValidatorRegistry, ValidatorRegistryBuilder, ValidatorProvider, factory functions

### Key Abstractions

```rust
// Primary extension point
// Implementors must be Send + Sync + 'static (cached in registry, shared across threads)
pub trait Validator: Send + Sync + 'static {
    fn validate(&self, path: &Path, content: &str, config: &LintConfig) -> Vec<Diagnostic>;
    fn name(&self) -> &'static str { /* default: short type name */ }
    fn metadata(&self) -> ValidatorMetadata { /* default: empty rule_ids */ }
}

// Plugin architecture for extensibility
pub trait ValidatorProvider: Send + Sync {
    fn name(&self) -> &str { /* default: short type name */ }
    fn validators(&self) -> Vec<(FileType, ValidatorFactory)>;
    fn named_validators(&self) -> Vec<(FileType, Option<&'static str>, ValidatorFactory)> { /* default: wraps validators() with None names */ }
}

// Registry with builder pattern and runtime filtering
// Stores cached Box<dyn Validator> instances; no per-file re-instantiation
pub struct ValidatorRegistry { /* ... */ }

impl ValidatorRegistry {
    pub fn builder() -> ValidatorRegistryBuilder;
    pub fn with_defaults() -> Self;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agent-sh/agnix](https://github.com/agent-sh/agnix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
