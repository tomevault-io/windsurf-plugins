---
trigger: always_on
description: This document provides guidance for AI agents working on the Open Data Hub (ODH) Dashboard monorepo.
---

# AGENTS.md - ODH Dashboard

This document provides guidance for AI agents working on the Open Data Hub (ODH) Dashboard monorepo.

## Repository Overview

ODH Dashboard is a **monorepo** containing the main dashboard application and multiple feature packages. It provides the web UI for Red Hat OpenShift AI (RHOAI) and Open Data Hub.

## Repository Structure

```text
odh-dashboard/
├── frontend/                    # Main dashboard frontend application
│   └── src/
│       └── __mocks__/          # Shared mock data (@odh-dashboard/internal/__mocks__)
├── backend/                     # Main dashboard backend (Node.js/Express)
│   └── src/
├── dashboard-operator/          # Dashboard Module Controller (Go, controller-runtime)
│   ├── api/v1alpha1/           # CRD types (Dashboard kind)
│   ├── cmd/manager/            # Controller entry point
│   ├── internal/controller/    # Reconciler, actions, support utilities
│   └── config/                 # Generated CRD, RBAC, manager manifests
├── distributions/               # Independently-deployable dashboard variants
│   ├── base/                    # Shared app shell library (not deployed on its own)
│   ├── core-bff/                # Full Go BFF + React frontend (has BFF)
│   └── rhaii/                   # RHAII-specific distribution (frontend-only)
├── packages/                    # Feature packages
│   ├── cypress/                # Cypress test framework and shared tests
│   ├── gen-ai/                 # Gen AI / LLM features (has BFF)
│   ├── maas/                   # Mod Arch starter (has BFF)
│   ├── model-registry/         # Model Registry UI (has BFF)
│   ├── model-serving/          # Model Serving UI
│   └── ...                     # Other packages
├── .github/                    # GitHub workflows and templates
├── .tekton/                    # Tekton CI/CD pipelines
└── docs/                       # Documentation
```

## Development Requirements

- **Node.js**: >= 22.0.0
- **npm**: >= 10.0.0
- **Go**: >= 1.26 (for packages with BFF), >= 1.25 (for dashboard-operator)

## Key Technologies

| Technology    | Purpose                                    |
| ------------- | ------------------------------------------ |
| React 18      | Frontend framework                         |
| TypeScript    | Type safety                                |
| PatternFly v6 | Primary UI component library (RHOAI/ODH)   |
| Material UI   | Secondary UI library (Kubeflow mode)       |
| Webpack       | Build tooling with Module Federation       |
| Cypress       | E2E and component testing                  |
| Jest          | Unit testing                               |
| Turbo         | Monorepo task runner                       |

## Common Commands

```bash
# Install dependencies
npm install

# Start development server (main dashboard)
npm run dev

# Build all packages
npm run build

# Run tests
npm run test

# Lint all packages
npm run lint
npm run lint:fix

# Type checking
npm run type-check
```

## Documentation

**[BOOKMARKS.md](BOOKMARKS.md)** indexes key documentation for frontend areas, the backend, and packages. Review relevant docs for the area you are working on before starting a task.

**[Multi-Agent Workflows](docs/multi-agent-workflows.md)** covers running parallel agent sessions locally (git worktrees, Agent Teams, terminal layout) and remotely (Ambient platform).

## Package-Specific Guidelines

Some packages have their own AGENTS.md with package-specific guidance. Check the package directory for its own AGENTS.md file.

## Specialized Agent Rules

Before performing certain tasks, read and follow the corresponding specialized rules.

Rules live in `.claude/rules/`. Read the relevant rule file before starting the task.

| Rule                        | File                          | Trigger                                                                        |
| --------------------------- | ----------------------------- | ------------------------------------------------------------------------------ |
| **Architecture**            | `architecture.md`             | When making structural changes, adding packages, modifying package boundaries, or working on distributions |
| **BFF Go**                  | `bff-go.md`                   | When working on Go BFF code in `packages/*/bff/` or `distributions/core-bff/bff/` |
| **Contract Tests**          | `contract-tests.md`           | When working on contract tests or BFF API validation                           |
| **Conventions**             | `conventions.md`              | When writing or reviewing TypeScript, React, or backend code                   |
| **CSS & PatternFly**        | `css-patternfly.md`           | When writing or modifying styles, SCSS, or PatternFly components               |
| **Distributions**           | `distributions.md`            | When working on code in `distributions/`                                       |
| **Cypress E2E Tests**       | `cypress-e2e.md`              | When creating or modifying E2E tests, Robot Framework migrations               |
| **Cypress Mock Tests**      | `cypress-mock.md`             | When creating or modifying mock/component tests                                |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [opendatahub-io/odh-dashboard](https://github.com/opendatahub-io/odh-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
