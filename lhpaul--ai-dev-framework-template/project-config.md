---
trigger: always_on
description: This is the primary AI agent guidance file for this project. It follows the [AGENTS.md](https://github.com/agentsmd/agents.md) open format and is read by all AI coding assistants (Claude Code, Cursor, Codex, Gemini CLI, etc.).
---

# AGENTS.md

This is the primary AI agent guidance file for this project. It follows the [AGENTS.md](https://github.com/agentsmd/agents.md) open format and is read by all AI coding assistants (Claude Code, Cursor, Codex, Gemini CLI, etc.).

> **Note for Claude Code users**: `CLAUDE.md` is a symlink to this file.

---

## Project Overview

> **TODO**: Fill this section via the project setup agent (`docs/workflow/setup/protocol.md`), or manually describe your project here.
>
> - What does this project do?
> - Who are the users?
> - What problem does it solve?

---

## Repository Structure

> **TODO**: Fill this section after running the project setup. Reference `docs/project/2-repo-architecture.md` for details.

---

## Key Documentation

Always refer to these docs for authoritative guidance:

| Document                                                                                                                                                       | Purpose                                                                                         |
| -------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| [`docs/project/1-business-domain.md`](docs/project/1-business-domain.md)                                                                                       | Domain entities, business rules, glossary                                                       |
| [`docs/project/2-repo-architecture.md`](docs/project/2-repo-architecture.md)                                                                                   | Repository structure, packages, apps                                                            |
| [`docs/project/3-software-architecture.md`](docs/project/3-software-architecture.md)                                                                           | Tech stack, design patterns, architecture decisions                                             |
| [`docs/project/4-database-model.md`](docs/project/4-database-model.md)                                                                                         | Data model, schema, access patterns (if applicable)                                             |
| [`docs/best-practices/1-general.md`](docs/best-practices/1-general.md)                                                                                         | General coding standards                                                                        |
| [`docs/best-practices/2-version-control.md`](docs/best-practices/2-version-control.md)                                                                         | Git conventions                                                                                 |
| [`docs/best-practices/3-testing.md`](docs/best-practices/3-testing.md)                                                                                         | Testing standards                                                                               |
| [`docs/best-practices/STACK-SPECIFIC.md`](docs/best-practices/STACK-SPECIFIC.md)                                                                               | Stack-specific conventions                                                                      |
| [`REVIEW.md`](REVIEW.md)                                                                                                                                       | Canonical review contract for spec, plan, and code review gates                                 |
| [`docs/workflow/development-workflow/README.md`](docs/workflow/development-workflow/README.md)                                                                 | AI development workflow (master doc)                                                            |
| [`docs/workflow/development-workflow/protocols/00-add-backlog-item-protocol.md`](docs/workflow/development-workflow/protocols/00-add-backlog-item-protocol.md) | Create backlog work items in a configured tracker (before spec/plan work)                       |
| [`docs/workflow/development-workflow/agent-model-config.md`](docs/workflow/development-workflow/agent-model-config.md)                                         | Model assignments, tool restrictions, and override guide for all agents                         |
| [`.ai-dev-workflow.yaml`](.ai-dev-workflow.yaml)                                                                                                               | Repo-level workflow integration manifest (review tools, issue tracker, VCS, browser automation) |
| [`LLM_RULES.md`](LLM_RULES.md)                                                                                                                                 | Agent commit rules enforced by Haystack pre-commit hooks (when installed)                       |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lhpaul/ai-dev-framework-template](https://github.com/lhpaul/ai-dev-framework-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
