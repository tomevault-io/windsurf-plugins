---
trigger: always_on
description: > This file is the project's authoritative spec for collaborating with AI coding agents (Claude Code, Codex, Cursor, Aider, etc.) and humans. `CLAUDE.md` is a symlink to this file. Replace `{{PROJECT_NAME}}`, `{{WEB_DOMAIN}}`, `{{API_DOMAIN}}`, `{{VPS_IP}}`, and `{{REGISTRY}}` with real values when bootstrapping a new project from this template.
---

# {{PROJECT_NAME}} — Monorepo

> This file is the project's authoritative spec for collaborating with AI coding agents (Claude Code, Codex, Cursor, Aider, etc.) and humans. `CLAUDE.md` is a symlink to this file. Replace `{{PROJECT_NAME}}`, `{{WEB_DOMAIN}}`, `{{API_DOMAIN}}`, `{{VPS_IP}}`, and `{{REGISTRY}}` with real values when bootstrapping a new project from this template.

## Project Overview

A monorepo with a decoupled frontend and backend, fully dockerized for local development and deployed via Kamal 2 to a single VPS.

## Tech Stack

| Layer    | Technology                                          |
| -------- | --------------------------------------------------- |
| Frontend | Vite + React + TypeScript + TanStack Router & Query |
| UI       | Tailwind CSS + shadcn/ui (Radix UI)                 |
| Backend  | Ruby on Rails 8 (API mode)                          |
| Auth     | Rails 8 built-in authentication (session cookies)   |
| Database | PostgreSQL                                          |
| Jobs     | Solid Queue                                         |
| API      | Versioned REST (`/api/v1/`)                         |
| Monorepo | pnpm workspaces + Docker Compose (local)            |
| Deploy   | Kamal 2 → VPS                                       |
| Testing  | RSpec (backend) + Vitest (frontend)                 |
| CI       | GitHub Actions                                      |
| Linting  | RuboCop + Brakeman + bundler-audit + ESLint         |

## Monorepo Structure

```
{{PROJECT_NAME}}/
├── apps/
│   ├── web/                  # Frontend - Vite + React  (see apps/web/AGENTS.md)
│   └── api/                  # Backend  - Rails 8 API   (see apps/api/AGENTS.md)
├── .github/workflows/ci.yml  # GitHub Actions CI pipeline
├── docker-compose.yml        # Local dev orchestration
├── pnpm-workspace.yaml
├── package.json
└── AGENTS.md                 # ← you are here (CLAUDE.md is a symlink)
```

## Per-App Guidance

When working inside a specific app, read its local `AGENTS.md` for app-specific patterns:

- **`apps/api/AGENTS.md`** — Rails patterns: service objects, controllers, Blueprinter serializers, RSpec.
- **`apps/web/AGENTS.md`** — React patterns: feature-based organization, TanStack Query, components, Vitest.

This root file covers everything cross-cutting (architecture, auth strategy, Docker, CI, deploy, git workflow).

Per-app contexts are auto-loaded from this file so they are available regardless of which directory the agent session starts in:

@apps/api/AGENTS.md
@apps/web/AGENTS.md

## Language

All code, comments, variable names, commit messages, branch names, documentation, and API responses MUST be written in English — regardless of the language used in conversation. This is a strict standard with no exceptions.

## Architecture & Design Principles

### SOLID (as a guide, not rigid dogma)

Apply SOLID principles pragmatically. They should improve code clarity and maintainability, not add unnecessary abstraction.

- **S - Single Responsibility**: Each class/module/component does one thing well. A service handles one business operation. A component renders one UI concern.
- **O - Open/Closed**: Prefer extending behavior over modifying existing code. Use composition in React, and inheritance/modules in Ruby when it makes sense.
- **I - Interface Segregation**: Keep interfaces small and focused. In Rails, don't bloat models with unrelated concerns. In React, keep prop interfaces tight.
- **L - Liskov Substitution**: Subclasses should be interchangeable with their parent. In practice: don't override methods in ways that break expectations.
- **D - Dependency Inversion**: Depend on abstractions when beneficial. Services receive dependencies instead of hardcoding them — but don't over-engineer simple cases.

**Rule of thumb**: If applying a SOLID principle adds more complexity than it removes, skip it. Three simple lines beat one clever abstraction.

### Development Workflow: Test-Driven Development (TDD)

Every new feature must start with tests before writing implementation code. Follow the Red-Green-Refactor cycle:

1. **Red** — Write failing tests that define the expected behavior
2. **Green** — Write the minimum code to make the tests pass
3. **Refactor** — Clean up the code while keeping tests green

**In practice:**

For a backend feature (e.g., new endpoint):

1. Write request specs (`spec/requests/api/v1/`) defining the API contract
2. Write service specs (`spec/services/`) defining the business logic
3. Write model specs if new models/validations are needed
4. Implement the code to make all tests pass

For a frontend feature (e.g., new page/component):

1. Write component/integration tests with Vitest + Testing Library
2. Implement the component to make tests pass

**Never skip this order.** Tests are not an afterthought — they define the feature.

### General Coding Guidelines

- Write simple, readable code. Prefer clarity over cleverness.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jcflorville/rails-tanstack-starter](https://github.com/jcflorville/rails-tanstack-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
