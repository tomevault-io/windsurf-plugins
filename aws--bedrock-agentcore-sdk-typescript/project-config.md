---
trigger: always_on
description: This document provides guidance specifically for AI agents working on the AWS Bedrock AgentCore TypeScript SDK codebase. For human contributor guidelines, see [CONTRIBUTING.md](CONTRIBUTING.md).
---

# Agent Development Guide - AWS Bedrock AgentCore TypeScript SDK

This document provides guidance specifically for AI agents working on the AWS Bedrock AgentCore TypeScript SDK codebase. For human contributor guidelines, see [CONTRIBUTING.md](CONTRIBUTING.md).

## Purpose and Scope

**AGENTS.md** contains agent-specific repository information including:
- Directory structure with summaries of what is included in each directory
- Development workflow instructions for agents to follow when developing features
- Coding patterns and testing patterns to follow when writing code
- Style guidelines, organizational patterns, and best practices

**For human contributors**: See [CONTRIBUTING.md](CONTRIBUTING.md) for setup, testing, and contribution guidelines.

## Directory Structure

```
bedrock-agentcore-sdk-typescript/
├── .github/                      # GitHub Actions workflows and templates
│   ├── ISSUE_TEMPLATE/           # GitHub issue templates
│   │   ├── bug_report.md         # Bug report template
│   │   ├── custom.md             # Custom issue template
│   │   └── feature_request.md    # Feature request template
│   ├── workflows/                # CI/CD workflow definitions
│   │   ├── integration-testing.yml  # Integration test workflow
│   │   ├── pr-automerge.yml      # Auto-merge workflow for PRs
│   │   ├── pr-validation.yml     # PR validation checks
│   │   ├── release.yml           # Release automation workflow
│   │   └── security-scanning.yml # Security scanning workflow
│   ├── dependabot.yml            # Dependabot configuration
│   └── labeler.yml               # Auto-labeling configuration
│
├── .husky/                       # Git hooks (pre-commit checks)
│   └── pre-commit                # Pre-commit hook script
│
├── docs/                         # Documentation files
│   ├── PR.md                     # Pull request guidelines and template
│   └── TESTING.md                # Comprehensive testing guidelines
│
├── src/                          # Source code (all production code)
│   ├── runtime/                  # Runtime server for hosting agents
│   │   ├── __tests__/            # Unit tests for runtime
│   │   ├── app.ts                # BedrockAgentCoreApp implementation
│   │   ├── index.ts              # Runtime exports
│   │   └── types.ts              # Runtime type definitions
│   └── tools/                    # Tool definitions and types
│       ├── browser/              # Browser automation tool
│       │   ├── __tests__/        # Unit tests for browser tool
│       │   ├── integrations/     # Framework-specific integrations
│       │   ├── client.ts         # Browser client implementation
│       │   ├── index.ts          # Browser tool exports
│       │   └── types.ts          # Browser tool type definitions
│       └── code-interpreter/     # Code interpreter tool
│           ├── __tests__/        # Unit tests for code interpreter
│           ├── integrations/     # Framework-specific integrations
│           ├── client.ts         # Code interpreter client implementation
│           ├── index.ts          # Code interpreter exports
│           └── types.ts          # Code interpreter type definitions
│
├── examples/                     # Example applications and usage demos
│   ├── deep-research-ui/         # Next.js UI for deep research workflows
│   │   ├── app/                  # Next.js app directory structure
│   │   │   ├── api/              # API route handlers
│   │   │   ├── deep-research/    # Deep research page components
│   │   │   ├── globals.css       # Global CSS styles
│   │   │   └── layout.tsx        # Root layout component
│   │   ├── lib/                  # Utility libraries and helpers
│   │   │   └── agents/           # Agent-related utilities
│   │   ├── example_com_screenshot.png  # Example screenshot
│   │   ├── hackernews_homepage.png     # HackerNews homepage screenshot
│   │   ├── next-env.d.ts         # Next.js TypeScript declarations
│   │   ├── next.config.ts        # Next.js configuration
│   │   ├── package-lock.json     # UI dependency lock file
│   │   ├── package.json          # UI dependencies and scripts
│   │   ├── postcss.config.mjs    # PostCSS configuration
│   │   ├── README.md             # UI setup and usage instructions
│   │   ├── tailwind.config.ts    # Tailwind CSS configuration
│   │   └── tsconfig.json         # TypeScript config for UI
│   ├── agent-research-assistant.ts  # Research assistant example
│   ├── agent-with-browser.ts     # Browser integration example
│   ├── agent-with-code-interpreter.ts  # Code interpreter example
│   ├── README.md                 # Examples overview and setup
│   ├── setup.js                  # Example setup utilities
│   └── streaming-examples.ts     # Streaming usage examples
│
├── scripts/                      # Build and utility scripts
│   └── bump-version.ts           # Version bumping script
│
├── test-package/                 # Test package for verification
│   ├── package-lock.json         # Test package dependency lock file
│   ├── package.json              # Test package dependencies

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aws/bedrock-agentcore-sdk-typescript](https://github.com/aws/bedrock-agentcore-sdk-typescript) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
