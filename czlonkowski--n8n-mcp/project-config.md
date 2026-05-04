---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> **Note:** This file is committed to a public OSS repository. Never add sensitive information (API keys, internal URLs, credentials, private infrastructure details) here.

## Project Overview

n8n-mcp is a comprehensive documentation and knowledge server that provides AI assistants with complete access to n8n node information through the Model Context Protocol (MCP). It serves as a bridge between n8n's workflow automation platform and AI models, enabling them to understand and work with n8n nodes effectively.

### Current Architecture:
```
src/
├── loaders/
│   └── node-loader.ts         # NPM package loader for both packages
├── parsers/
│   ├── node-parser.ts         # Enhanced parser with version support
│   └── property-extractor.ts  # Dedicated property/operation extraction
├── mappers/
│   └── docs-mapper.ts         # Documentation mapping with fixes
├── database/
│   ├── schema.sql             # SQLite schema
│   ├── node-repository.ts     # Data access layer
│   └── database-adapter.ts    # Universal database adapter (NEW in v2.3)
├── services/
│   ├── property-filter.ts     # Filters properties to essentials (NEW in v2.4)
│   ├── example-generator.ts   # Generates working examples (NEW in v2.4)
│   ├── task-templates.ts      # Pre-configured node settings (NEW in v2.4)
│   ├── config-validator.ts    # Configuration validation (NEW in v2.4)
│   ├── enhanced-config-validator.ts # Operation-aware validation (NEW in v2.4.2)
│   ├── node-specific-validators.ts  # Node-specific validation logic (NEW in v2.4.2)
│   ├── property-dependencies.ts # Dependency analysis (NEW in v2.4)
│   ├── type-structure-service.ts # Type structure validation (NEW in v2.22.21)
│   ├── expression-validator.ts # n8n expression syntax validation (NEW in v2.5.0)
│   └── workflow-validator.ts  # Complete workflow validation (NEW in v2.5.0)
├── types/
│   ├── type-structures.ts      # Type structure definitions (NEW in v2.22.21)
│   ├── instance-context.ts     # Multi-tenant instance configuration
│   └── session-state.ts        # Session persistence types (NEW in v2.24.1)
├── constants/
│   └── type-structures.ts      # 22 complete type structures (NEW in v2.22.21)
├── templates/
│   ├── template-fetcher.ts    # Fetches templates from n8n.io API (NEW in v2.4.1)
│   ├── template-repository.ts # Template database operations (NEW in v2.4.1)
│   └── template-service.ts    # Template business logic (NEW in v2.4.1)
├── scripts/
│   ├── rebuild.ts             # Database rebuild with validation
│   ├── validate.ts            # Node validation
│   ├── test-nodes.ts          # Critical node tests
│   ├── test-essentials.ts     # Test new essentials tools (NEW in v2.4)
│   ├── test-enhanced-validation.ts # Test enhanced validation (NEW in v2.4.2)
│   ├── test-structure-validation.ts # Test type structure validation (NEW in v2.22.21)
│   ├── test-workflow-validation.ts # Test workflow validation (NEW in v2.5.0)
│   ├── test-ai-workflow-validation.ts # Test AI workflow validation (NEW in v2.5.1)
│   ├── test-mcp-tools.ts      # Test MCP tool enhancements (NEW in v2.5.1)
│   ├── test-n8n-validate-workflow.ts # Test n8n_validate_workflow tool (NEW in v2.6.3)
│   ├── test-typeversion-validation.ts # Test typeVersion validation (NEW in v2.6.1)
│   ├── test-workflow-diff.ts  # Test workflow diff engine (NEW in v2.7.0)
│   ├── test-tools-documentation.ts # Test tools documentation (NEW in v2.7.3)
│   ├── fetch-templates.ts     # Fetch workflow templates from n8n.io (NEW in v2.4.1)
│   └── test-templates.ts      # Test template functionality (NEW in v2.4.1)
├── mcp/
│   ├── server.ts              # MCP server with enhanced tools
│   ├── tools.ts               # Tool definitions including new essentials
│   ├── tools-documentation.ts # Tool documentation system (NEW in v2.7.3)
│   └── index.ts               # Main entry point with mode selection
├── utils/
│   ├── console-manager.ts     # Console output isolation (NEW in v2.3.1)
│   └── logger.ts              # Logging utility with HTTP awareness
├── http-server-single-session.ts  # Single-session HTTP server (NEW in v2.3.1)
│                                   # Session persistence API (NEW in v2.24.1)
├── mcp-engine.ts              # Clean API for service integration (NEW in v2.3.1)
│                                # Session persistence wrappers (NEW in v2.24.1)
└── index.ts                   # Library exports
```

## Common Development Commands

```bash
# Build and Setup
npm run build          # Build TypeScript (always run after changes)
npm run rebuild        # Rebuild node database from n8n packages
npm run validate       # Validate all node data in database

# Testing
npm test               # Run all tests
npm run test:unit      # Run unit tests only
npm run test:integration # Run integration tests
npm run test:coverage  # Run tests with coverage report
npm run test:watch     # Run tests in watch mode
npm run test:structure-validation # Test type structure validation (Phase 3)

# Run a single test file
npm test -- tests/unit/services/property-filter.test.ts

# Linting and Type Checking

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [czlonkowski/n8n-mcp](https://github.com/czlonkowski/n8n-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
