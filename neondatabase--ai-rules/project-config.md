---
trigger: always_on
description: > **Note**: This repository has been archived. Content has moved to [neondatabase/agent-skills](https://github.com/neondatabase/agent-skills).
---

# Neon AI Rules - Project Documentation

> **Note**: This repository has been archived. Content has moved to [neondatabase/agent-skills](https://github.com/neondatabase/agent-skills).

## Project Overview

This repository contains a comprehensive suite of AI-powered development tools for Neon, including:

1. **Context Rules** (`.mdc` files): Markdown Context files that guide AI systems
2. **Claude Code Plugin**: Full-featured plugin with guided skills and MCP server integration
3. **Neon SDK Rules**: Guidelines for TypeScript and Python SDKs

The primary audience includes AI developers using Claude, Cursor, and other AI-powered code assistants who work with Neon databases.

## Repository Structure

```
.
├── *.mdc                          # Context rule files (16 total)
├── .claude-plugin/
│   └── marketplace.json           # Marketplace metadata
├── neon-plugin/                   # Claude Code plugin
│   ├── .claude-plugin/
│   │   └── plugin.json            # Plugin metadata
│   ├── .mcp.json                  # MCP server configuration
│   ├── evals/                     # Skill evaluation tests
│   └── skills/                    # Guided skills for Claude Code (6 total)
│       ├── add-neon-docs/         # Documentation reference installer
│       ├── neon-auth/             # Neon Auth integration
│       │   ├── guides/            # Setup guides (Next.js, React SPA)
│       │   └── templates/         # Auth client templates
│       ├── neon-drizzle/
│       │   ├── guides/            # Step-by-step workflow guides
│       │   ├── references/        # Technical reference docs
│       │   ├── scripts/           # Utility scripts
│       │   └── templates/         # Code templates
│       ├── neon-js/               # Full Neon JS SDK integration
│       │   ├── guides/            # Setup guides
│       │   └── templates/         # Client templates
│       ├── neon-serverless/
│       └── neon-toolkit/
├── references/                    # Shared technical reference docs (16 files)
│   ├── code-generation-rules.md
│   ├── neon-auth-*.md             # Auth-related references (11 files)
│   │   # Includes: common-mistakes, components, provider-config,
│   │   # setup (general, nextjs, nodejs, react-spa), troubleshooting,
│   │   # ui (general, nextjs, react-spa)
│   └── neon-js-*.md               # Neon JS references (4 files)
│       # Includes: adapters, data-api, imports, theming
├── mcp-prompts/                   # MCP prompt templates
├── .claude/
│   └── settings.local.json        # Local Claude Code settings
├── .serena/                       # Serena code intelligence cache
├── CHANGELOG.md                   # Version history and release notes
├── CONTRIBUTING.md                # Contribution guidelines
├── LICENSE                        # MIT License
├── README.md                      # User-facing documentation
└── CLAUDE.md                      # This file

```

## Context Rules (.mdc files)

### Getting Started (2 files)
- **neon-get-started.mdc**: Interactive onboarding guide for Neon projects
- **neon-get-started-kiro.mdc**: Kiro-specific onboarding guide

### Core Integration Rules (5 files)
- **neon-auth.mdc**: Neon Auth integration with `@neondatabase/auth` package
- **neon-js.mdc**: Full Neon JS SDK with `@neondatabase/neon-js` package
- **neon-serverless.mdc**: Serverless database connections and pooling
- **neon-drizzle.mdc**: Drizzle ORM integration with Neon
- **neon-toolkit.mdc**: Ephemeral database creation for testing

### SDK Rules (2 files)
- **neon-typescript-sdk.mdc**: TypeScript SDK usage patterns
- **neon-python-sdk.mdc**: Python SDK usage patterns

### API Rules (7 files)
- **neon-api-guidelines.mdc**: General API best practices
- **neon-api-projects.mdc**: Project management API
- **neon-api-branches.mdc**: Branch management API
- **neon-api-endpoints.mdc**: Compute endpoint management API
- **neon-api-organizations.mdc**: Organization and role management API
- **neon-api-keys.mdc**: API key and authentication management
- **neon-api-operations.mdc**: Operation execution and monitoring

## Claude Code Plugin Structure

### Plugin Configuration
- `.claude-plugin/plugin.json`: Contains plugin metadata (name, version, description)
- `.mcp.json`: Configures connection to Neon's remote MCP server

### Skills Directory
Each skill is self-contained with multiple components:

1. **SKILL.md**: Describes the skill's purpose and workflow
2. **scripts/**: Executable utilities for the skill (TypeScript or shell)
3. **templates/**: Code examples and templates
4. **guides/** (optional): Step-by-step guides for different scenarios
5. **references/** (optional): Technical reference documentation

#### Neon Drizzle Skill
- **Purpose**: Drizzle ORM setup and database management with comprehensive workflow support
- **Scripts**: `generate-schema.ts`, `run-migration.ts`
- **Templates**: Schema examples, drizzle config (HTTP and WebSocket adapters)
- **Guides**:
  - `new-project.md`: Setting up Drizzle in new projects
  - `existing-project.md`: Integrating Drizzle into existing codebases
  - `schema-only.md`: Schema-first workflow without migrations
  - `troubleshooting.md`: Common issues and solutions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [neondatabase/ai-rules](https://github.com/neondatabase/ai-rules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
