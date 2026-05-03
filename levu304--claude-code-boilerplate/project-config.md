---
trigger: always_on
description: **Version:** {{VERSION}}
---

# {{PROJECT_NAME}} Coding Standards & Best Practices

**Version:** {{VERSION}}
**Last Updated:** {{DATE}}
**Architecture:** {{ARCHITECTURE_TYPE}}

---

## Quick Start

1. Copy this file to your project root as `CLAUDE.md`
2. Replace all `{{PLACEHOLDER}}` values with your project specifics
3. Delete language/framework sections that don't apply to your stack
4. Add project-specific patterns as needed

---

## Project Configuration

<!-- Fill these values to customize this template -->

| Setting | Value | Options |
|---------|-------|---------|
| **Language** | `{{LANGUAGE}}` | TypeScript, JavaScript, Python, Go, Rust |
| **Package Manager** | `{{PACKAGE_MANAGER}}` | pnpm, bun, npm, yarn, pip, poetry, cargo, go mod |
| **Backend Framework** | `{{BACKEND_FRAMEWORK}}` | Hono, Elysia, Express, Fastify, FastAPI, Gin, Axum |
| **Frontend Framework** | `{{FRONTEND_FRAMEWORK}}` | React, Vue, Svelte, Solid, None |
| **Database/ORM** | `{{DATABASE_ORM}}` | Drizzle, Prisma, SQLAlchemy, GORM, Diesel |
| **Testing Framework** | `{{TESTING_FRAMEWORK}}` | Vitest, Jest, pytest, go test, cargo test |
| **Linter/Formatter** | `{{LINTER_FORMATTER}}` | Biome, ESLint+Prettier, Ruff, golangci-lint, rustfmt |

---

## Table of Contents

1. [Technology Selection Guidelines](#1-technology-selection-guidelines)
2. [Naming Conventions](#2-naming-conventions)
3. [Language-Specific Standards](#3-language-specific-standards)
4. [Backend Patterns](#4-backend-patterns)
5. [Frontend Patterns](#5-frontend-patterns)
6. [Database Patterns](#6-database-patterns)
7. [Testing Standards](#7-testing-standards)
8. [SOLID Principles](#8-solid-principles)
9. [Security Best Practices](#9-security-best-practices)
10. [Error Handling](#10-error-handling)
11. [Performance Guidelines](#11-performance-guidelines)
12. [Documentation Standards](#12-documentation-standards)
13. [Git Workflow](#13-git-workflow)
14. [Issue Hierarchy](#14-issue-hierarchy)
15. [AI Agent Workflow](#15-ai-agent-workflow) - *"No Code, No Problem"*
    - [15.0 TDD-First Principle (ABSOLUTE REQUIREMENT)](#150-tdd-first-principle-absolute-requirement)
    - [15.0.1 Zero-Impact Implementation (ABSOLUTE REQUIREMENT)](#1501-zero-impact-implementation-absolute-requirement)
    - [15.4 Review Requirements (Score ≥ 9.5 to Merge)](#154-review-requirements-score-based-approval)
16. [Claude Code Configuration](#16-claude-code-configuration)

---

## 1. Technology Selection Guidelines

### Selection Criteria

| Priority | Criteria | Description |
|----------|----------|-------------|
| 1 | **Recency** | Prefer technologies released/updated within last 3 years |
| 2 | **Active Maintenance** | Active development, regular releases, responsive maintainers |
| 3 | **Community Adoption** | Growing community, good documentation, ecosystem support |
| 4 | **Type Safety** | First-class type support (TypeScript, type hints, generics) |
| 5 | **Performance** | Modern optimizations (tree-shaking, lazy loading, async) |

### Technology Evaluation Checklist

Before adopting any new technology:

- [ ] **Release Date**: From the last 3 years?
- [ ] **Last Update**: Updated within last 6 months?
- [ ] **Community Activity**: Growing or stable community?
- [ ] **Type Support**: Native types or excellent definitions?
- [ ] **Bundle/Binary Size**: Optimized for production?
- [ ] **Breaking Changes**: Stable API with clear migration paths?

### Preferred Modern Stack by Language

#### TypeScript/JavaScript (2023-2025)

| Category | Preferred | Avoid |
|----------|-----------|-------|
| Runtime | Bun, Deno, Node 20+ | Node < 18 |
| Backend | Hono, Elysia, Fastify | Express (legacy) |
| Frontend | React 18+, Solid, Svelte 5 | React < 17 |
| Meta Framework | TanStack Start, Next 14+, Nuxt 3 | CRA, Next < 13 |
| ORM | Drizzle, Prisma 5+ | Sequelize, TypeORM |
| Validation | Zod, Valibot | Joi, Yup |
| Testing | Vitest, Playwright | Jest (prefer Vitest) |
| Build | Vite, Turbopack, Rsbuild | Webpack |
| Package Manager | pnpm, Bun | npm, Yarn Classic |

#### Python (2023-2025)

| Category | Preferred | Avoid |
|----------|-----------|-------|
| Runtime | Python 3.11+ | Python < 3.9 |
| Backend | FastAPI, Litestar | Flask, Django (for APIs) |
| Async | asyncio, HTTPX | requests (sync only) |
| ORM | SQLAlchemy 2.0, SQLModel | SQLAlchemy < 2.0 |
| Validation | Pydantic v2 | Pydantic v1 |
| Testing | pytest, pytest-asyncio | unittest |
| Linting | Ruff, mypy | flake8, pylint |
| Package Manager | uv, poetry, pdm | pip alone |

#### Go (2023-2025)

| Category | Preferred | Avoid |
|----------|-----------|-------|
| Runtime | Go 1.21+ | Go < 1.18 |
| Backend | Gin, Echo, Fiber | net/http alone |
| ORM | GORM, sqlc, Ent | raw SQL everywhere |
| Validation | go-playground/validator | manual validation |
| Testing | testify, gomock | testing alone |
| Linting | golangci-lint | golint (deprecated) |

#### Rust (2023-2025)

| Category | Preferred | Avoid |
|----------|-----------|-------|
| Runtime | Rust 1.70+ | Rust < 1.60 |
| Backend | Axum, Actix-web 4 | Rocket (older) |
| Async | Tokio | async-std |
| ORM | SeaORM, Diesel | raw SQL |
| Serialization | serde | manual parsing |
| Testing | cargo test, mockall | - |
| Linting | clippy, rustfmt | - |

---

## 2. Naming Conventions

### 2.1 Files and Directories

| Type | Convention | Example |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [levu304/claude-code-boilerplate](https://github.com/levu304/claude-code-boilerplate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
