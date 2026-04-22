---
trigger: always_on
description: Wegent is an open-source AI-native operating system for defining, organizing, and running intelligent agent teams.
---

# AGENTS.md

Wegent is an open-source AI-native operating system for defining, organizing, and running intelligent agent teams.

---

## 📋 Project Overview

**Multi-module architecture:**
- **Backend** (FastAPI + SQLAlchemy + MySQL): RESTful API and business logic
- **Frontend** (Next.js 15 + TypeScript + React 19): Web UI with shadcn/ui components
- **Executor**: Task execution engine (Claude Code, Agno, Dify, ImageValidator)
- **Executor Manager**: Task orchestration via Docker
- **Chat Shell**: Lightweight AI chat engine for Chat Shell type (LangGraph + multi-LLM)
- **Shared**: Common utilities, models, and cryptography

**Core principles:**
- Kubernetes-inspired CRD design (Ghost, Model, Shell, Bot, Team, Task, Skill, Workspace)
- System should be simple AFTER your work - prioritize code simplicity and extensibility

**📚 Documentation Principle:**
- **AGENTS.md**: Core concepts, coding principles, and quick reference only
- **docs/**: Detailed architecture, design documents, and comprehensive guides
- When adding new features, put detailed docs in `docs/en/` and `docs/zh/`, reference from AGENTS.md

**Recent design docs:**
- Prompt caching optimization via dynamic context injection: [`docs/zh/developer-guide/dynamic-context.md`](docs/zh/developer-guide/dynamic-context.md)
- English version: [`docs/en/developer-guide/dynamic-context.md`](docs/en/developer-guide/dynamic-context.md)
- Exposing capabilities guide (how to make features available to AI agents): [`docs/zh/developer-guide/exposing-capabilities.md`](docs/zh/developer-guide/exposing-capabilities.md)
- English version: [`docs/en/developer-guide/exposing-capabilities.md`](docs/en/developer-guide/exposing-capabilities.md)

**📚 Documentation Writing Rules:**
- All documentation files MUST include frontmatter with `sidebar_position` for ordering:
  ```markdown
  ---
  sidebar_position: 1
  ---
  ```
- Document titles should NOT repeat the sidebar category name (e.g., use "概述" instead of "AI 编码" when under AI Coding category)
- Write Chinese docs first (`docs/zh/`), then create English versions (`docs/en/`)
- Use consistent heading hierarchy: `#` for title, `##` for sections, `###` for subsections

**📚 Detailed Documentation:** See `docs/en/` or `docs/zh/` for comprehensive guides on setup, testing, architecture, and user guides.

---

## 📖 Terminology: Team vs Bot

**⚠️ CRITICAL: Understand the distinction between code-level terms and UI-level terms.**

| Code/CRD Level | Frontend UI (Chinese) | Frontend UI (English) | Description |
|----------------|----------------------|----------------------|-------------|
| **Team** | **智能体** | **Agent** | The user-facing AI agent that executes tasks |
| **Bot** | **机器人** | **Bot** | A building block component that makes up a Team |

**Key Relationship:**
```
Bot = Ghost (提示词) + Shell (运行环境) + Model (AI模型)
Team = Bot(s) + Collaboration Mode (协作模式)
Task = Team + Workspace (代码仓库)
```

**Naming Convention:**
- API Routes / Database / Code: Use CRD names (`Team`, `Bot`)
- Frontend i18n values (zh-CN): Use UI terms (`智能体`, `机器人`)
- Frontend i18n values (en): Use `Agent`/`Team` and `Bot`

---

## 🧪 Testing

**Always run tests before committing.** Target coverage: 40-60% minimum.

**⚠️ Python modules use [uv](https://docs.astral.sh/uv/) for dependency management. Always use `uv run` to execute Python commands.**


**Test principles:**
- Follow AAA pattern: Arrange, Act, Assert
- Mock external services (Anthropic, OpenAI, Docker, APIs)
- Test edge cases and error conditions
- Keep tests independent and isolated

**E2E Testing Rules:**
- ⚠️ E2E tests MUST NOT fail gracefully - no `test.skip()`, no silent failures
- ⚠️ NO frontend mocking of backend APIs - send real HTTP requests
- If a test fails, FIX the issue - never skip to make CI pass

---

## 💻 Code Style

**⚠️ All code comments MUST be written in English.**

### General Principles

- **High cohesion, low coupling**: Each module/class should have a single responsibility
- **File size limit**: If a file exceeds **1000 lines**, split it into multiple sub-modules
- **Function length**: Max 50 lines per function (preferred)

### Code Design Guidelines

⚠️ **Follow these guidelines when implementing new features or modifying existing code:**

1. **Long-term maintainability over short-term simplicity**: When multiple implementation approaches exist, avoid solutions that are simpler to implement now but will increase maintenance costs in the long run. Choose the approach that balances implementation effort with long-term sustainability.

2. **Use design patterns for decoupling**: Actively consider applying design patterns (e.g., Strategy, Factory, Observer, Adapter) to decouple modules and improve code flexibility. This makes the codebase easier to extend and test.

3. **Manage complexity through extraction**: If a module is already complex, prioritize extracting common logic into utilities or creating new modules rather than adding more complexity to the existing module. When in doubt, split rather than extend.

4. **Reference, extract, then reuse**: Before implementing new functionality, always:
   - Search for existing implementations that solve similar problems

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wecode-ai/Wegent](https://github.com/wecode-ai/Wegent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
