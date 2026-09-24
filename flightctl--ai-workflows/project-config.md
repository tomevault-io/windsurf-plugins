---
trigger: always_on
description: This file provides guidance to AI coding assistants when working with this repository.
---

# AGENTS.md

This file provides guidance to AI coding assistants when working with this repository.

## Project Overview

This repository contains reusable AI coding workflows and focused skills that can be installed globally or per-project in any environment (Cursor, Claude Code, Gemini, Codex). Each package is a self-contained directory with structured markdown files that AI agents can read and execute.

**Current simple skills:**
- **gh-stack** — Manages stacked PRs with gh-stack (creation, viewing, editing, push, submit, sync, rebase, merge, checkout)
- **report-bug** — Configurable, evidence-based Jira Bug reporting with explicit confirmation

**Current workflows:**
- **ai-ready** — Codebase scanning and AGENTS.md generation (update)
- **bugfix** — Systematic bug resolution (assess, reproduce, diagnose, fix, test, review, document, pr)
- **code-review** — AI-driven code review with human-in-the-loop decisions (start, continue, clean)
- **cve-fix** — Automated CVE remediation from Jira tickets (start, scan, patch, validate, pr, backport, close | standalone: report)
- **design** — Design-and-decompose workflow (ingest, research, draft, decompose, revise, publish, respond, sync)
- **docs-writer** — Documentation creation workflow (gather, plan, draft, validate, apply, mr)
- **e2e** — Story-to-tests workflow for [QE] stories (ingest, plan, revise, code, validate, publish, respond)
- **implement** — Story-to-code workflow (ingest, plan, revise, code, validate, publish, respond)
- **kcs** — KCS Solution article workflow (gather, draft, validate, handoff)
- **prd** — Requirements-to-PRD workflow (ingest, clarify, draft, revise, publish, respond)
- **rebase-stack** — Rebase a stacked-branch chain with conflict guidance, per-branch validation, and push (start, continue, validate, push)
- **sizing** — Pre-cycle Feature sizing with T-shirt sizes and team effort breakdowns (ingest, assess, apply)
- **skill-reviewer** — Meta-workflow that audits AI skill directories
- **triage** — Bulk Jira bug triage with AI-driven categorization and HTML reports

## Architecture

### Workflow Structure

Every workflow follows this canonical structure:

```text
workflow-name/
  SKILL.md              # Entry point with YAML frontmatter (name, version, description)
  guidelines.md         # Behavioral rules: principles, hard limits, safety, quality
  README.md             # Human-readable documentation (prerequisites, artifacts, usage)
  skills/
    controller.md       # Optional discovery and ambiguous-input router
    dispatch.md         # Optional lightweight explicit-phase dispatcher
    completion.md       # Optional centralized next-step guidance
    phase-name.md       # Implementation for each phase
  commands/
    phase-name.md       # Thin wrappers that invoke a controller, dispatcher, SKILL.md, or phase
  scripts/              # Optional — deterministic operations invoked by skills
  prompts/              # Optional — prompt templates for sub-agent delegation
```

Simple skills live at `skills/{skill-name}/` with a `SKILL.md` entry point and
only the references, scripts, or assets they need.

### Simple Skill Structure

```text
skills/
  skill-name/
    SKILL.md              # Entry point with YAML frontmatter
    references/           # Optional conditional instructions and schemas
    templates/            # Optional generated-content templates
    scripts/              # Optional deterministic implementation helpers
```

Simple skills are focused capabilities, not phase-based workflows. Add only the
resources required by the skill; they do not need a controller, commands,
guidelines, README, or artifact lifecycle by default.

**Key architectural principles:**
1. **Auto-discovery**: The installer discovers top-level `*/SKILL.md` workflows and `skills/*/SKILL.md` simple skills; package names must be globally unique
2. **Progressive disclosure**: SKILL.md is thin (under 30 lines); details live in workflow guidelines/phases or a simple skill's references
3. **Relative paths**: All file references must be relative to the file's location (for symlink compatibility)
4. **Phase-based execution**: Most workflows operate through discrete phases with explicit transitions
5. **Shared resources**: Cross-cutting concerns live in `_shared/` and are referenced by relative path from workflows or simple skills
6. **Phase overrides**: Projects can override individual phases by placing a replacement skill file at `.workflows/{workflow}/skills/{phase}.md` in their repo root. The controller or lightweight dispatcher checks for this override before falling back to the built-in default. See CONTRIBUTING.md for details.

### Shared Resources (`_shared/`)

```text
_shared/
  provenance-schema.md            # Provenance contract for planning docs (footer + session log)
  content-rules.md                # Shared generated-content rules for all workflows
  review-protocol.md              # Shared code review criteria, finding format, severity definitions
  sizing-rubric.md                # Shared sizing definitions (T-shirt sizes, heuristics, team effort guidance)
  scripts/
    provenance.py                 # Capture/render CLI (used by prd and design provenance recipes)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [flightctl/ai-workflows](https://github.com/flightctl/ai-workflows) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
