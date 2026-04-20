---
trigger: always_on
description: Use when:
---

# Agent Instructions

This repository contains agent skills and Claude Code plugins for HashiCorp products, including Terraform and Packer for infrastructure-as-code development.

## Repository Structure

```
agent-skills/
├── terraform/
│   ├── code-generation/
│   │   ├── .claude-plugin/plugin.json
│   │   └── skills/
│   │       ├── azure-verified-modules/
│   │       ├── terraform-style-guide/
│   │       ├── terraform-test/
│   │       └── terraform-search-import/
│   ├── module-generation/
│   │   ├── .claude-plugin/plugin.json
│   │   └── skills/
│   │       ├── refactor-module/
│   │       └── terraform-stacks/
│   └── provider-development/
│       ├── .claude-plugin/plugin.json
│       └── skills/
│           ├── new-terraform-provider/
│           ├── run-acceptance-tests/
│           ├── provider-actions/
│           └── provider-resources/
├── packer/
│   ├── builders/
│   │   ├── .claude-plugin/plugin.json
│   │   └── skills/
│   │       ├── aws-ami-builder/
│   │       ├── azure-image-builder/
│   │       └── windows-builder/
│   └── hcp/
│       ├── .claude-plugin/plugin.json
│       └── skills/
│           └── push-to-registry/
├── .claude-plugin/marketplace.json
├── README.md
└── AGENTS.md
```

## Installation Methods

### Method 1: Claude Code Plugin Installation

Install plugins using Claude Code CLI. First add the marketplace, then install plugins:

```bash
# Add the agent-skills marketplace
claude plugin marketplace add hashicorp/agent-skills

# Install plugins
claude plugin install terraform-code-generation@hashicorp
claude plugin install terraform-module-generation@hashicorp
claude plugin install terraform-provider-development@hashicorp
claude plugin install packer-builders@hashicorp
claude plugin install packer-hcp@hashicorp
```

Or use the interactive interface within Claude Code:
```
/plugin
```

This opens a tabbed interface where you can:
- **Discover**: Browse available plugins from all marketplaces
- **Installed**: View and manage installed plugins
- **Marketplaces**: Add, remove, or update marketplaces

Additional plugin management commands:
```bash
# Disable a plugin
claude plugin disable terraform-code-generation@hashicorp

# Re-enable a plugin
claude plugin enable terraform-code-generation@hashicorp

# Uninstall a plugin
claude plugin uninstall terraform-code-generation@hashicorp

# Update a plugin
claude plugin update terraform-code-generation@hashicorp
```

Installation scopes (use `--scope` flag):
- `user` (default): Available across all projects
- `project`: Shared with team via `.claude/settings.json`
- `local`: Project-specific, gitignored

### Method 2: Individual Skill Installation

Install individual skills using `npx skills add`:

```bash
# List all available skills
npx skills add hashicorp/agent-skills

# Code generation skills
npx skills add hashicorp/agent-skills/terraform/code-generation/skills/terraform-style-guide
npx skills add hashicorp/agent-skills/terraform/code-generation/skills/terraform-test
npx skills add hashicorp/agent-skills/terraform/code-generation/skills/azure-verified-modules
npx skills add hashicorp/agent-skills/terraform/code-generation/skills/terraform-search-import

# Module generation skills
npx skills add hashicorp/agent-skills/terraform/module-generation/skills/refactor-module
npx skills add hashicorp/agent-skills/terraform/module-generation/skills/terraform-stacks

# Provider development skills
npx skills add hashicorp/agent-skills/terraform/provider-development/skills/new-terraform-provider
npx skills add hashicorp/agent-skills/terraform/provider-development/skills/run-acceptance-tests
npx skills add hashicorp/agent-skills/terraform/provider-development/skills/provider-actions
npx skills add hashicorp/agent-skills/terraform/provider-development/skills/provider-resources

# Packer builder skills
npx skills add hashicorp/agent-skills/packer/builders/skills/aws-ami-builder
npx skills add hashicorp/agent-skills/packer/builders/skills/azure-image-builder
npx skills add hashicorp/agent-skills/packer/builders/skills/windows-builder

# Packer HCP skills
npx skills add hashicorp/agent-skills/packer/hcp/skills/push-to-registry
```

Skills are installed to `~/.claude/skills/` or project `.claude/skills/` directory.

### Method 3: Manual Installation

Copy skills directly to your Claude Code configuration:

```bash
# Clone the repository
git clone https://github.com/hashicorp/agent-skills.git

# Copy a plugin to Claude Code plugins directory
cp -r agent-skills/terraform/code-generation ~/.claude/plugins/

# Or copy individual skills
cp -r agent-skills/terraform/code-generation/skills/terraform-style-guide ~/.claude/skills/
```

## Plugin Contents

### terraform-code-generation

Skills for generating and validating Terraform HCL code:

| Skill | Description |
|-------|-------------|
| `terraform-style-guide` | Generate Terraform HCL code following HashiCorp style conventions and best practices |
| `terraform-test` | Writing and running `.tftest.hcl` test files |
| `azure-verified-modules` | Azure Verified Modules (AVM) requirements and certification |
| `terraform-search-import` | Discover existing resources with Terraform Search and bulk import into state |

### terraform-module-generation

Skills for creating and refactoring Terraform modules:

| Skill | Description |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hashicorp/agent-skills](https://github.com/hashicorp/agent-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
