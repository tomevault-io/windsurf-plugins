---
trigger: always_on
description: Agent-agnostic automated development loop that implements The Loop methodology:
---

# AGENTS.md

## Project: Alpha Loop

Agent-agnostic automated development loop that implements The Loop methodology:
**Plan (GitHub Issues) -> Build (AI Agent) -> Test -> Review -> Ship (PR)**

## Tech Stack

| Layer | Technology |
|-------|------------|
| **Runtime** | Node.js, TypeScript, ESM |
| **CLI Framework** | Commander.js |
| **AI Agents** | Any CLI agent (Codex, Codex, OpenCode) |
| **Source of Truth** | GitHub (Issues = kanban, PRs = reviews, Actions = CI) |
| **Package Manager** | pnpm |

## Commands

```bash
alpha-loop init          # Full onboarding: config, templates, vision, scan, sync
alpha-loop run           # Run the loop continuously
alpha-loop run --once    # Process one issue and exit
alpha-loop run --dry-run # Dry run (preview, no changes)
alpha-loop run --epic <N>        # Process an epic (sub-issues in checklist order, auto-verify on completion)
alpha-loop run --verify-only <N> # Run just the epic verification pass
alpha-loop scan          # Generate/refresh project context
alpha-loop plan          # Generate project scope (milestones + issues) from seed inputs
alpha-loop add           # Create a new issue from a free-form description using AI
alpha-loop triage        # Analyze and improve existing issues
alpha-loop roadmap       # Organize open issues into milestones
alpha-loop vision        # (deprecated) Use "alpha-loop plan" instead
alpha-loop auth          # Save authenticated browser state
alpha-loop resume        # Resume stranded work from crashed sessions
alpha-loop resume --issue 34     # Resume a specific issue
alpha-loop review        # Analyze learnings, propose agent/skill improvements
alpha-loop review --apply        # Apply improvements and create draft PR
alpha-loop history       # View session history
alpha-loop history <name> --qa    # Show QA checklist for session
alpha-loop history --clean        # Remove old session data
pnpm test               # Run all tests
pnpm build              # Build TypeScript to dist/
```

## Directory Structure

```
alpha-loop/
├── src/
│   ├── cli.ts                  # CLI entry point (Commander setup)
│   ├── commands/               # Subcommand handlers
│   │   ├── auth.ts             # Browser auth state management
│   │   ├── history.ts          # Session history viewer
│   │   ├── init.ts             # Config template creation
│   │   ├── resume.ts           # Resume stranded work from crashed sessions
│   │   ├── review.ts           # Self-improvement loop (learnings → proposals)
│   │   ├── run.ts              # Main loop execution
│   │   ├── scan.ts             # Project context generation
│   │   └── vision.ts           # Vision document setup
│   ├── engine/                 # Multi-agent engine
│   │   ├── agents.ts           # Agent CLI map and arg builder (Codex, codex, opencode)
│   │   └── prerequisites.ts    # System requirement checks
│   └── lib/                    # Shared libraries
│       ├── agent.ts            # Agent runner abstraction
│       ├── config.ts           # YAML config loading
│       ├── context.ts          # Project context management
│       ├── github.ts           # GitHub API (issues, PRs, labels)
│       ├── learning.ts         # Learning extraction/application
│       ├── logger.ts           # Structured logging
│       ├── pipeline.ts         # Issue processing pipeline
│       ├── preflight.ts        # Pre-run test validation
│       ├── prerequisites.ts    # Tool availability checks
│       ├── prompts.ts          # Agent prompt generation
│       ├── session.ts          # Session management
│       ├── shell.ts            # Shell execution helpers
│       ├── testing.ts          # Test runner integration
│       ├── vision.ts           # Vision document helpers
│       └── worktree.ts         # Git worktree management
├── tests/                      # Test suite (mirrors src/ structure)
├── templates/                  # DISTRIBUTION: starter files shipped with npm package
│   ├── skills/                 # Default skills installed by `alpha-loop init`
│   └── agents/                 # Default agent prompts installed by `alpha-loop init`
├── .alpha-loop.yaml            # Loop configuration (agent, model, harnesses, etc.)
├── .alpha-loop/
│   ├── templates/              # THIS REPO'S OWN skills and sub-agent definitions
│   │   ├── skills/             # Skill definitions (synced to harness-specific paths)
│   │   └── agents/             # Sub-agent prompts (implementer.md, reviewer.md)
│   ├── learnings/              # Tracked in git — team-shared knowledge
│   │   └── proposed-updates/   # Proposed improvements from `alpha-loop review`
│   └── sessions/               # Gitignored — local logs, screenshots
├── .Codex/                    # Auto-synced from .alpha-loop/templates/ (Codex)
├── .agents/                    # Auto-synced from .alpha-loop/templates/ (Codex, Cursor, etc.)
└── .codex/                     # Auto-synced from .alpha-loop/templates/ (Codex agents)
```

## Two templates/ directories — don't confuse them

This repo has TWO `templates/` directories with different purposes:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bradtaylorsf/alpha-loop](https://github.com/bradtaylorsf/alpha-loop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
