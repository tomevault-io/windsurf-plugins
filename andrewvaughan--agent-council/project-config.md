---
trigger: always_on
description: > **This file** defines project rules, conventions, and standards. For skill workflow details, council reference, and plugin documentation, see [`.claude/README.md`](.claude/README.md).
---

# {PROJECT_NAME} Project Instructions <!-- TODO: Replace {PROJECT_NAME} with your project name -->

> **This file** defines project rules, conventions, and standards. For skill workflow details, council reference, and plugin documentation, see [`.claude/README.md`](.claude/README.md).

## Table of Contents

- [GitHub Repository](#github-repository)
- [Documentation Standards](#documentation-standards)
- [Code Conventions](#code-conventions)
- [Tech Stack](#tech-stack)
- [Infrastructure Philosophy](#infrastructure-philosophy)
- [Documentation Requirements](#documentation-requirements)
- [Development Workflow](#development-workflow)
  - [Routing: Quick Fix vs. Plan Feature](#routing-quick-fix-vs-plan-feature)
  - [Skill Boundaries](#skill-boundaries)
  - [Pipeline State Tracking](#pipeline-state-tracking)
  - [Project & Roadmap Integration](#project--roadmap-integration)
  - [GitHub Milestones](#github-milestones)
  - [Parent Issues and Sub-Issues](#parent-issues-and-sub-issues)
  - [Issue Scheduling and Dates](#issue-scheduling-and-dates)
  - [GitHub API Usage](#github-api-usage)
  - [Label Management](#label-management)
  - [Pull Request CI Requirements](#pull-request-ci-requirements)
- [Security Audit Scope](#security-audit-scope)
- [Quality Standards](#quality-standards)
  - [Scientific Citation Requirements](#scientific-citation-requirements)
- [REST API Standards](#rest-api-standards)
- [User-Facing Content Style](#user-facing-content-style)
- [Agent Collaboration Matrix](#agent-collaboration-matrix)
- [Council Conflict Resolution](#council-conflict-resolution)
- [Agent & Tooling Changes](#agent--tooling-changes)

## GitHub Repository

> [!IMPORTANT]
> <!-- TODO: Replace {OWNER}/{REPO} with your actual GitHub owner and repository name (e.g., "myorg/myproject"). The local directory name may not match the GitHub repo name. **Always use `--repo {OWNER}/{REPO}`** with `gh` commands, or omit `--repo` to let `gh` infer from the git remote. -->
> The GitHub repository for this project is **`{OWNER}/{REPO}`**. The local directory name may not match the GitHub repo name. **Always use `--repo {OWNER}/{REPO}`** with `gh` commands, or omit `--repo` to let `gh` infer from the git remote.

When using the `gh` CLI:

- **Do NOT guess the repo name** from the directory path. The git remote is the source of truth.
- **Preferred**: Omit `--repo` so `gh` auto-detects from the git remote
- **If specifying explicitly**: Always use `--repo {OWNER}/{REPO}` <!-- TODO: Replace {OWNER}/{REPO} with your GitHub owner/repo -->
- **Project board**: "{PROJECT_BOARD_NAME}" (project #{PROJECT_NUMBER}, owner: `{OWNER}`) <!-- TODO: Replace {PROJECT_BOARD_NAME}, {PROJECT_NUMBER}, and {OWNER} with your GitHub Projects board name, number, and owner -->

## Documentation Standards

**NEVER create non-standard markdown files in project root** (e.g., RUN.md, INSTALL.md, SETUP.md, QUICKSTART.md).

Allowed root markdown files: README.md, CONTRIBUTING.md, CHANGELOG.md, AGENTS.md, LICENSE

Where to put documentation:

- Project overview, setup, usage: README.md
- Contribution workflow: CONTRIBUTING.md
- Technical deep-dives: docs/ directory with descriptive names
- Decision records: docs/decisions/NNN-title.md
- Master document index: docs/INDEX.md

If content doesn't fit in README.md, create a file in docs/ and link from README.md.

**When adding new files to `docs/`**, add them to the appropriate table in `docs/INDEX.md` so agents can discover them. Include the document type and a one-line description.

### Markdown Frontmatter

**All project documentation files must include YAML frontmatter** to help agents quickly identify and understand each document's purpose. This applies to `README.md`, `CONTRIBUTING.md`, `CHANGELOG.md`, and files in `docs/`.

Required fields:

```yaml
---
type: <document type>
description: <one-line summary of the document's purpose>
---
```

**`type` values:**

| Type        | Used for                         | Examples                                 |
| ----------- | -------------------------------- | ---------------------------------------- |
| `guide`     | Tutorials, walkthroughs, how-tos | `docs/DEVELOPMENT.md`, `CONTRIBUTING.md` |
| `overview`  | Project-level summaries          | `README.md`                              |
| `reference` | API docs, specs, lookup tables   | `docs/api.md`                            |

When creating new project documentation, always include the appropriate frontmatter block before any content.

### Markdown Rendering

All project markdown is intended to be read in **GitHub Flavored Markdown (GFM)** renderers. Use GFM features actively to improve readability and communication.

#### Formatting Rules

- **Never place consecutive bold-label lines** without a blank line or list syntax between them. GFM collapses adjacent lines into a single paragraph.
- **Use bullet points (`-`)** for structured metadata, key-value pairs, and any multi-field blocks (e.g., Date/Council/Status headers in decision records).
- **Use blank lines** between paragraphs, after headings, and before/after lists.
- **Use tables** for structured data with 3+ columns.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [andrewvaughan/agent-council](https://github.com/andrewvaughan/agent-council) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
