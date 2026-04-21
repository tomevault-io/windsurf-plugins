---
trigger: always_on
description: Comprehensive coding guidelines and instructions for hve-core
---


# General Instructions

Items in the Highest Priority Rules section from attached instructions files override any conflicting guidance.

<!-- <highest-priority-rules> -->
## Priority Rules

* Conventions and styling from the codebase take precedence for all changes.
* Instructions files not already attached are read before deciding on edits.
* Breaking changes are acceptable.
* Backward-compatibility layers or legacy support are added only when explicitly requested.
* Tests, scripts, and one-off markdown docs are created or modified only when explicitly requested.

Rules for comments:

* Remain brief and factual, describing behavior, intent, invariants, and edge cases.
* Thought processes, step-by-step reasoning, and narrative comments do not appear in code.
* Comments that contradict current behavior are removed or updated.
* Temporal markers (phase references, dates, task IDs) are removed from code files during any edit.

Rules for fixing errors:

* Proactively fix any problem encountered while working in the codebase, even when unrelated to the original request.
* Root-cause fixes are preferred over symptom-only patches.
* Further investigation of the codebase or through tools is always allowed.
<!-- </highest-priority-rules> -->

<!-- <project-structure> -->
## Project Structure

This repository contains documentation, scripts, and tooling for the HVE (Hyper Velocity Engineering) Core project.

### Directory Organization

The project is organized into these main areas:

* Documentation (`docs/`) - Getting started guides, templates, RPI workflow documentation, and contribution guidelines.
* Scripts (`scripts/`) - Automation for linting, security validation, extension packaging, and development tools.
* Skills (`.github/skills/{collection-id}/`) - Self-contained skill packages, by convention organized by collection.
* Extension (`extension/`) - VS Code extension source and packaging.
* GitHub Configuration (`.github/`) - Workflows, instructions, prompts, agents, and issue templates, typically organized into `{collection-id}` subdirectories.
* Collections (`collections/`) - YAML and markdown manifests defining bundled sets of agents, prompts, instructions, and skills.
* Logs (`logs/`) - Output from validation and analysis scripts.

### Scripts Organization

Scripts are organized by function:

* Collections (`scripts/collections/`) - Collection validation and shared helper modules.
* Extension (`scripts/extension/`) - Extension packaging and preparation.
* Linting (`scripts/linting/`) - Markdown validation, link checking, frontmatter validation, and PowerShell analysis.
* Security (`scripts/security/`) - Dependency pinning validation, SHA staleness checks, and action version consistency.
* Library (`scripts/lib/`) - Shared utilities such as verified downloads.
* Plugins (`scripts/plugins/`) - Plugin generation and marketplace validation.

### Skills Organization

By convention, skills are self-contained packages organized under `.github/skills/{collection-id}/{skill-name}/`. Each skill folder contains a `SKILL.md` file with domain-specific instructions, and may include other markdown files that are referenced by `SKILL.md` along with `scripts/`, `references/`, `assets/`, or other subdirectories.

### Documentation Structure

* HVE Guide (`docs/hve-guide/`) - Project lifecycle stages and role-specific guides.
  * Lifecycle (`docs/hve-guide/lifecycle/`) - AI-assisted project lifecycle stage documentation.
  * Roles (`docs/hve-guide/roles/`) - Role-specific guides for engineers, leads, architects, and other contributors.
* Getting Started (`docs/getting-started/`) - Installation and first workflow guides with multiple setup methods.
* RPI (`docs/rpi/`) - Task researcher, planner, and implementor workflow documentation.
* Contributing (`docs/contributing/`) - Guidelines for instructions, prompts, agents, and AI artifacts.
* Templates (`docs/templates/`) - Templates for custom agents, instructions, and prompts.

### Documentation Templates

Templates for agent and prompt outputs are stored in `docs/templates/`:

* `docs/templates/full-review-output-format.md` - Code review full output format.
* `docs/templates/standards-review-output-format.md` - Standards review output format.
* `docs/templates/engineering-fundamentals.md` - Engineering fundamentals reference.
* `docs/templates/brd-template.md` - Business requirements document template.
* `docs/templates/user-journey-template.md` - User journey template.
* `docs/templates/adr-template-solutions.md` - Architecture decision record template.
* `docs/templates/rca-template.md` - Root cause analysis template.

### Copilot Tracking

The `.copilot-tracking/` directory (gitignored) contains AI-assisted workflow artifacts:

* Work Items (`.copilot-tracking/workitems/`) - ADO work item discovery and planning.
* Pull Requests (`.copilot-tracking/pr/`) - PR reference generation, handoff, and review tracking.
* Changes (`.copilot-tracking/changes/`) - Change tracking and implementation logs.
* Plans (`.copilot-tracking/plans/`) - Task implementation plans and planning logs.
* Details (`.copilot-tracking/details/`) - Task plan implementation details.
* Research (`.copilot-tracking/research/`) - Technical research findings and subagent research outputs.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/hve-core](https://github.com/microsoft/hve-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
