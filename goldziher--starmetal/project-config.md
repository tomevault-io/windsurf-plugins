---
trigger: always_on
description: Provides safe CRUD tools for reading and modifying .ai-rulez/ content
---

<!--
🤖 AI-RULEZ :: GENERATED FILE — DO NOT EDIT DIRECTLY
Project: starmetal
Generated: 2026-07-08 15:53:42
Source: .ai-rulez/config.toml
Target: AGENTS.md
Content: rules=38, sections=0, agents=9

WHAT IS AI-RULEZ
AI-Rulez is a directory-based AI governance tool. All configuration lives in
the .ai-rulez/ directory. This file is auto-generated from source files.

.AI-RULEZ FOLDER ORGANIZATION
Root content (always included):
  .ai-rulez/config.toml    Main configuration (presets, profiles)
  .ai-rulez/rules/         Mandatory rules for AI assistants
  .ai-rulez/context/       Reference documentation
  .ai-rulez/skills/        Specialized AI prompts
  .ai-rulez/agents/        Agent definitions

Domain content (profile-specific):
  .ai-rulez/domains/{name}/rules/    Domain-specific rules
  .ai-rulez/domains/{name}/context/  Domain-specific documentation
  .ai-rulez/domains/{name}/skills/   Domain-specific AI prompts

Profiles in config.toml control which domains are included.

INSTRUCTIONS FOR AI AGENTS
1. NEVER edit this file (AGENTS.md) - it is auto-generated

2. ALWAYS edit files in .ai-rulez/ instead:
   - Add/modify rules: .ai-rulez/rules/*.md
   - Add/modify context: .ai-rulez/context/*.md
   - Update config: .ai-rulez/config.toml
   - Domain-specific: .ai-rulez/domains/{name}/rules/*.md

3. PREFER using the MCP Server (if available):
   Command: npx -y ai-rulez@latest mcp
   Provides safe CRUD tools for reading and modifying .ai-rulez/ content

4. After making changes: ai-rulez generate

5. Complete workflow:
   a. Edit source files in .ai-rulez/
   b. Run: ai-rulez generate
   c. Commit both .ai-rulez/ and generated files

Documentation: https://github.com/Goldziher/ai-rulez
Content-Hash: blake3:9349c5bf6a9ec6cf1b14628c434e36c510981fc6d4d485c3ef77c58c931d4495
Source-Hash: blake3:e00328718fdf9d52bee0235cd24c90a7a87623124516cc3c56678f1fbd117d4b
-->

# starmetal

Multi-language high-performance self-hosted package registry and registry proxy

## Rules

### agent-workflow

**Priority:** high

Prefer subagents for non-trivial work — implementation, research, file exploration. Parallelize aggressively — launch independent subagents in a single message. Always critically review subagent output — check actual file changes, verify correctness, fix issues before reporting done. Never trust subagent summaries at face value; the summary describes intent, not necessarily what happened. Work in iterations: delegate → critically review → fix → verify. Run tests after every change — never assume code works without verification.

### anti-patterns

**Priority:** high

No magic numbers — use named constants. No global state — use dependency injection. No inheritance for code reuse — prefer composition. No bare exception handlers — catch specific types. No mocking internal services — use real objects for integration tests. No blocking I/O in async code paths — keep async paths fully async.

### atomic-commits

**Priority:** high

Each commit represents one logical change. Don't mix unrelated changes. Use conventional commits format (`feat:`, `fix:`, `chore:`, `refactor:`, `docs:`, `test:`). Keep commits small and focused for easier review and bisection.

### avoid-duplication

**Priority:** medium

Extract shared logic after the third repetition, not before. Three similar lines of code are better than a premature abstraction. When extracting, ensure the shared code has a single reason to change — if two callers would evolve the logic differently, keep them separate. Premature abstraction creates worse coupling than duplication.

### basemind-mcp

**Priority:** high

- Keep Basemind configured as an ai-rulez plugin through `[[plugins]]`; do not add Basemind as a raw `mcp_server`.
- Configure the ai-rulez MCP server with `npx -y ai-rulez@latest mcp` so agents can manage `.ai-rulez/` safely.
- Edit ai-rulez source files first, then regenerate outputs with `npx -y ai-rulez@latest generate --gitignore`.
- When Basemind MCP tools are available, prefer them for code navigation and repository context before falling back to shell tools:
  `outline`, `search_symbols`, `find_references`, `find_callers`, and `workspace_grep` for code search;
  `recent_changes`, `blame_file`, `blame_symbol`, `diff_file`, `diff_outline`, and `commits_touching` for git history;
  `search_documents`, `web_scrape`, `web_crawl`, and `web_map` for docs and web retrieval.
- Use shell, `rg`, and raw `git` when Basemind is unavailable, when exact raw output is required, or when a task runner/check is the source of truth.

### batch-operations

**Priority:** medium

Group related file reads and writes into single operations. Combine independent tool calls in parallel rather than sequentially. When making multiple edits to the same file, batch them into one edit operation. Prefer multi-file search tools over individual file reads when exploring.

### branch-hygiene

**Priority:** medium

Use descriptive branch names. Keep branches short-lived. Delete merged branches. Rebase or merge from main regularly to avoid drift.

### commit-messages

**Priority:** high


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Goldziher/starmetal](https://github.com/Goldziher/starmetal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-02 -->
