---
trigger: always_on
description: **DevsForge Marketplace** is the ultimate plugin ecosystem for Claude Code CLI, providing 161 enterprise-grade plugins including 93 AI agents, 65 commands, and 3 advanced super plugins. This community-driven marketplace transforms Claude Code from a powerful AI coding assistant into a comprehensive development platform with specialized capabilities across all aspects of software development, DevOps, security, and business operations.
---

# DevsForge Marketplace

## Overview

**DevsForge Marketplace** is the ultimate plugin ecosystem for Claude Code CLI, providing 161 enterprise-grade plugins including 93 AI agents, 65 commands, and 3 advanced super plugins. This community-driven marketplace transforms Claude Code from a powerful AI coding assistant into a comprehensive development platform with specialized capabilities across all aspects of software development, DevOps, security, and business operations.

**Version**: 1.2.0
**Repository**: https://github.com/devsforge/marketplace
**License**: MIT
**Status**: Production Ready

### Key Features

- **161 Production-Ready Plugins**: Curated, tested, and enterprise-validated
- **Zero Dependencies**: Fully audited code with no external dependencies
- **Advanced Features**: Hooks, MCP servers, and event-driven automation
- **Comprehensive Documentation**: 250+ lines per plugin with real-world examples
- **Security First**: All plugins manually reviewed and security scanned
- **Community Driven**: Open source with active contributor community

## Architecture

### Repository Structure

```
devsforge-marketplace/
├── .claude-plugin/
│   └── marketplace.json          # Central marketplace registry (v1.2.0)
│                                  # - 161 plugin definitions
│                                  # - Metadata and versioning
│                                  # - Plugin source paths
│
├── plugins/
│   ├── agents/                    # 93 AI Agent Plugins
│   │   ├── ai-architect/
│   │   │   ├── .claude-plugin/
│   │   │   │   └── plugin.json   # Agent metadata
│   │   │   └── agents/
│   │   │       └── ai-engineer.md # Agent prompt template
│   │   │
│   │   ├── authentication-specialist/  # v1.1 - JWT/OAuth2 expert
│   │   ├── cache-strategist/           # v1.1 - Redis/caching patterns
│   │   ├── cors-security-expert/       # v1.1 - CORS configuration
│   │   ├── cost-optimizer/             # v1.1 - Cloud cost optimization
│   │   ├── database-expert/            # v1.1 - SQL optimization
│   │   ├── docker-specialist/          # v1.1 - Container optimization
│   │   ├── error-handler/              # v1.1 - Error handling patterns
│   │   ├── file-upload-specialist/     # v1.1 - S3/file handling
│   │   ├── form-validation-expert/     # v1.1 - Input validation
│   │   ├── git-workflow-expert/        # v1.1 - Git strategies
│   │   ├── graphql-specialist/         # v1.1 - GraphQL design
│   │   ├── microservices-architect/    # v1.1 - Microservices patterns
│   │   ├── pagination-expert/          # v1.1 - Pagination strategies
│   │   ├── rate-limiter/               # v1.1 - Rate limiting
│   │   ├── regex-master/               # v1.1 - Regex patterns
│   │   ├── rest-api-designer/          # v1.1 - REST API design
│   │   ├── serverless-engineer/        # v1.1 - Serverless architecture
│   │   ├── sql-query-optimizer/        # v1.1 - SQL optimization
│   │   ├── sre-reliability-engineer/   # v1.1 - SRE practices
│   │   ├── technical-writer-pro/       # v1.1 - Documentation
│   │   └── webhook-integrator/         # v1.1 - Webhook patterns
│   │   └── ... (70 more agents)
│   │
│   ├── commands/                  # 65 Command Plugins
│   │   ├── api-contract-tester/        # v1.1 - API testing
│   │   ├── bundle-analyzer/            # v1.1 - Bundle analysis
│   │   ├── changelog-generator/        # v1.1 - Changelog automation
│   │   ├── console-cleaner/            # v1.1 - Remove console logs
│   │   ├── docker-compose-generator/   # v1.1 - Docker compose files
│   │   ├── env-config-manager/         # v1.1 - Environment config
│   │   ├── env-validator/              # v1.1 - .env validation
│   │   ├── feature-flag-manager/       # v1.1 - Feature flags
│   │   ├── git-commit-helper/          # v1.1 - Smart commits
│   │   ├── hotfix-deployer/            # v1.1 - Hotfix workflows
│   │   ├── htaccess-generator/         # v1.1 - Apache config
│   │   ├── import-organizer/           # v1.1 - Import sorting
│   │   ├── incident-reporter/          # v1.1 - Incident management
│   │   ├── json-validator/             # v1.1 - JSON validation
│   │   ├── license-compliance/         # v1.1 - License checking
│   │   ├── migration-helper/           # v1.1 - DB migrations
│   │   ├── mock-data-generator/        # v1.1 - Test data
│   │   ├── nginx-config-generator/     # v1.1 - Nginx config
│   │   ├── package-updater/            # v1.1 - Dependency updates
│   │   ├── regex-tester/               # v1.1 - Regex testing
│   │   ├── robots-txt-generator/       # v1.1 - Robots.txt
│   │   ├── schema-generator/           # v1.1 - Schema generation
│   │   ├── sql-formatter/              # v1.1 - SQL beautification
│   │   ├── type-generator/             # v1.1 - TypeScript types
│   │   └── unused-code-finder/         # v1.1 - Dead code detection
│   │   └── ... (40 more commands)
│   │

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [devsforge/marketplace](https://github.com/devsforge/marketplace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
