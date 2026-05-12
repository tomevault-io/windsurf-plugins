---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Purpose

claude-rio is a hook framework for Claude Code CLI that improves skill, agent, and command activation through deterministic keyword matching. It provides explicit suggestions to Claude about relevant skills/agents/commands based on user prompts, significantly increasing activation probability.

## Technology Stack

- **Languages:** Bash (shell wrappers), JavaScript/Node.js (handlers)
- **Architecture:** Two-tier system (shell preprocessing + Node.js execution)
- **Type System:** JSDoc annotations
- **Dependencies:** chalk, commander, fs-extra, ora (CLI only)
- **Runtime:** Node.js >= 18.0.0

## Project Structure

```
claude-rio/
├── cli/                           # Command-line interface
│   ├── index.js                   # Main CLI entry point
│   ├── commands/
│   │   ├── setup.js               # Install hook framework
│   │   ├── generate-matchers.js   # Generate matchers using Claude Haiku
│   │   └── remove.js              # Uninstall framework + all matchers
│   └── utils/
│       ├── claude-checker.js      # Check if Claude CLI is available
│       ├── claude-generator.js    # Generate matchers using Claude Haiku
│       ├── copy-templates.js      # Copy hooks to .claude directory
│       ├── matcher-validator.js   # Validate matcher structure and fields
│       ├── platform.js            # OS detection (Windows/Unix)
│       ├── prompt-builder.js      # Build prompts for Claude to generate matchers
│       └── skill-scanner.js       # Scan .claude for skills/agents/commands
├── hooks/                         # Hook framework (distributed to .claude)
│   ├── UserPromptSubmit/
│   │   ├── hook.sh                # Shell wrapper (finds matchers) - Unix
│   │   ├── hook.ps1               # Shell wrapper (finds matchers) - Windows
│   │   ├── handler.cjs            # Node.js handler (executes matchers)
│   │   ├── types.cjs              # JSDoc type definitions
│   │   ├── validations.cjs        # Payload and result validation
│   │   └── formatter.cjs          # Output formatting
│   └── utils/                     # Shared utilities for hooks
│       ├── index.cjs              # Central export point
│       ├── io.cjs                 # stdin/stdout operations
│       ├── logger.cjs             # File-based logging
│       ├── result.cjs             # Result-style error handling
│       ├── transcript.cjs         # Conversation history utilities
│       └── validations.cjs        # Validation helpers
├── matchers/                      # Universal matcher template
│   └── UserPromptSubmit.rio.matcher.cjs  # Template used by setup command
├── examples/                      # Example matcher patterns
│   ├── keyword/UserPromptSubmit.rio.matcher.cjs
│   ├── typo-tolerant/UserPromptSubmit.rio.matcher.cjs
│   ├── file-based/UserPromptSubmit.rio.matcher.cjs
│   ├── history-aware/UserPromptSubmit.rio.matcher.cjs
│   └── config-based/UserPromptSubmit.rio.matcher.cjs
├── tests/                         # Test suites
│   ├── docker/                    # Docker-based integration tests
│   └── haiku-visual/              # Haiku generation visual tests
├── package.json                   # Package metadata
├── CHANGELOG.md                   # Version history
├── README.md                      # User-facing documentation
└── CLAUDE.md                      # This file
```

## Key Concepts

### Two-Tier Architecture

The system uses a **shell-first approach** to minimize overhead:

1. **Shell wrapper (`hook.sh`)** runs first
   - Uses `find` command to locate matcher files
   - Searches skills: `.claude/skills/*/rio/UserPromptSubmit.matcher.cjs`
   - Searches agents: `.claude/agents/*.rio.matcher.cjs`
   - Searches commands: `.claude/commands/*.rio.matcher.cjs`
   - If no matchers found → exits immediately (~10-20ms)
   - If matchers found → passes paths to Node.js via `MATCHER_PATHS` env var

2. **Node.js handler (`handler.cjs`)** only executes when matchers exist
   - Reads matcher paths from environment
   - Loads and executes each matcher function
   - Validates matcher results
   - Formats output as JSON for Claude Code
   - Total time: ~50-100ms

**Performance:** ~70% faster when no matchers exist (most common case).

### Matchers

Matchers are JavaScript functions that determine skill/agent/command relevance:

- **Input:** Context object with `{prompt, cwd, transcriptPath, sessionId, permissionMode, meta, transcript}`
- **Output:** `{version: "2.0", matchCount: number, type: string}`
- **Validation:** All fields are MANDATORY (no undefined/null allowed)
- **Types:** Sync or async (use async only for conversation history)
- **Scoring:** Handler calculates `score = min(matchCount, 10) / maxMatchCount`

**Required Output Fields:**
- `version`: Always "2.0"
- `matchCount`: Number of keyword matches (0+, capped at 10 for scoring)
- `type`: "skill" | "agent" | "command" - Type of matcher

**How Scoring Works:**
1. Matchers count keyword matches and return `matchCount`
2. Handler caps each `matchCount` at 10 to prevent keyword inflation
3. Handler calculates relative scores: `score = cappedCount / maxMatchCount`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alex-popov-tech/claude-rio](https://github.com/alex-popov-tech/claude-rio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
