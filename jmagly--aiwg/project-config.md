---
trigger: always_on
description: Framework for improving AI-generated content quality with voice profiles, validation tools, and specialized agents.
---

# AIWG


@AIWG.md

Framework for improving AI-generated content quality with voice profiles, validation tools, and specialized agents.

## Quick Start

```bash
# Validate content for voice consistency
/writing-validator "path/to/content.md"

# SDLC workflows (use natural language)
"transition to elaboration"
"run security review"
"where are we?"

# Agent deployment
aiwg use sdlc
```

## Repository Structure

```
agentic/code/
├── frameworks/
│   ├── sdlc-complete/        # Complete SDLC coverage
│   ├── forensics-complete/   # Digital forensics & incident response
│   ├── security-engineering/ # Applied security (crypto, chain-of-trust, factors, degraded modes, supply-chain, physical threats)
│   ├── media-marketing-kit/  # Full marketing operations
│   ├── media-curator/        # Media archive management
│   └── research-complete/    # Research workflow automation
├── addons/
│   └── voice-framework/     # Voice profiles
└── agents/                  # Writing quality agents

src/                         # CLI and MCP server implementation
├── extensions/              # Unified extension system
│   ├── types.ts            # Extension type definitions
│   ├── commands/           # Command extension definitions
│   └── registry.ts         # Extension registry
test/                        # Test suites and fixtures
tools/                       # Build and deployment scripts
docs/                        # Documentation
├── cli-reference.md         # All 50 CLI commands
├── extensions/              # Extension system docs
│   ├── overview.md
│   ├── creating-extensions.md
│   └── extension-types.md
```

## Context Loading Strategy

**Automatic (via path-scoped rules)**:

| Working in... | Rules loaded |
|---------------|--------------|
| `.aiwg/**` | SDLC orchestration |
| `**/*.md` | Voice framework |
| `src/**`, `test/**` | Development conventions |
| `.claude/agents/**` | Agent deployment |

**On-demand (via @-mentions)**:

Use `@path/to/file.md` in your message to load specific documentation:
- `@agentic/code/frameworks/sdlc-complete/docs/orchestrator-architecture.md` - Full orchestration details
- `@agentic/code/frameworks/sdlc-complete/agents/manifest.json` - SDLC agent listing
- `@.aiwg/requirements/UC-*.md` - Specific requirements
- `@docs/cli-reference.md` - Complete CLI command reference
- `@docs/extensions/overview.md` - Extension system architecture

## Multi-Platform Support

All 10 providers receive all 4 artifact types (agents, commands, skills, rules). OpenClaw additionally receives behaviors. Support level indicates how the platform discovers artifacts: **native** (auto-discovered), **conventional** (AIWG directory), **aggregated** (single-file + discrete).

| Platform | Agents | Commands | Skills | Rules | Command |
|----------|--------|----------|--------|-------|---------|
| Claude Code | `.claude/agents/` | `.claude/commands/` | `.claude/skills/` | `.claude/rules/` | `aiwg use sdlc` |
| OpenAI/Codex | `.codex/agents/` | `~/.codex/prompts/` | `~/.codex/skills/` | `.codex/rules/` | `aiwg use sdlc --provider codex` |
| GitHub Copilot | `.github/agents/` | `.github/prompts/` | `.github/prompts/` | `.github/instructions/` | `aiwg use sdlc --provider copilot` |
| Factory AI | `.factory/droids/` | `.factory/commands/` | `.factory/skills/` | `.factory/rules/` | `aiwg use sdlc --provider factory` |
| Cursor | `.cursor/agents/` | `.cursor/commands/` | `.cursor/skills/` | `.cursor/rules/` | `aiwg use sdlc --provider cursor` |
| OpenCode | `.opencode/agent/` | `.opencode/command/` | `.opencode/skill/` | `.opencode/rule/` | `aiwg use sdlc --provider opencode` |
| Warp Terminal | `.warp/agents/` + WARP.md | `.warp/commands/` | `.warp/skills/` | `.warp/rules/` | `aiwg use sdlc --provider warp` |
| Windsurf | AGENTS.md | `.windsurf/workflows/` | `.windsurf/skills/` | `.windsurf/rules/` | `aiwg use sdlc --provider windsurf` |
| OpenClaw | `~/.openclaw/agents/` | `~/.openclaw/commands/` | `~/.openclaw/skills/` | `~/.openclaw/rules/` | `aiwg use sdlc --provider openclaw` |
| Hermes | AGENTS.md | — | `~/.hermes/skills/` | — | `aiwg use sdlc --provider hermes` |

**Special cases:**
- **Codex**: Commands and skills deploy to home directory (`~/.codex/prompts/`, `~/.codex/skills/`) for user-level availability across all projects
- **Copilot**: Agents use `.agent.md` format (Markdown + YAML frontmatter). Commands deploy as prompt files (`.github/prompts/`). Rules deploy as path-scoped instructions (`.github/instructions/`)
- **Warp**: Agents and commands are also aggregated into `WARP.md` for single-file context loading
- **Windsurf**: Agents are aggregated into `AGENTS.md` at project root
- **OpenClaw**: All artifacts deploy to home directory (`~/.openclaw/`). First provider to support behaviors (`~/.openclaw/behaviors/`)

## Writing Principles

1. **Apply appropriate voice** - Match audience (technical-authority, friendly-explainer, executive-brief, casual-conversational)
2. **Maintain sophistication** - Preserve domain-appropriate vocabulary
3. **Include authenticity markers** - Add opinions, acknowledge trade-offs
4. **Vary structure** - Mix sentence lengths and styles
5. **Be specific** - Exact metrics, concrete examples

## Installation


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jmagly/aiwg](https://github.com/jmagly/aiwg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
