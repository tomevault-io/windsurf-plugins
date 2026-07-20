---
trigger: always_on
description: This document is the **primary authoritative source** of context for all AI assistants (Claude, GitHub Copilot, Codeium, Continue.dev, Junie, etc.) contributing to OWASP Juice Shop. It provides comprehensive guidelines to maintain code quality, security, and adherence to project standards.
---

# AI Agent Guidelines for OWASP Juice Shop

This document is the **primary authoritative source** of context for all AI assistants (Claude, GitHub Copilot, Codeium, Continue.dev, Junie, etc.) contributing to OWASP Juice Shop. It provides comprehensive guidelines to maintain code quality, security, and adherence to project standards.

## Project Overview

- **Project**: OWASP Juice Shop - an intentionally insecure web application for security training
- **Primary Languages**: TypeScript, JavaScript, Angular (frontend)
- **Key Technologies**: Node.js (22–25 with 24 being the default), Express, SQLite/Sequelize, MongoDB/MarsDB, Angular 21.x
- **Testing**: Node.js built-in test runner (server unit tests), Supertest (API integration), Vitest (frontend unit tests), Cypress (E2E tests)
- **Code Style**: JS Standard Style (enforced via ESLint)
- **Repository**: [juice-shop/juice-shop](https://github.com/juice-shop/juice-shop)

## Key Files and Directories

- `app.ts` / `server.ts` - Application entry points
- `lib/` - Utility functions and libraries (including `lib/startup/` for initialization)
- `routes/` - Express route handlers
- `models/` - Sequelize data models (SQLite)
- `data/` - Data creation and management (`data/static/` for challenges, users, codefixes)
- `views/` - Server-rendered templates (Handlebars `.hbs` and Pug `.pug`)
- `test/server/` - Server unit tests (Node.js built-in test runner)
- `test/api/` - API integration tests (Node.js built-in test runner + Supertest)
- `frontend/src/` - Angular frontend code (tests use Vitest)
- `cypress/` - E2E tests (Cypress)
- `rsn/` - Refactoring Safety Net scripts and cache
- `config/` - Configuration files (YAML, multiple themed configs like `ctf.yml`, `default.yml`)
- `i18n/` - Internationalization files (do NOT modify directly)
- `ftp/` - Files served via the simulated FTP directory
- `monitoring/` - Grafana dashboard config
- `.github/workflows/` - CI/CD pipelines
- `encryptionkeys/` - Encryption key files

## Important Constraints

1. **Security Context**: This project contains intentional vulnerabilities for training. New vulnerabilities must be approved by maintainers and well-documented.
2. **Challenge Development**: Consult maintainers before creating new challenges. AI-generated challenges risk being duplicate, unsolvable, or dysfunctional.
3. **Code Changes and RSN**: When modifying challenge-related code, the Refactoring Safety Net must pass.
4. **Dependency Updates**: Verify compatibility with `package.json` and `frontend/package.json`.
5. **Translation Modifications**: Use [Crowdin](https://crowdin.com/project/owasp-juice-shop), not direct file editing.

## Recommended Use Cases

### ✅ Good Use Cases

- **Code Analysis**: Understanding existing code structure and patterns
- **Refactoring**: Improving code quality while maintaining functionality
- **Test Writing**: Creating unit, integration, and e2e tests
- **Bug Fixing**: Identifying and resolving issues
- **Documentation**: Writing clear comments and documentation

### ⚠️ Use with Caution

- **Challenge Development**: Consult with maintainers before creating new challenges.
- **Security Vulnerabilities**: Ensure AI-suggested vulnerabilities are intentional and appropriate for the project.
- **Dependencies**: Verify any suggested package updates for compatibility.
- **Architecture Changes**: Discuss major structural changes with maintainers first.

## Essential Guidelines

### 1. Clean Up AI-Generated Noise

**Required** per CONTRIBUTING.md rule #6: Remove unnecessary AI-generated content before submitting PRs.

**Remove**:
- Verbose comments explaining obvious code
- Generic placeholder comments
- Overly detailed docstrings for simple functions
- Repetitive explanations, `console.log` statements

**Keep**:
- Meaningful comments for complex logic
- Challenge hints and metadata
- Security-relevant documentation

### 2. Code Style Compliance

Always run ESLint before committing (unless only `REFERENCES.md` or `SOLUTIONS.md` were modified):
```bash
npm run lint
```
The AI should suggest code following [JS Standard Style](http://standardjs.com/), but always verify.

### 3. Testing Requirements

For any code changes (unless only `REFERENCES.md` or `SOLUTIONS.md` were modified):
- **Unit/Integration Tests**: New features and changes should have tests.
- **E2E Tests**: Required for new/modified challenges.
- **RSN (Refactoring Safety Net)**: Required when modifying existing code that is part of a coding challenge (see the [verify-rsn-fix skill](./.ai/skills/verify-rsn-fix/SKILL.md) for details).
- **Run Tests Locally**:
  ```bash
  npm test                    # Runs frontend, server, and api tests
  npm run test:frontend       # Frontend unit tests (Vitest)
  npm run test:server         # Server unit tests only (Node.js built-in test runner)
  npm run test:api            # API integration tests (Node.js built-in test runner + Supertest)
  npm start & npm run test:e2e  # E2E tests (Cypress)
  npm run rsn                 # Refactoring Safety Net
  ```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [juice-shop/juice-shop](https://github.com/juice-shop/juice-shop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
