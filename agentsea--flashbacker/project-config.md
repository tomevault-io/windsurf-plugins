---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

```bash
# Build and development
npm run build              # Compile TypeScript to lib/
npm run dev               # Watch mode compilation
npm link                  # Link for global CLI testing

# Testing (Most tests deleted as obsolete)
npm test                  # Build + run remaining tests (main command to use)
npm run test:unit         # Unit tests (if any remain)
npm run test:coverage     # Coverage report

# Code quality (Run before commits)
npm run lint              # ESLint TypeScript files
npm run lint:fix          # Auto-fix ESLint issues (use this first)
npm run format            # Prettier formatting

# Prerequisites checking
npm run setup:prereqs     # Check Node.js/npm compatibility before development

# Development workflow
npm run build && npm link  # After changes, rebuild and relink globally
flashback --version        # Verify installation (should show 2.2.6)
```

**Development Pattern**: Always run `npm run lint:fix` before `npm run build` to auto-fix style issues.

## Architecture Overview

### Hybrid AI+Computer Operations Pattern (CRITICAL)

This codebase implements a breakthrough pattern that separates **programmatic operations** (handled by CLI) from **intelligence operations** (handled by Claude/subagents):

**Programmatic Operations → `flashback` CLI:**
- File management (archive, prune, directory creation)  
- Data extraction (JSONL parsing, git status, session detection)
- System operations (file paths, formatting, structural tasks)
- **Why CLI**: Claude does these differently each time, creating inconsistency

**Intelligence Operations → Claude Subagent Prompts:**
- Analysis (code review, architectural assessment, security)
- Content creation (session summaries, documentation, recommendations)  
- Decision making (prioritization, strategic planning, creative work)
- **Why prompts**: Require domain expertise, context understanding, intelligence

**The Pattern:**
```bash
/fb:persona architect "review auth system"
# 1. Slash command → Claude runs: flashback persona architect --context "review auth system"  
# 2. CLI programmatically gathers context (consistent every time)
# 3. Claude spawns subagent with gathered context for intelligent analysis
```

### Dual-Layer AI System (CRITICAL - v2.2.x)

The system provides **two complementary ways** to access 12 AI specialists:

**Layer 1: Personas (Immediate Analysis)**
- **Usage**: `/fb:persona {name} "request"`
- **Purpose**: Fast, in-conversation analysis using specialist templates
- **Implementation**: Direct template reading from `.claude/flashback/personas/{name}.md`
- **Benefits**: Immediate specialist expertise without leaving current conversation

**Layer 2: Agents (Project-Aware Analysis)**  
- **Usage**: `@agent-{name} "request"` (Claude Code native agents)
- **Purpose**: Deep analysis in dedicated subagent conversations with full project context
- **Context Bundle**: Agents auto-gather REMEMBER.md, WORKING_PLAN.md, conversation history
- **Implementation**: Claude Code agents with context gathering via `flashback agent --context`
- **Benefits**: Project-aware analysis with complete understanding of codebase patterns

**20 Available Specialists**: architect, security, refactorer, performance, frontend, backend, devops, qa, analyzer, mentor, product, code-critic, debate-moderator, debt-hunter, database-architect, api-designer, data-engineer, platform-engineer, docker-master, john-carmack, fix-master

**IMPORTANT**: The templates/ directory is currently empty in this repository. Templates are distributed at runtime via dynamic scanning from bundled npm package. Never hardcode template content - always use `getRequiredFlashbackDirectories()` from `src/utils/file-utils.ts` for dynamic template discovery.

### Template-Driven Architecture (CRITICAL)

**NEVER hardcode paths or fallback content**. The entire system is template-driven:

- **Init system**: ALWAYS uses bundles from `./templates` to distributed files
- **Dynamic scanning**: All structure derived from template bundle at runtime via `getRequiredFlashbackDirectories()`
- **Zero exceptions**: Template-first approach prevents drift and ensures consistency
- **Variable replacement**: Templates use `{{PROJECT_NAME}}`, `{{VERSION}}`, `{{TIMESTAMP}}`
- **Agent Templates**: Must be `.md` files with standard YAML frontmatter (name, description only)

### Core System Architecture

**CLI Entry Point:** `src/cli.ts`
- Commander.js-based CLI with subcommands
- Each command maps to `src/commands/*.ts`
- Version dynamically read from `package.json`

**Working Commands (Post-Cleanup):**
- `init`: Template-driven project initialization with failsafe memory protection and MCP server integration
- `persona`: AI persona system with 17 specialists (architect, security, database-architect, api-designer, data-engineer, platform-engineer, etc.)
- `agent`: Context gathering for Claude Code agent subagents 
- `memory`: REMEMBER.md management for persistent project knowledge
- `working-plan`: Development plan tracking across sessions with AI analysis

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agentsea/flashbacker](https://github.com/agentsea/flashbacker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
