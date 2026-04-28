---
trigger: always_on
description: This repository manages AWS cloud infrastructure using Terraform. All cloud resources are defined as code, version-controlled, and deployed through automated pipelines. Claude Code assists with authoring, reviewing, refactoring, and validating Terraform configurations for AWS.
---

# Terraform AWS Infrastructure

## Description

This repository manages AWS cloud infrastructure using Terraform. All cloud resources are defined as code, version-controlled, and deployed through automated pipelines. Claude Code assists with authoring, reviewing, refactoring, and validating Terraform configurations for AWS.

---

## Project Overview

- **Cloud Provider:** AWS
- **IaC Tool:** Terraform (OpenTofu compatible)
- **State Backend:** S3 + DynamoDB locking
- **Secrets:** Never stored in code — use AWS Secrets Manager or SSM Parameter Store references
- **Environments:** Separated by Terraform workspace; variable files live in `vars/<workspace-name>/` — workspace names vary by project

---

## Architecture

```
.
├── CLAUDE.md
├── .mcp.json
├── main.tf
├── variables.tf
├── outputs.tf
├── providers.tf
├── vars/
│   └── <workspace-name>/
│       ├── terraform.tfvars
│       └── <additional>.tfvars
├── modules/
│   └── <module-name>/
│       ├── main.tf
│       ├── variables.tf
│       ├── outputs.tf
│       └── README.md
├── .claude/
│   ├── agents/           # Sub-agent definitions
│   ├── commands/         # Slash command prompts
│   ├── hooks/            # Pre/PostToolUse shell scripts
│   ├── skills/           # Reusable skill prompts
│   └── settings.json     # Hooks and permissions config
└── docs/
```

- **`vars/`** — One subdirectory per Terraform workspace. Each contains a `terraform.tfvars` plus any additional `.tfvars` files needed for that workspace. To target a workspace: `terraform workspace select <name>` then `terraform plan -var-file=vars/<name>/terraform.tfvars`.
- **`providers.tf`** — Defines the `terraform` block (`required_version`, `required_providers`) and all provider configurations (e.g., `aws`). This is the single source of truth for provider versions and AWS region/assume-role settings.
- **`modules/`** — Reusable, composable Terraform modules scoped by AWS service or logical domain. Each module is self-contained with its own variables, outputs, and README.
- **`docs/`** — Architecture decision records (ADRs), diagrams, and runbooks. Reference with `@docs/<file>.md` when needed.

> Treat every workspace with caution. See Safety Layers.

---

## MCP Servers

MCP servers are configured in `.mcp.json` at the project root. The following server is active for this repository:

- **`hashicorp/terraform-mcp-server`** — provides real-time access to Terraform Registry provider documentation, modules, and policies. Ensures generated Terraform uses accurate, up-to-date resource arguments rather than potentially stale training data.
- **`awslabs/aws-documentation-mcp-server`** — provides real-time access to AWS service documentation. Useful when authoring resources, verifying service behaviour, or checking API-level details not covered by Terraform provider docs. No AWS credentials required.

Both servers run locally over stdio. See `.mcp.json` for configuration.

---

## Custom Agents

Agent definitions live in `.claude/agents/`. Each agent file contains a YAML frontmatter block (name, description, model, tools, skills) followed by behavioural instructions.

| Agent | Model | Purpose |
|---|---|---|
| `devops-engineer` | opus | Authors, scaffolds, and refactors Terraform code; validates configurations against project conventions |
| `devsecops-engineer` | sonnet | Audits Terraform for security misconfigurations, IAM over-permissioning, public exposure, and missing encryption |
| `finops-engineer` | haiku | Analyses Terraform configurations for cost impact; identifies expensive defaults and recommends optimisations |

**Agent routing rules:**
- Always delegate AWS and Terraform questions, tasks, and commands to the `devops-engineer` agent — do not answer them directly.

---

## Memory & Team Knowledge Sharing

Whenever you save a memory (feedback, project context, or workflow rules), also add the same information to the relevant section of this `CLAUDE.md` file so all team members benefit from it in their sessions.

---

## Skills

Skills live in `.claude/skills/`, each as a subdirectory containing a `SKILL.md` file with YAML frontmatter (name + description) and instructions. Claude loads them automatically when relevant.

| Skill directory | Purpose |
|---|---|
| `skills/terraform-style/` | HCL formatting, naming, and structure conventions |
| `skills/aws-tagging/` | Required and recommended AWS resource tags |
| `skills/project-context/` | Project-specific context: account structure, approved services, and architectural decisions |

Example frontmatter for a skill:
```yaml
---
name: aws-tagging
description: Applies required AWS tagging standards to Terraform resources. Use when writing or reviewing any resource that supports tags.
---
```

*Keep each `SKILL.md` focused and under 500 lines. Move bulky reference material into a `references/` subdirectory within the skill.*

---

## Commands

Slash commands live in `.claude/commands/`. Run them with `/command-name` inside a session.

| Command | Description |
|---|---|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jacobscunn07) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-15 -->
