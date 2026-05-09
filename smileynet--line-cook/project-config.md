---
trigger: always_on
description: > See [README.md](README.md) for philosophy, influences, and user documentation.
---

# Line Cook

> See [README.md](README.md) for philosophy, influences, and user documentation.

Technical details for working on Line Cook itself.

## Kitchen Theme

**Goal**: Low cognitive load with light theming for disambiguation.

### Why Kitchen Terms?

Command names use creative kitchen terms to help users distinguish Line Cook commands from other plugins. `/line:mise` won't be confused with another tool's `/plan` command.

### When to Use Theme vs. Direct Terms

| Context | Use | Example |
|---------|-----|---------|
| **Command names** | Themed (disambiguation) | `/line:mise`, `/line:plate` |
| **Command descriptions** | Direct (clarity) | "Create work breakdown before starting" |
| **Tutorials & casual docs** | Light flavor text OK | "Check the ticket rail (ready tasks)" |
| **Technical docs** | Direct terms only | "ready tasks", not just "ticket rail" |

The theme is a **recognition aid**, not a **learning barrier**. Always include the actual meaning.

### Terminology Glossary

| Theme Term | Actual Meaning | Use in... |
|------------|----------------|-----------|
| Ticket Rail | Ready task list | Flavor text only |
| House Rules | Project config (CLAUDE.md) | Flavor text only |
| Plate Check | Quality verification | Flavor text only |
| Shift Notes | Commit message | Flavor text only |
| House Book | Git remote | Flavor text only |
| Tasting Dish | Minimal e2e proof | Flavor text only |
| Spice | Domain knowledge addon plugin | Plugin type |
| ORDER_UP | Task ready for review | Internal signal |
| GOOD_TO_GO | Review passed | Internal signal |

## Overview

```
/mise ─────────────────────────────────→ /prep → /cook → /serve → /tidy → /plate → /close-service
  │                                         ↓       ↓       ↓        ↓        ↓            ↓
  ├─ /brainstorm → brainstorm.md          sync   execute  review   commit  validate    validate
  ├─ /sample → walkthrough.md                                              (feature)     (epic)
  ├─ /scope → menu-plan.yaml
  └─ /finalize → beads + specs
```

Or use `/run` for the full execution cycle, `/mise` for the full planning cycle.

## Commands

| Command | Purpose |
|---------|---------|
| `/init` | Verify setup |
| `/getting-started` | Learn the workflow |
| `/onboarding` | Interactive walkthrough |
| `/whats-new` | Browse recent changes |
| `/doctor` | Diagnose issues |
| `/mise` | Create work breakdown (orchestrates brainstorm→sample→scope→finalize) |
| `/brainstorm` | Explore problem space (divergent thinking) |
| `/sample` | Walk through user experience (experiential thinking) |
| `/scope` | Create structured work breakdown (convergent thinking) |
| `/finalize` | Convert plan to beads and create test specs |
| `/plan-audit` | Audit bead structure, quality, and hygiene |
| `/prep` | Sync git, show ready tasks |
| `/cook` | Execute task with TDD cycle |
| `/serve` | Review code changes |
| `/tidy` | Commit and push changes |
| `/plate` | Validate completed feature |
| `/close-service` | Validate completed epic |
| `/run` | Run full workflow cycle |
| `/loop` | Autonomous loop management |
| `/architecture-audit` | Analyze codebase structure and code smells |
| `/decision` | Record, list, or supersede architecture decisions |
| `/respond` | Post templated follow-up on a GitHub issue as `line-sous-chef[bot]` |
| `/inspect-issues` | Review open PRs and standalone issues |
| `/help` | Contextual help for Line Cook commands |

## Platform Command Naming

Claude Code and OpenCode use different command naming conventions:

| Platform | Syntax | Example |
|----------|--------|---------|
| Claude Code | `namespace:command` | `/line:prep` |
| OpenCode | `namespace-command` | `/line-prep` |

- **Claude Code**: Uses `plugin.json` namespace + flat filename → `line:prep`
- **OpenCode**: Uses file path as command name → `line-prep`

## Platform Architecture

**Line Cook supports THREE separate AI coding platforms:**

| Platform | Type | CLI Tool | Product |
|----------|------|----------|----------|
| Claude Code | Plugin system | claude CLI | Anthropic product |
| OpenCode | TUI application | opencode CLI | anomalyco/opencode |
| Kiro CLI | CLI application | kiro-cli | kiro.dev |

Line Cook provides separate implementations for each:
  - `plugins/claude-code/` = Claude Code plugin
  - `plugins/opencode/` = OpenCode plugin
  - `plugins/kiro/` = Kiro CLI agents

## Dependencies

- **beads** (`bd`) - Git-native issue tracking for multi-session work
- **Claude Code**, **OpenCode**, or **Kiro** - AI coding assistant

## Spice Rack (Domain Addon Plugins)

**Spices** are skills-only plugins that add domain-specific knowledge to Line Cook's planning workflow. They load automatically during `/mise` (brainstorm, scope, finalize) when Claude detects relevant project context.

### How Spices Work

- Each spice is a separate GitHub repo with `.claude-plugin/plugin.json` + `skills/` directory
- Listed in line-cook's `marketplace.json` via GitHub source reference
- Users install alongside the core plugin: `/plugin install game-spice@line-cook`
- Skills activate by keyword matching in SKILL.md frontmatter descriptions
- No commands, agents, or scripts — pure knowledge that enhances existing workflow

### Available Spices


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [smileynet/line-cook](https://github.com/smileynet/line-cook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
