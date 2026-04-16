---
trigger: always_on
description: **ALWAYS read these files at the start of every session:**
---

# GitHub Copilot Custom Instructions - EU Parliament Monitor

## 📋 Required Reading on Session Start

**ALWAYS read these files at the start of every session:**

1. **`.github/workflows/copilot-setup-steps.yml`** - Workflow configuration and permissions
2. **`.github/copilot-mcp.json`** - MCP server configuration and available tools
3. **`README.md`** - Project overview, features, and documentation links
4. **`.github/skills/`** - Skills library (security, architecture, compliance, testing, gh-aw)
5. **`.github/agents/`** - Specialized agents for delegation

## 🎯 Project Overview

**EU Parliament Monitor** is a European Parliament Intelligence Platform generating multi-language news articles (14 languages) using EP open data via MCP server integration, powered by GitHub Agentic Workflows.

- **Stack**: Node.js 25, TypeScript 6, HTML5/CSS3, Vitest, Playwright, ESLint
- **License**: Apache-2.0 | **Deployment**: AWS S3/CloudFront (primary) with GitHub Pages as fallback/runbook
- **Data**: European Parliament MCP Server (`european-parliament-mcp-server@1.2.1`)
- **Languages**: EN, SV, DA, NO, FI, DE, FR, ES, NL, AR, HE, JA, KO, ZH
- **Agentic Workflows**: 10 gh-aw markdown workflows for automated news generation
- **Security**: ISO 27001, NIST CSF 2.0, CIS Controls v8.1, GDPR, NIS2, EU CRA

## 🤖 Available Agents

| Agent | Purpose |
|-------|---------|
| **news-journalist** | The Economist-style EU Parliament reporting in 14 languages |
| **data-pipeline-specialist** | European Parliament MCP server integration and data pipelines |
| **frontend-specialist** | HTML5/CSS3, WCAG 2.1 AA accessibility, responsive design |
| **quality-engineer** | Testing, HTML validation, accessibility testing, performance |
| **devops-engineer** | GitHub Actions, CI/CD, gh-aw workflow compilation |
| **documentation-architect** | C4 models, Mermaid diagrams, API documentation |
| **product-task-agent** | Issue creation, product management, ISMS tracking |
| **agentic-workflows** | gh-aw workflow creation, debugging, and upgrades |

**Delegate specialized tasks to the appropriate agent.**

## 🏗️ Build & Test Commands

```bash
npm run lint          # ESLint (lint TypeScript in src/)
npm run htmlhint      # HTMLHint validation (HTML files)
npm run test          # Run unit tests (Vitest)
npm run test:coverage # Tests with coverage reporting
npm run test:e2e      # Playwright E2E tests
npm run generate-news # Generate multi-language news articles
npm run docs:generate # Generate TypeDoc API docs
npm run format        # Prettier formatting
npm run build         # TypeScript compilation
```

## 🔄 GitHub Agentic Workflows (gh-aw)

This project uses **10 gh-aw markdown workflows** in `.github/workflows/*.md` for automated news generation. These are compiled to `.lock.yml` files and run AI agents (Copilot/Claude/Codex) in sandboxed GitHub Actions with safe outputs.

**Workflow files**: `news-breaking.md`, `news-weekly-review.md`, `news-monthly-review.md`, `news-week-ahead.md`, `news-month-ahead.md`, `news-committee-reports.md`, `news-motions.md`, `news-propositions.md`, `news-article-generator.md`, `news-translate.md`

**Key concepts**: Safe outputs (create-pull-request with constraints), AWF firewall (Squid proxy allowlist), 5-layer security model, JSONL artifacts, lock file compilation.

**gh-aw docs**: https://github.github.com/gh-aw/ | [Abridged](https://github.github.com/gh-aw/llms-small.txt) | [Full](https://github.github.com/gh-aw/llms-full.txt)

## 🚨 Critical Rules

### MUST Follow
1. **Security First** - Follow Secure Development Policy, no secrets in code
2. **WCAG 2.1 AA** - All content must be accessible
3. **Test Before Commit** - Run `npm run lint && npm run test` before committing
4. **Multi-Language** - Changes affecting content must consider all 14 languages
5. **ISMS Compliance** - Follow ISO 27001, NIST CSF, CIS Controls frameworks
6. **Architecture Docs** - Update ARCHITECTURE.md, SECURITY_ARCHITECTURE.md when relevant
7. **Minimal Changes** - Make surgical, focused changes only
8. **gh-aw Workflows** - Compile with `gh aw compile --validate` after editing .md workflows

### MUST NOT Do
1. **Never** hard-code secrets, credentials, or API keys
2. **Never** break WCAG 2.1 AA compliance
3. **Never** skip testing before committing
4. **Never** use deprecated crypto (MD5, SHA-1, DES, 3DES)
5. **Never** merge Dependabot PRs that modify compiled `.lock.yml` files directly — recompile with `gh aw compile` instead

## 📐 Architecture Documentation (C4 Model)

**Current**: ARCHITECTURE.md, DATA_MODEL.md, FLOWCHART.md, STATEDIAGRAM.md, MINDMAP.md, SWOT.md, SECURITY_ARCHITECTURE.md, THREAT_MODEL.md

**Future**: FUTURE_ARCHITECTURE.md, FUTURE_DATA_MODEL.md, FUTURE_FLOWCHART.md, FUTURE_STATEDIAGRAM.md, FUTURE_MINDMAP.md, FUTURE_SWOT.md, FUTURE_SECURITY_ARCHITECTURE.md

**Reference**: [Hack23 Secure Development Policy](https://github.com/Hack23/ISMS-PUBLIC/blob/main/Secure_Development_Policy.md)

## 🔒 ISMS Compliance

All work MUST align with [Hack23 ISMS-PUBLIC](https://github.com/Hack23/ISMS-PUBLIC):

| Framework | Version | Scope |
|-----------|---------|-------|
| ISO 27001 | 2022 | Information security management |
| NIST CSF | 2.0 | Cybersecurity framework |
| CIS Controls | v8.1 | Security best practices |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Hack23) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
