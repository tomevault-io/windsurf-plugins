---
trigger: always_on
description: Validates that each plugin manifest (`<plugin>/.claude-plugin/plugin.json` and `<plugin>/.codex-plugin/plugin.json`) parses, has the required fields, and that the marketplace catalogs reference plugin directories that actually exist. Runs offline and gates PRs from forks before the LLM-driven jobs.
---

# Pulumi Agent Skills

This repository contains official Pulumi agent skills for infrastructure as code workflows.

## Repository Structure

Skills are organized into four plugin groups:

### Migration Plugin (`migration/`)

Skills for converting and importing infrastructure from other tools to Pulumi:

- **pulumi-terraform-to-pulumi**: Migrate Terraform projects to Pulumi
- **pulumi-cdk-to-pulumi**: Migrate AWS CDK applications to Pulumi
- **cloudformation-to-pulumi**: Migrate AWS CloudFormation stacks/templates to Pulumi
- **pulumi-arm-to-pulumi**: Migrate Azure ARM templates and Bicep to Pulumi

### Pulumi Plugin (`pulumi/`)

Entry-point and specialized skills for writing and operating Pulumi infrastructure:

- **pulumi-overview**: Entry-point skill across `pulumi do`, IaC projects, and Pulumi Cloud; routes to specialized skills
- **pulumi-best-practices**: Best practices for writing reliable Pulumi programs
- **pulumi-component**: Guide for authoring ComponentResource classes
- **pulumi-automation-api**: Best practices for using Pulumi Automation API
- **pulumi-esc**: Guidance for working with Pulumi ESC (Environments, Secrets, and Configuration)
- **provider-upgrade**: Safe workflows for upgrading Pulumi providers without unintended infrastructure changes
- **package-usage**: Track which stacks across an organization use a package and at what versions

### Package Maintenance Plugin (`package-maintenance/`)

Skills for maintaining Pulumi provider repositories (provider authors and bridge maintainers):

- **pulumi-upgrade-provider**: Automate Pulumi provider repo upgrades with the upgrade-provider tool
- **upstream-patches**: Create, amend, remove, and rebase patches for Terraform provider submodules

### Delegation Plugin (`delegation/`)

Skills for handing off in-progress work from coding agents to Pulumi Neo for delegated execution:

- **pulumi-neo-handoff**: Transfer the current work to a new Pulumi Neo task with goal, repository pointers, and a compacted conversation summary

## Installation

### Claude Code Plugin System

```bash
/plugin marketplace add pulumi/agent-skills
/plugin install pulumi-migration
/plugin install pulumi
/plugin install pulumi-delegation
/plugin install pulumi-package-maintenance
```

### OpenAI Codex

```bash
codex plugin marketplace add pulumi/agent-skills
```

After the marketplace registers, install plugins from the Codex TUI: run `codex`, open the plugin marketplace with `/plugins`, and pick `pulumi-migration`, `pulumi`, `pulumi-delegation`, or `pulumi-package-maintenance`.

### Universal (all agents)

Install all skills:

```bash
npx skills add pulumi/agent-skills --skill '*'
```

Or install individual plugin groups:

```bash
npx skills add pulumi/agent-skills/migration --skill '*'             # 4 migration skills
npx skills add pulumi/agent-skills/pulumi --skill '*'                # 7 pulumi skills (overview + specialized)
npx skills add pulumi/agent-skills/delegation --skill '*'            # 1 delegation skill
npx skills add pulumi/agent-skills/package-maintenance --skill '*'   # 2 package-maintenance skills
```

This works with Claude Code, Cursor, Copilot, Codex, and other agent tools.

## Skill Writing Conventions

### Skill Name Format

Skill names follow the pattern: `pulumi-<feature>`. The entry-point router is **`pulumi-overview`** (install via the `pulumi` plugin).

### Trigger Phrases

Skill descriptions should include keywords and phrases that trigger automatic activation. Use "MUST be loaded when" or "Use when" to clarify activation criteria.

Example:
```yaml
description: Convert an AWS CDK application to Pulumi. This skill MUST be loaded whenever a user requests migration or conversion of a CDK application to Pulumi.
```

### Progressive Disclosure

Keep the main SKILL.md file focused and concise (under 500 lines recommended). For detailed reference material, add additional markdown files in the skill directory with meaningful names.

Example structure:
```
pulumi-cdk-to-pulumi/
├── SKILL.md              # Main skill file
├── cdk-convert.md        # Reference: cdk2pulumi tool usage
├── cdk-importer.md       # Reference: cdk-importer tool usage
└── cloudformation-id-lookup.md  # Reference: import ID lookup
```

Reference these files from the main SKILL.md using relative links:
```markdown
For detailed tool usage, see [cdk-convert.md](cdk-convert.md).
```

### Cross-Skill References

When one skill references another, use the pattern: `Use skill <skill-name>`.

Example:
- "Use skill `pulumi-component` for in-depth component authoring guidance"

## Testing

The `tests/` directory contains three pytest-based test suites that run automatically on every pull request via `.github/workflows/tests.yml`. Two require `ANTHROPIC_API_KEY`; the manifest test runs offline.

### Manifest validation (`test_manifests.py`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pulumi/agent-skills](https://github.com/pulumi/agent-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
