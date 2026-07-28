---
trigger: always_on
description: This file provides guidance to Claude Code when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with code in this repository.

## Claude Agent Instructions

Claude, you have access to skills that must be loaded based on the task at hand.
Read the relevant skill file(s) **before** writing code.

| Trigger                                                | Skill to read                                           |
| ------------------------------------------------------ | ------------------------------------------------------- |
| Before implementing, reviewing, or refactoring frontend code | `.claude/skills/frontend_specialist.md`            |
| Before writing components, forms, pages, or hooks      | `.claude/skills/coding_standards.md`                    |
| Before writing or reviewing any test file              | `.claude/skills/testing_guidelines.md`                  |
| Before coding with React, SWR, Vitest, Vite, or PF6   | `.claude/skills/library_references.md`                  |
| Before reviewing a pull request                        | `.claude/skills/pr_review.md`                           |
| When fixing SonarCloud issues                          | `.claude/skills/sonarcloud-remediation/sonarcloud-remediation.md` |

> **Enforced by hook:** A `PreToolUse` hook (`.claude/hooks/skill-gate.sh`) will **block** your first `Edit`/`Write` to each source file category and list the required skills. The file-to-skill mapping lives in [`.claude/skill-triggers.json`](.claude/skill-triggers.json) (single source of truth). Read the listed skills with the `Read` tool, then retry. This applies to all contributors automatically.

> **To add or change skill triggers:** Edit [`.claude/skill-triggers.json`](.claude/skill-triggers.json). The hook and this table should stay in sync -- the JSON is what the hook enforces at runtime.

### Common PR Mistakes Checklist

Before finishing any code task, verify:

1. No hardcoded API paths — use `awxAPI`/`edaAPI`/`hubAPI`/`gatewayAPI` tagged template helpers
2. Workspace PageForm wrappers used — `AwxPageForm`/`EdaPageForm`/`HubPageForm`/`PlatformPageForm`, not raw `PageForm`
3. Framework components checked first — search `/framework` before creating new components
4. MSW for API mocking in tests — mock at the HTTP level, not by mocking CRUD functions
5. `userEvent.setup()` in tests — never `fireEvent`
6. Accessible queries first — `getByRole` > `getByLabelText` > `getByText` > `getByTestId`
7. Never compare translated display strings in logic — compare raw API values
8. No `any` types — use proper types or `unknown` with type guards
9. `Readonly<Props>` on all component props
10. Quality gates pass — `npm run tsc && npm run eslint && npm run prettier:fix`

### Feature Preservation

Never remove existing features, routes, or components without explicit
instruction. If a task might affect existing functionality, confirm before
proceeding.

## Project Overview

This is the Ansible UI monorepo built with React, TypeScript, and PatternFly. The project uses NPM workspaces and is structured as a unified UI that integrates multiple services:

- **Platform** - Unified gateway UI for AAP (main entry point)
- **AWX** - AWX UI
- **EDA** - Event-Driven Ansible UI
- **Hub** - Automation Hub UI
- **Chatbot** - Ansible Virtual Assistant UI
- **Framework** - Shared UI framework using PatternFly
- **Common** - Shared components and utilities

## Running Tests

- **Unit/Component Tests**: `npm run vitest` (uses Vitest)
- **Playwright Integration Tests**: See Playwright Testing section below
- **Linting**: `npm run eslint`
- **Type Checking**: `npm run tsc`

## Architecture

### Monorepo Structure

- `/platform` - Main Platform UI (unified entry point)
- `/framework` - Shared UI framework
- `/frontend/awx` - AWX Controller UI
- `/frontend/eda` - Event-Driven Ansible UI
- `/frontend/hub` - Automation Hub UI
- `/frontend/chatbot` - Chatbot UI
- `/frontend/common` - Shared components
- `/playwright` - E2E tests

### Key Technologies

- **React 18** with TypeScript
- **PatternFly** for UI components
- **React Hook Form** for form management
- **React Router** for navigation
- **SWR** for data fetching
- **i18next** for internationalization
- **Vite** for build tooling
- **Vitest** for unit testing
- **Playwright** for E2E testing
- **NX** for monorepo management

### API Integration

Each service has its own API helper wrapper - use these instead of raw URLs:

- `gatewayAPI`/users/``- Platform:`/api/gateway/`
- `awxAPI`/projects/``- AWX:`/api/controller/v2/`
- `edaAPI`/events/``- EDA:`/api/eda/v1/`
- `hubAPI`/collections/``- Hub:`/api/galaxy/`

## Development Commands

### Essential Commands

```bash
# Setup and dependencies
npm ci                    # Install dependencies
npm run clean             # Clean build artifacts

# Testing and quality
npm test                  # Run all tests (TypeScript, ESLint, Prettier, Vitest)
npm run tsc               # Type checking
npm run fix               # Fix linting and formatting
npm run i18n              # Generate translations

# Playwright tests (from /playwright directory)
cd playwright && npm run live    # Run against live server
cd playwright && npm run mock    # Run against mocked data

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ansible/ansible-ui](https://github.com/ansible/ansible-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
