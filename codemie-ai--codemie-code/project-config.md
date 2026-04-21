---
trigger: always_on
description: **Purpose**: AI-optimized execution guide for Claude Code agents working with the CodeMie Code codebase
---

# CLAUDE.md

**Purpose**: AI-optimized execution guide for Claude Code agents working with the CodeMie Code codebase

**Note**: CodeMie Code is an umbrella project with multiple agent plugins (Claude, Gemini, OpenCode, etc.) and provider integrations.

---

## 📚 GUIDE IMPORTS

This document references detailed guides stored in `.codemie/guides/`. Key references:

### 🚨 MANDATORY RULE: Check Guides First

**BEFORE searching files or codebase for information, you MUST:**

1. **Always check** if the guides referenced below contain relevant information for the prompt or task
2. **Use the Task Classifier** in [Instant Start](#-instant-start-read-first) to identify which guides are relevant
3. **Load and review** the appropriate P0 (required) guides BEFORE performing direct file searches
4. **Only proceed** to codebase searches after confirming guides don't contain the needed patterns/information

**This rule is MANDATORY and must be followed without exceptions.**

**Why**: Guides contain curated patterns, best practices, and architectural decisions. Checking them first:
- Prevents reinventing existing patterns
- Ensures consistency with established conventions
- Saves time by providing direct answers to common questions
- Reduces risk of introducing anti-patterns

**Workflow**: Prompt → Task Classifier → Load Relevant Guides → Check Guide Content → Then Search Codebase (if needed)

---

### 📖 Guide References by Category

**Architecture**:
- [Architecture]: .codemie/guides/architecture/architecture.md

**Development Practices**:
- [Development Practices]: .codemie/guides/development/development-practices.md
- [Code Quality]: .codemie/guides/standards/code-quality.md

**Testing**:
- [Testing Patterns]: .codemie/guides/testing/testing-patterns.md

**Standards & Workflows**:
- [Git Workflow]: .codemie/guides/standards/git-workflow.md
- [Code Quality]: .codemie/guides/standards/code-quality.md

**Integration & Security**:
- [External Integrations]: .codemie/guides/integration/external-integrations.md
- [Security Practices]: .codemie/guides/security/security-practices.md

---

## ⚡ INSTANT START (Read First)

### 1. Critical Rules (MANDATORY - Always Check)

| Rule | Trigger | Action Required |
|------|---------|-----------------|
| 🚨 **Check Guides First** | ANY new prompt/task | ALWAYS check relevant guides BEFORE searching codebase → [Guide Imports](#-guide-imports) |
| 🚨 **Testing** | User says "write tests", "run tests" | ONLY then work on tests → [Testing Policy](#testing-policy) |
| 🚨 **Git Ops** | User says "commit", "push", "create PR" | ONLY then do git operations → [Git Policy](#git-operations-policy) |
| 🚨 **Node Environment** | ANY TypeScript/npm command | No activation needed (Node.js >=20.0.0) → [Env Policy](#environment-policy) |
| 🚨 **Shell** | ANY shell command | ONLY bash/Linux syntax → [Shell Policy](#shell-environment-policy) |

**Emergency Recovery**: If commands fail → Check [Troubleshooting](#-troubleshooting-quick-reference)

### 2. Task Classifier (What am I doing?)

**Scan user request for keywords → Load guides → Execute**

| Keywords | Complexity | Load Guide (P0=Required) | Also Load (P1=Optional) |
|----------|-----------|--------------------------|-------------------------|
| **plugin, registry, agent, adapter** | Medium-High | .codemie/guides/architecture/architecture.md | .codemie/guides/integration/external-integrations.md |
| **architecture, layer, structure, pattern** | Medium | .codemie/guides/architecture/architecture.md | .codemie/guides/development/development-practices.md |
| **test, vitest, mock, coverage** | Medium | .codemie/guides/testing/testing-patterns.md | .codemie/guides/development/development-practices.md |
| **error, exception, validation** | Medium | .codemie/guides/development/development-practices.md | .codemie/guides/security/security-practices.md |
| **security, sanitize, credential** | High | .codemie/guides/security/security-practices.md | .codemie/guides/development/development-practices.md |
| **provider, sso, bedrock, litellm, langgraph** | Medium-High | .codemie/guides/integration/external-integrations.md | .codemie/guides/architecture/architecture.md |
| **cli, command, commander** | Medium | .codemie/guides/architecture/architecture.md | .codemie/guides/development/development-practices.md |
| **workflow, ci/cd, github, gitlab** | Medium | .codemie/guides/standards/git-workflow.md | - |
| **lint, eslint, format, code quality** | Simple | .codemie/guides/standards/code-quality.md | - |
| **commit, branch, pr, git** | Simple | .codemie/guides/standards/git-workflow.md | - |

**Guide Path**: All guides in `.codemie/guides/<category>/`

**Complexity Guide**:
- **Simple**: 1 file, < 5 min, obvious pattern → Use direct tools (Grep/Glob/Read)
- **Medium**: 2-5 files, 5-15 min, standard patterns → Use direct tools + guide reference
- **High**: 6+ files, 15+ min, architectural decisions → Consider EnterPlanMode or Task tool

### 3. Self-Check Before Starting

Before any action:
- [ ] Did I check relevant guides FIRST (before searching codebase)?
- [ ] Which critical rules apply? (guides/testing/git/env/shell)
- [ ] What keywords did I identify?
- [ ] What's the complexity level?

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [codemie-ai/codemie-code](https://github.com/codemie-ai/codemie-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
