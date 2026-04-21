---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a **Claude Code Skills Marketplace** repository containing DevOps-focused skills. It follows the Claude Code plugin/skill architecture where each skill is a self-contained directory with reference materials, scripts, and structured guidance.

## Repository Structure

```
devops-claude-skills/
├── .claude-plugin/
│   └── marketplace.json          # Marketplace registry (lists all available skills)
├── iac-terraform/                # Skill: Terraform & Terragrunt
│   ├── .claude-plugin/
│   │   └── plugin.json          # Skill metadata
│   └── skills/
│       ├── SKILL.md             # Main skill content (loaded when skill invoked)
│       ├── references/          # Supporting documentation
│       ├── scripts/             # Diagnostic/utility scripts
│       └── assets/              # Templates and resources
├── k8s-troubleshooter/          # Skill: Kubernetes troubleshooting
│   ├── .claude-plugin/
│   │   └── plugin.json
│   └── skills/
│       ├── SKILL.md
│       ├── references/
│       └── scripts/
├── aws-cost-optimization/       # Skill: AWS cost optimization & FinOps
│   ├── .claude-plugin/
│   │   └── plugin.json
│   ├── SKILL.md                 # Main skill content (at root level)
│   ├── scripts/                 # 6 automated analysis scripts
│   ├── references/              # Best practices and alternatives
│   └── assets/templates/        # Monthly report template
├── ci-cd/                       # Skill: CI/CD pipelines & DevSecOps
│   ├── .claude-plugin/
│   │   └── plugin.json
│   ├── SKILL.md
│   ├── scripts/
│   ├── references/
│   └── assets/
├── gitops-workflows/            # Skill: GitOps with ArgoCD & Flux
│   ├── .claude-plugin/
│   │   └── plugin.json
│   ├── SKILL.md                 # Main skill content (at root level)
│   ├── scripts/                 # 8 automated analysis scripts
│   ├── references/              # 8 comprehensive guides
│   └── assets/                  # ArgoCD, Flux, secrets templates
└── monitoring-observability/    # Skill: Monitoring & observability
    ├── .claude-plugin/
    │   └── plugin.json
    ├── SKILL.md                 # Main skill content (at root level)
    ├── scripts/                 # 6 automated analysis scripts
    ├── references/              # 6 comprehensive guides
    └── assets/templates/        # Prometheus alerts, OTel config, runbooks
```

## Architecture Principles

### 1. Skills as Self-Contained Modules
Each skill directory is completely independent and contains:
- **SKILL.md**: Main instructional content (the "brain" of the skill)
- **references/**: Detailed supporting documentation (troubleshooting guides, best practices)
- **scripts/**: Python automation scripts for diagnostics and validation
- **assets/**: Templates, examples, and reusable resources
- **.claude-plugin/plugin.json**: Metadata (name, description, version, author)

### 2. Two-Layer Documentation Pattern
Skills use a two-layer documentation approach:
- **SKILL.md**: Workflow-oriented guidance with decision trees, quick references, and "when to use" sections
- **references/*.md**: Deep-dive documentation for specific topics (e.g., `troubleshooting.md`, `best_practices.md`)

This pattern allows SKILL.md to stay focused on workflows while references provide comprehensive detail when needed.

### 3. Diagnostic Scripts Pattern
Each skill includes Python scripts that follow a common pattern:
- Accept relevant parameters (path, namespace/pod, etc.)
- Perform automated checks and analysis
- Provide structured output with actionable recommendations
- Can be run standalone or integrated into workflows

## Available Skills

### iac-terraform
**Purpose**: Terraform and Terragrunt infrastructure as code workflows

**Key Components**:
- `skills/SKILL.md`: Comprehensive Terraform/Terragrunt workflows including module development, state management, and troubleshooting
- `skills/references/best_practices.md`: Project structure, state management, module design, security practices, CI/CD patterns
- `skills/references/troubleshooting.md`: Common issues (state locks, drift, provider errors, resource issues, performance)
- `skills/scripts/inspect_state.py`: Terraform state inspection and drift detection
- `skills/scripts/validate_module.py`: Module validation against best practices
- `skills/assets/templates/MODULE_TEMPLATE.md`: Complete module structure template

**Workflow Pattern**: Decision-tree based (Is this reusable? → Module vs Environment Config)

### k8s-troubleshooter
**Purpose**: Systematic Kubernetes troubleshooting and incident response

**Key Components**:
- `skills/SKILL.md`: Systematic troubleshooting workflow from context gathering to post-incident
- `skills/references/common_issues.md`: Detailed issue library (ImagePullBackOff, CrashLoopBackOff, OOMKilled, node issues, networking, storage, RBAC)
- `skills/references/incident_response.md`: Structured incident response framework with severity levels and playbooks
- `skills/scripts/cluster_health.py`: Cluster-wide health check (nodes, system pods, failures)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ahmedasmar/devops-claude-skills](https://github.com/ahmedasmar/devops-claude-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
