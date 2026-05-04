---
trigger: always_on
description: This file provides comprehensive guidance for AI assistants (Claude, Gemini, ChatGPT, etc.) working with this security operations repository. It serves as the primary orientation document for understanding the project structure, capabilities, and specialized workflows.
---

# LLM Agent Instructions

This file provides comprehensive guidance for AI assistants (Claude, Gemini, ChatGPT, etc.) working with this security operations repository. It serves as the primary orientation document for understanding the project structure, capabilities, and specialized workflows.

## Project Overview

This is a security operations (SOC) runbook and persona system designed to guide LLM agents through standardized security workflows. The repository contains documentation, procedural guides, and configuration scripts for AI-assisted security operations.

## Repository Structure

- `rules_bank/` - Master source directory containing all documentation, personas, and runbooks
  - `personas/` - Security role definitions (SOC Analysts, Threat Hunters, Incident Responders, etc.)
  - `run_books/` - Step-by-step procedural guides for security tasks
    - `common_steps/` - Reusable procedure components
    - `irps/` - Incident Response Plans for major incidents
    - `guidelines/` - Best practices and documentation guides
- `.claude/` - Claude Code configuration directory
  - `rules_bank/` - Symlink to ../rules_bank (provides access to all content)
- `.clinerules/` - Cline configuration directory
  - `rules_bank/` - Symlink to ../rules_bank (provides access to all content)
- `.gemini/` - Gemini CLI configuration directory
  - `rules_bank/` - Symlink to ../rules_bank (provides access to all content)
- `reports/` - Generated security reports and examples

## Multi-LLM Architecture

This repository is designed for seamless integration across multiple AI assistants through a unified content structure:

### Supported AI Tools
- **Claude Code**: Accesses content through `.claude/rules_bank/` symlinks
- **Cline**: Utilizes `.clinerules/rules_bank/` directory structure  
- **Gemini CLI**: Reads from `.gemini/rules_bank/` configuration
- **Other LLMs**: Can directly access `rules_bank/` master content

### Unified Content Access
All AI tools access identical content through symlinks to the master `rules_bank/` directory. This architecture:
- Ensures consistency across different AI platforms
- Eliminates content duplication and maintenance overhead
- Provides standardized security workflows regardless of the AI assistant used
- Maintains specialized tool configurations while sharing core knowledge

## Configuration Scripts

### Setting Active Persona
```bash
python set_persona_rules.py <persona_name>
# Example: python set_persona_rules.py tier1_soc_analyst
```
Currently supported personas: `tier1_soc_analyst`, `threat_hunter`

### Linking Common Steps
```bash
python symlink_common_steps.py
```
Run this after changing personas to ensure common steps remain accessible.

## Key Concepts & Capabilities

### Core Components
- **Runbooks**: Tactical, step-by-step procedures for specific security tasks (alert triage, IOC enrichment, threat hunting)
- **IRPs**: End-to-end incident response plans for major incidents following PICERL lifecycle  
- **Personas**: Predefined security roles with specific responsibilities, skills, tool access, and specialized slash commands
- **MCP Integration**: Model Context Protocol tools for Chronicle SIEM, SOAR, Google Threat Intelligence, and Security Command Center

### Advanced Features
- **Slash Commands**: Specialized commands for security workflows (primarily supported in Claude)
- **Dynamic Configuration**: Persona-based context switching and tool access management
- **Report Generation**: Automated security report creation with standardized templates
- **Threat Intelligence**: Active integration with Google Threat Intelligence and security feeds
- **Multi-Platform SIEM**: Chronicle, SOAR case management, and cloud security integration

## Skills & Persona System

The repository includes a skills-based workflow system with persona-driven orchestration.

### Skills Directory Structure

```
skills/
├── _personas/          # Persona manifest files (YAML)
│   ├── tier1-analyst.yaml
│   ├── tier2-analyst.yaml
│   ├── threat-hunter.yaml
│   └── incident-responder.yaml
├── _workflows/         # Composite/meta-skills
│   ├── full-alert-triage/
│   └── full-investigation/
├── _roles/             # IAM role documentation
│   └── iam-matrix.md
└── <skill-name>/       # Individual atomic skills
    └── SKILL.md
```

### Using Personas

Persona manifests define which skills each security role can use and their typical workflows.

**Available Personas:**

| Persona | Primary Function | Key Skills |
|---------|-----------------|------------|
| `tier1-analyst` | Alert triage, initial assessment | triage-alert, enrich-ioc, check-duplicates |
| `tier2-analyst` | Deep investigation, escalated cases | deep-dive-ioc, correlate-ioc, triage-malware |
| `threat-hunter` | Proactive hunting | hunt-apt, hunt-ioc, hunt-threat, hunt-lateral-movement |
| `incident-responder` | PICERL lifecycle management | respond-ransomware, respond-malware, respond-phishing |

**Activating a Persona:**

Each LLM platform activates personas differently:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dandye/ai-runbooks](https://github.com/dandye/ai-runbooks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
