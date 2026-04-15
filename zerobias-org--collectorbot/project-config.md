---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with collector bot content in this repository.
---

# CLAUDE.md - ZeroBias Community Collector Bot Repository

This file provides guidance to Claude Code (claude.ai/code) when working with collector bot content in this repository.

## IMPORTANT: Development Rules

**Before creating or modifying any collector bot, Claude MUST read and follow:**
- **[.claude/COLLECTORBOT_RULES.md](.claude/COLLECTORBOT_RULES.md)** - Comprehensive rules for all file types, naming conventions, and patterns
- **[.claude/DEVELOPMENT_WORKFLOW.md](.claude/DEVELOPMENT_WORKFLOW.md)** - Step-by-step workflow for creating and modifying collectors
- **[.claude/ADVANCED_MAPPING_GUIDE.md](.claude/ADVANCED_MAPPING_GUIDE.md)** - Advanced mapping, validation, and relationship handling
- **[.claude/VENDOR_PATTERNS.md](.claude/VENDOR_PATTERNS.md)** - Vendor-specific architectural patterns and best practices

**Creation & Validation System:**
- **[/create-collector](.claude/skills/create-collector.md)** - Create new collector with automatic validation
- **[/review-collector](.claude/skills/review-collector.md)** - Comprehensive validation using 8 parallel agents
- **[.claude/agents/*.md](.claude/agents/)** - 10 specialized agents (2 for creation, 8 for validation)

**Rules document defines:**
- Package structure and naming conventions (`package/<vendor>/<suite-if-exists>/<product>/`)
- NPM scope rules (`@zerobias-org/collectorbot-*` for this repository)
- Schema package usage (base vs `-ts` packages)
- Required vs optional source files
- Batch processing and groupId rules (CRITICAL for data integrity)
- Configuration file templates and validation rules

**Workflow document defines:**
- 10-phase development process from planning to commit
- Modification workflows for existing collectors
- Dependency verification (module and schema must exist)
- Configuration file creation and code generation
- Implementation patterns and best practices
- Testing and documentation requirements
- Schema class selection (collect everything possible)
- Parameter design (ask user what they need)
- Concurrency limits for nested requests (≤30 total)

**Advanced mapping guide defines:**
- Schema validation using base + TypeScript packages
- Date vs DateTime handling (Object.assign workaround)
- Semantic field mapping strategies
- Enum mapping patterns
- ID generation from unique fields
- Object relationship discovery and implementation
- Complex type conversions and edge cases
- TODO pattern for uncertain mappings
- Performance optimization (memory management, concurrency, pagination)
- Incremental collection patterns (timestamp, state, cursor-based)
- Retry logic and resilience (transient failures, rate limiting)

**Vendor patterns document defines:**
- AWS: Account + region patterns, ARN-based IDs, metadata structure
- Azure: Subscription + resource group patterns
- GCP: Project + zone patterns
- GitHub: Organization patterns, rate limit considerations
- Microsoft 365/Entra: Tenant-based patterns
- Generic SaaS: Multi-tenant patterns
- Shared collectors: Global data patterns

## Project Overview

This is the **ZeroBias Community Collector Bot Repository** containing open-source ETL (Extract/Transform/Load) logic for data collection. Collector bots read from source systems using Hub Modules and write to AuditgraphDB (objects described by Schema classes).

**Repository Role:** Community-contributed, open-source content for automated data collection from external systems

## Repository Structure

```
collectorbot/
├── .claude/                    # Claude Code configuration
│   └── COLLECTORBOT_RULES.md  # ⭐ MUST READ - Development rules
├── package/                    # Collector bot packages
│   └── <vendor>/              # Vendor directory
│       └── <suite?>/          # Optional suite directory
│           └── <product>/     # Product collector
│               ├── package.json
│               ├── collector.yml
│               ├── hub.yml
│               ├── parameters.yml
│               ├── tsconfig.json
│               ├── .eslintrc
│               ├── .mocharc.json
│               ├── generated/     # Auto-generated (DO NOT EDIT)
│               ├── src/           # Source code
│               └── test/          # Tests
├── collector/                  # ⚠️ DEPRECATED - Do not use
└── lerna.json                 # Monorepo configuration
```

## Quick Reference

### Creating a New Collector (Automated)

**Use the creation skill:**
```bash
/create-collector <vendor> <suite?> <product>

# Examples:
/create-collector avigilon alta access
/create-collector github github
/create-collector auditmation generic tls
```

This will:
1. Verify module and schema dependencies exist
2. Create directory structure and configuration files
3. Generate code and spawn implementation + mapping agents in parallel
4. Create documentation (README, USERGUIDE)
5. Build and lint
6. **Automatically validate with /review-collector**
7. Report compliance score and any issues

**Manual creation** (if needed):
1. Create directory: `package/<vendor>/<suite?>/<product>/`
2. Copy structure from existing collector
3. Follow `.claude/DEVELOPMENT_WORKFLOW.md` step-by-step

### Key Commands

```bash
# Install dependencies
npm install

# Generate code from hub.yml and parameters.yml

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zerobias-org) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
