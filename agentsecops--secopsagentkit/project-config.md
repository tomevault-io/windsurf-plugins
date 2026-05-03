---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

SecOpsAgentKit is a collection of Claude Code skills for security operations, DevSecOps, and secure SDLC practices. The repository follows a skill-based architecture where each security capability is packaged as a self-contained skill with YAML frontmatter, executable scripts, reference documentation, and configuration templates.

## Core Architecture

### Skill Structure

Every skill follows a standardized four-part structure:

1. **SKILL.md** (Required): Main skill file with YAML frontmatter defining metadata and markdown body containing procedural knowledge. Must be under 500 lines - use progressive disclosure pattern.

2. **scripts/** (Optional): Executable Python/Bash scripts for deterministic operations. Scripts must be tested and executable before submission.

3. **references/** (Optional): Reference documentation loaded on-demand by Claude. Contains detailed security framework mappings (OWASP, CWE, MITRE ATT&CK), remediation guides, and tool documentation.
 
4. **assets/** (Optional): Templates and configuration files used in output (not loaded into context). Includes CI/CD configs, rule templates, boilerplate code.

### Directory Organization

```
skills/
├── _template/              # DO NOT MODIFY - base template for all skills
├── appsec/                 # Application Security skills
├── devsecops/              # DevSecOps & CI/CD Security
├── secsdlc/                # Secure SDLC
├── threatmodel/            # Threat Modeling
├── compliance/             # Compliance & Auditing
└── incident-response/      # Incident Response
```

Each category contains skills following `skills/<category>/<tool/skill-type>-<tool/skill-name>/` structure.

### Frontmatter Architecture

All skills use YAML frontmatter with strict validation:

**Required fields**: `name`, `description`, `version`, `maintainer`, `category`, `tags`

**Critical**: The `description` field must include "Use when:" clause - this determines skill triggering in Claude Code.

**Versioning**: Semantic versioning (MAJOR.MINOR.PATCH). New skills start at 0.1.0.

**Categories**: Must be one of seven valid categories: `appsec`, `devsecops`, `secsdlc`, `threatmodel`, `compliance`, `incident-response`, `offsec`

**Frameworks**: List security frameworks referenced (OWASP, CWE, MITRE-ATT&CK, NIST, SOC2, PCI-DSS, GDPR, ISO27001, HIPAA)

## Development Commands

### Creating a New Skill

```bash
# Initialize from template
./scripts/init_skill.sh <skill-name> <category>

# Example: Create SAST skill in appsec category
./scripts/init_skill.sh sast-analyzer appsec
```

The initialization script:
- Validates skill name (must be kebab-case)
- Validates category (must be one of seven valid categories)
- Creates directory structure from `skills/_template/`
- Auto-populates frontmatter with skill name and category
- Makes scripts executable

### Validating a Skill

```bash
# Validate skill structure and frontmatter
./scripts/validate_skill.py skills/<category>/<skill-name>

# Example
./scripts/validate_skill.py skills/appsec/sast-semgrep
```

**Requires**: PyYAML (`pip install -r requirements.txt`)

The validator checks:
- YAML frontmatter format and required fields
- Semantic versioning compliance
- Description quality (length, "Use when" clause presence)
- Name format (kebab-case)
- Category validity
- No auxiliary files (README.md, CHANGELOG.md forbidden)
- Script executability
- Reference file linkage from SKILL.md

### Marketplace Integration

Skills are registered in `skills/.claude-plugin/marketplace.json`:

```json
{
  "plugins": [
    {
      "name": "plugin-name",
      "description": "Plugin description",
      "skills": ["./category/skill-name"]
    }
  ]
}
```

When adding a skill, update this file to register it under the appropriate plugin.

## Skill Development Patterns

### Progressive Disclosure

Keep SKILL.md concise (<500 lines). Move detailed content to `references/`:

- **SKILL.md**: Core workflows, quick start, common patterns
- **references/**: Detailed framework mappings, comprehensive remediation guides, extensive rule libraries

Link references clearly from SKILL.md with specific "when to read" guidance.

### Security-First Requirements

Every skill must document:
- **Sensitive Data Handling**: How to handle secrets, credentials, PII
- **Access Control**: Required permissions and authorization context
- **Audit Logging**: What should be logged for compliance
- **Compliance**: Relevant standards (SOC2, GDPR, PCI-DSS)
- **Safe Defaults**: Secure-by-default configurations

### Script Requirements

Scripts in `scripts/` directory:
- Must be tested before submission
- Must be executable (`chmod +x`)
- Should follow language-specific templates in `_template/scripts/`
- Must handle errors gracefully with proper exit codes
- Should support common flags (--help, --verbose, --output)

### Writing Style

Use imperative form in all instructions:
- ✓ "Run the security scan"
- ✗ "You should run the security scan"

Avoid generic development advice already known to Claude.

## Key Constraints


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AgentSecOps/SecOpsAgentKit](https://github.com/AgentSecOps/SecOpsAgentKit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
