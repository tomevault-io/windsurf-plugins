---
trigger: always_on
description: nav is a minimalist AI coding agent. By default it uses a **hashline-based editing system** for precise code modifications: it references lines by `LINE:HASH` anchors from read output, reducing edit conflicts when files change. Optional config **`editMode`: `"searchReplace"`** switches to plain-text reads and a traditional `old_string` / `new_string` edit tool instead (no hashlines).
---

# nav - Agent Guidelines

## Project Overview

nav is a minimalist AI coding agent. By default it uses a **hashline-based editing system** for precise code modifications: it references lines by `LINE:HASH` anchors from read output, reducing edit conflicts when files change. Optional config **`editMode`: `"searchReplace"`** switches to plain-text reads and a traditional `old_string` / `new_string` edit tool instead (no hashlines).

**Built for [Bun](https://bun.sh)** — leverages Bun's native APIs (`Bun.file()`, `Bun.spawn()`, `Bun.hash.xxHash32()`) for optimal performance.

**Core capabilities:**
- Read, edit, and write files with hash-based line tracking
- Execute shell commands (with optional macOS sandboxing)
- Support multiple LLM providers (OpenAI, Anthropic, Google, Ollama)
- Auto-detect context windows and perform handovers when approaching limits
- Optional **subagents** (`.nav/subagents/*.md`) and per-session **tool allowlists** (`tools` / `subagent.tools` in `nav.config.json`)
- Custom slash commands and project-specific instructions

**Target users:** Developers who want a fast, minimal coding assistant that can navigate codebases, make precise edits, and execute tasks without reproducing large code blocks.

## Project Structure

```
src/
  tools/           # Core editing tools (read, edit, write, shell, shell-status) + subagent tool
    read.ts        # Reads files with hashline format (LINE:HASH|content)
    edit.ts        # Edits files using LINE:HASH anchors
    write.ts       # Creates new files
    shell.ts       # Executes shell commands, handles backgrounding
    shell-status.ts # Monitors background processes
    index.ts       # Exports all tools

  subagents.ts     # Load .nav/subagents/*.md definitions (frontmatter + body)
  tool-names.ts    # Canonical tool name set for config allowlists

  agent.ts         # Main agent loop - handles tool calls and conversation flow
  llm.ts           # LLM provider abstraction (OpenAI, Anthropic, Google, Ollama)
  config.ts        # Configuration loading (CLI, env vars, config files)
  hashline.ts      # Hashline format implementation (LINE:HASH generation/parsing)
  diff.ts          # Diff generation for verbose mode
  prompt.ts        # System prompt and input handling
  commands.ts      # Built-in slash commands (/clear, /model, /handover, /tasks, etc.)
  custom-commands.ts # User-defined slash commands from .nav/commands/*.md
  skills.ts        # Agent skills loaded from SKILL.md files
  skill-watcher.ts # Watches skill directories for changes, triggers reload
  create-skill.ts  # /create-skill command prompt builder
  create-subagent.ts # /create-subagent command prompt builder
  tasks.ts         # Task management — persistent task list in .nav/tasks.json
 plans.ts         # Plan management — persistent plan store in .nav/plans.json
  logger.ts        # JSONL session logging to .nav/logs/
  process-manager.ts # Background process tracking for shell commands
  init.ts          # /init command — generates AGENTS.md from project context
  index.ts         # Entry point
  
  sandbox.ts       # macOS Seatbelt sandboxing - re-execs nav with filesystem restrictions
  theme.ts         # Color themes (nordic/classic) for terminal output
  tree.ts          # Project file tree generator with smart compaction
  tui.ts           # Terminal UI with readline, input queuing, and streaming support

scripts/
  build.ts         # Build standalone binaries for all platforms using Bun's --compile

sandbox/
  nav-permissive.sb # macOS Seatbelt profile for sandboxing

.github/workflows/
  release.yml      # GitHub Actions workflow for automated binary builds and releases
```

## Commands

### Development
```bash
# Install dependencies
bun install

# Type checking
bunx tsc --noEmit

# Run directly (no build needed with Bun)
bun run src/index.ts

# Development with watch mode
bun run --watch src/index.ts

# Link globally for local testing
bun link

# Build standalone binaries
bun run build                    # All platforms
bun run build:darwin-arm64       # Current platform (example)
```

### Testing
Test manually by running `nav` in a test directory:
```bash
nav "describe this codebase"
nav -v "make a small change to test.ts"
```

### Publishing
```bash
# Create a new release (builds binaries for all platforms via GitHub Actions)
npm version patch|minor|major
git push origin main --tags

# Or publish source to npm (requires Bun to be installed by users)
npm publish
```

See `RELEASE.md` for the complete release process.

Package name is `nav-agent` on npm, but the command is `nav`.

## Conventions

### Code Style
- **TypeScript strict mode** - All types must be explicit
- **ESM modules** - Use `import`/`export`
- **Bun runtime** - Uses Bun-specific APIs for performance
- **Minimal dependencies** - Only essential packages (LLM SDKs, no frameworks)
- **No external UI libraries** - Terminal output uses ANSI codes directly


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sandst1/nav](https://github.com/sandst1/nav) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
