---
trigger: always_on
description: Wingspan is a collection of AI-assisted engineering tools — skills, agents, and hooks — for the software development lifecycle.
---

# VGV Wingspan

Wingspan is a collection of AI-assisted engineering tools — skills, agents, and hooks — for the software development lifecycle.

## Repository Structure

```text
.claude-plugin/
  plugin.json          # Plugin manifest (name, version, keywords)
.mcp.json              # MCP server configuration (context7)
AGENTS.md              # This file — portable, tech-agnostic conventions
CLAUDE.md              # Claude Code entry point: imports AGENTS.md, adds Claude-specific Hooks
README.md              # Human-facing overview and install instructions
CONTRIBUTING.md        # Contributor guide (adding skills, hooks, commit format)
skills_lint.yaml       # skills_lint rule config used by the CI skills lint workflow
config/
  cspell.json          # Spell-check dictionary and settings
  custom.markdownlint.jsonc  # Markdown lint rule overrides
agents/                # Subagent definitions, grouped by role
  analysis/
    plan-splitting-agent.md       # Flags oversized plans to split across PRs
    user-flow-analysis-agent.md   # Analyzes specs for flow gaps and edge cases
  codebase-review/
    codebase-review-agent.md         # Reviews structure, conventions, pattern consistency
    code-simplicity-review-agent.md  # Flags YAGNI violations and over-engineering
    vgv-review-agent.md              # Reviews against VGV engineering standards
  quality-review/
    architecture-review-agent.md  # Validates layer separation and dependency direction
    pr-readiness-review-agent.md  # Checks formatting, static analysis, commit hygiene
    test-quality-review-agent.md  # Verifies test coverage and quality
  research/
    best-practices-research-agent.md  # Synthesizes best practices for the stack
    official-docs-research-agent.md   # Gathers official framework/library docs
hooks/
  hooks.json                    # Hook definitions (PreToolUse)
  recommend-plugins.sh          # Detects project type, recommends companion plugins
  test_recommend_plugins.sh     # Tests for the recommendation hook
  recommendations/
    vgv-ai-flutter-plugin.json  # Detection rule + recommendation for the Flutter plugin
skills/                # User-invocable and supporting skills (one dir per skill)
  brainstorm/SKILL.md            # Explore requirements and approaches
  plan/SKILL.md                  # Turn a brainstorm into an implementation plan
  build/SKILL.md                 # Execute a plan: implement, review, ship
  review/SKILL.md                # Run quality-review agents on demand
  hotfix/SKILL.md                # Fast path for emergency fixes
  debrief/SKILL.md               # Post-incident analysis document
  create/SKILL.md                # Scaffold a new project via companion plugins
  create-pr/SKILL.md             # Generate a PR title and description
  plan-technical-review/SKILL.md # Review externally-authored plans
  refine-approach/SKILL.md       # Iteratively improve a document
  rebase/SKILL.md                # Sync a feature branch with its base
  elements-of-style/SKILL.md     # Apply Strunk's Elements of Style to prose
  shared/              # Shared references and scripts used across skills
    references/        # Plan templates, review procedures, handoff steps
    scripts/           # detect-base-branch.sh, detect-review-scope.sh
```

Each skill directory may also carry a `references/` folder (deeper procedure docs) and a `scripts/` folder (helper shell scripts).

## Philosophy

Apply VGV's best practices and standards for scalable software to AI-assisted workflows. Each step of the development cycle should make subsequent steps clearer and closer to the user's intent. Build the right thing, build the thing right.

## Tech-Agnostic by Design

Wingspan handles the software development lifecycle — brainstorming, planning, building, and quality review. It does not enforce or assume any specific programming language, framework, or toolchain.

Technology-specific concerns (linting, formatting, scaffolding, framework conventions) belong in companion plugins. In Claude Code, Wingspan's recommendation hook detects project types and suggests the appropriate companion plugin automatically.

## Workflow

> The `/name` forms below are the canonical skill names. Invocation syntax varies by harness — Claude Code uses `/name`; other harnesses may differ.

The plugin supports three sequential phases:

1. **`/brainstorm`** — Explore requirements and approaches through collaborative dialogue. Produces a brainstorm document.
2. **`/plan`** — Transform brainstorm output into an actionable implementation plan. Includes codebase review, optional external research, flow analysis, and a mandatory quality review of the draft. Splits large plans into phases so `/build` executes one phase per context window.
3. **`/build`** — Execute implementation plans: implement one phase per context window (implement → validate → commit → checkpoint → clear), run quality review, and ship a pull request. Committing and pushing follow the user's chosen autonomy — per-phase auto-commits or full manual control — decided up front or from a saved preference.

Standalone Skills:

- **`/review`** — Run quality review agents on demand, independent of the build workflow.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [VeryGoodOpenSource/vgv-wingspan](https://github.com/VeryGoodOpenSource/vgv-wingspan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
