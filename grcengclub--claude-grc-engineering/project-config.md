---
trigger: always_on
description: Generates complete implementation packages including:
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is the official open-source Claude Code plugin marketplace of the [GRC Engineering Club](https://grcengclub.com) for Governance, Risk, and Compliance (GRC) professionals. The repository provides 30 specialized plugins organized into:

- **4 persona-based plugins** (grc-engineer, grc-auditor, grc-internal, grc-tprm)
- **21 framework-specific plugins** (soc2, nist-800-53, iso27001, fedramp-rev5, fedramp-20x, pci-dss, cmmc, hitrust, cis-controls, gdpr, csa-ccm, nydfs, dora, stateramp, essential8, glba, us-export, pbmm, ismap, irap)
- **4 Tier-1 connector plugins** (aws-inspector, gcp-inspector, github-inspector, okta-inspector) that emit findings matching `schemas/finding.schema.json`
- **OSCAL tooling plugins** (oscal, fedramp-ssp) wrapping external upstream projects

Each plugin contains commands (user-facing) and skills (AI agents that implement functionality).

## Architecture

### Plugin Structure

All plugins follow this structure:

```
plugins/{plugin-name}/
├── .claude-plugin/
│   └── plugin.json              # Plugin metadata (name, version, author)
├── commands/                    # User-facing commands (markdown files)
│   └── {command-name}.md        # Command documentation and instructions
├── skills/                      # AI agents that implement functionality
│   └── {skill-name}/
│       └── SKILL.md            # Skill prompt and behavior definition
├── config/                      # Configuration files (grc-engineer only)
│   ├── frameworks/             # Framework control mappings (YAML)
│   └── providers/              # Cloud provider templates (AWS, Azure, GCP, K8s)
└── scripts/                     # Node.js implementation scripts (grc-engineer only)
    └── {command-name}.js       # Command logic
```

### Two Plugin Patterns

**1. Persona Plugins (grc-engineer, grc-auditor, grc-internal, grc-tprm)**

- Include `scripts/` directory with Node.js implementations
- Include `config/` directory with YAML configurations
- Commands invoke scripts: `node scripts/map-control.js $ARGUMENTS`

**2. Framework Plugins (soc2, nist-800-53, iso27001, etc.)**

- Command files are self-contained with embedded instructions
- No scripts or config directories
- Skills contain framework expertise as prompts

### Key Directories

- `.claude-plugin/marketplace.json` - Marketplace configuration listing all plugins
- `plugins/grc-engineer/config/frameworks/` - Framework control definitions (SOC2, ISO27001, NIST 800-53, etc.)
- `plugins/grc-engineer/config/providers/` - Cloud provider evidence collection templates (aws.yaml, azure.yaml, gcp.yaml, kubernetes.yaml)
- `docs/ENTERPRISE-DEPLOYMENT.md` - AWS Bedrock and Google Vertex AI configuration

## Plugin Development

### Adding a New Framework Plugin

Framework plugins are lightweight and follow this pattern:

1. Create directory: `plugins/frameworks/{framework-name}/`
2. Add plugin.json with metadata
3. Create commands as markdown files with instructions
4. Create skills with framework expertise
5. Register in `.claude-plugin/marketplace.json`

Example: See `plugins/frameworks/soc2/` or `plugins/frameworks/pci-dss/`

### Adding a New Command to grc-engineer

1. Create command file: `plugins/grc-engineer/commands/{command}.md`
2. Create script: `plugins/grc-engineer/scripts/{command}.js`
3. If needed, add framework config: `plugins/grc-engineer/config/frameworks/{framework}.yaml`
4. Create skill: `plugins/grc-engineer/skills/{skill-name}/SKILL.md`

### Configuration File Patterns

**Framework configs** (`config/frameworks/*.yaml`):

```yaml
controls:
  CC6.1:
    title: Logical and Physical Access Controls
    keywords: [iam, access control, authentication]
```

**Provider templates** (`config/providers/*.yaml`):

```yaml
templates:
  mfa_root:
    keywords: [mfa, root]
    formats:
      python: |
        import boto3
        # Evidence collection script
      bash: |
        aws iam get-account-summary
```

## Testing Plugins

### Local Testing

```bash
# Clone and run with local plugins
git clone https://github.com/GRCEngClub/claude-grc-engineering.git
claude --plugin-dir ./claude-grc-engineering
```

### Testing Specific Commands

```bash
# Test grc-engineer command
/grc-engineer:map-control main.tf SOC2

# Test framework command
/soc2:assess detailed

# Test evidence collection
/grc-engineer:collect-evidence CC6.1 AWS python
```

## Enterprise Deployment

This repository supports enterprise deployments via AWS Bedrock and Google Vertex AI. See `docs/ENTERPRISE-DEPLOYMENT.md` for:

- AWS Bedrock configuration (FedRAMP High in GovCloud)
- Google Vertex AI configuration (FedRAMP High as of October 2025)
- IAM permissions and authentication
- Compliance certifications (HIPAA, SOC 2)

Configuration via environment variables:

```bash
# AWS Bedrock
export CLAUDE_CODE_USE_BEDROCK=1
export AWS_REGION=us-east-1

# Google Vertex AI
export CLAUDE_CODE_USE_VERTEX=1
export CLOUD_ML_REGION=us-east5
export ANTHROPIC_VERTEX_PROJECT_ID=your-project-id
```

## Marketplace Installation

Users install plugins via:

```bash
# Add marketplace (one-time)
/plugin marketplace add GRCEngClub/claude-grc-engineering

# Install specific plugins

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GRCEngClub/claude-grc-engineering](https://github.com/GRCEngClub/claude-grc-engineering) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
