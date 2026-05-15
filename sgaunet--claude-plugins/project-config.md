---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a **Claude Code Plugin Marketplace** repository that contains specialized agent definitions for DevOps and software engineering workflows. The repository follows the Claude Code plugin ecosystem structure, providing reusable agents that can be proactively activated based on context.

## Architecture

### Marketplace Structure
```
.claude-plugin/
└── marketplace.json          # Marketplace definition and plugin registry

plugins/
├── devops-infrastructure/    # Infrastructure & operations agents
│   ├── .claude-plugin/
│   │   └── plugin.json
│   ├── agents/               # Agent definitions (5 agents)
│   ├── commands/             # Custom slash commands (1 command)
│   └── .mcp.json             # MCP server integration
│
├── software-engineering/     # General engineering workflows
│   ├── .claude-plugin/
│   │   └── plugin.json
│   ├── agents/               # Agent definitions (7 agents)
│   ├── commands/             # Custom slash commands (9 commands)
│   └── .mcp.json             # MCP server integration
│
└── go-specialist/            # Go language expertise
    ├── .claude-plugin/
    │   └── plugin.json
    ├── agents/               # Agent definitions (1 agent)
    ├── commands/             # Custom slash commands (10 commands)
    └── .mcp.json             # MCP server integration
```

### Agent Architecture

Agents are defined in **markdown files with YAML frontmatter** in `plugins/*/agents/`. Each agent follows this structure:

**Frontmatter fields:**
- `name`: Agent identifier (required)
- `description`: When to use this agent (required, shown in Claude Code UI)
- `model`: LLM model to use (sonnet/opus/haiku/inherit)
- `tools`: Optional tool restrictions (agents use `tools`, not `allowed-tools`)
- `color`: UI color (red/blue/green/yellow/purple/orange/pink/cyan)
- `effort`: Optional effort level override (low/medium/high/max)
- `maxTurns`: Optional max turns before agent must complete
- `skills`: Optional skill references for specialized workflows

**Content sections:**
1. **Proactive Triggers**: Automatic activation conditions based on file patterns or keywords
2. **Core Capabilities**: Domain expertise and technical knowledge
3. **Implementation Patterns**: Best practices and approach
4. **Deliverables**: Expected outputs and artifacts
5. **Multi-Agent Coordination**: Collaboration patterns with other agents (in body, not frontmatter)

### Available Plugins & Agents

#### devops-infrastructure
Infrastructure, deployment, and operations specialists.

| Agent | Purpose | Proactive Activation |
|-------|---------|---------------------|
| `aws-specialist` | AWS cloud architecture, Well-Architected Framework, cost optimization | .tf (AWS), cloudformation, "aws", ARNs |
| `cicd-specialist` | CI/CD pipelines (GitHub Actions, GitLab CI, Forgejo) | .github/workflows, pipeline terms |
| `database-specialist` | Schema design, query optimization, migrations | .sql files, "slow query", "index" |
| `devops-specialist` | Infrastructure as Code (Terraform, Ansible, CloudFormation) | .tf, .yml files, cloud terms |
| `postgresql-specialist` | PostgreSQL 16+ advanced features, query optimization, replication | .sql, migrations, "postgresql", slow query |

#### software-engineering
General development workflows and best practices.

| Agent | Purpose | Proactive Activation |
|-------|---------|---------------------|
| `code-review-enforcer` | Code quality, security, best practices review | After file modifications |
| `debugger` | Root cause analysis, error resolution | Errors, test failures |
| `docs-architect` | Long-form technical documentation generation | Documentation requests (uses Opus model) |
| `html-first-frontend` | HTMX, Alpine.js, Bootstrap | HTML/CSS without frameworks |
| `license-specialist` | Open source license compliance for SaaS | Dependency files, license terms |
| `payment-integrator` | Stripe, PayPal, subscription billing | Payment SDKs, checkout flows |
| `security-auditor` | Security vulnerability detection, OWASP compliance, auth/crypto review | After auth/crypto/API changes |

#### go-specialist
Go language expertise and optimization.

| Agent | Purpose | Proactive Activation |
|-------|---------|---------------------|
| `golang-pro` | Go 1.25+ development, concurrency, performance | .go files, go.mod |

## Development Commands

### Task Runner (Task)
This project uses [Task](https://taskfile.dev) for automation:

```bash
# List all available tasks
task

# Validate plugin structure
task check
```

### Plugin Validation

```bash
# Validate individual plugin
cd plugins/<plugin-name> && claude plugin validate .

# Validate marketplace
claude plugin validate .claude-plugin/marketplace.json
```

### Plugin Installation (End Users)

Add this marketplace to Claude Code:
```bash
/plugin marketplace add sgaunet/claude-plugins
```

Or link locally for development:
```bash
/plugin marketplace add ./path/to/claude-plugins
```

## Working with Agents

### Creating New Agents

1. Choose appropriate plugin:
   - `devops-infrastructure` for infrastructure/operations agents

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sgaunet/claude-plugins](https://github.com/sgaunet/claude-plugins) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
