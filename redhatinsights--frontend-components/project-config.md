---
trigger: always_on
description: This is a monorepo of Red Hat Cloud Services components for React.js applications on console.redhat.com. It provides reusable UI components, utilities, and tooling for platform services.
---

# Frontend Components - Monorepo

## Repository Overview

This is a monorepo of Red Hat Cloud Services components for React.js applications on console.redhat.com. It provides reusable UI components, utilities, and tooling for platform services.

### Architecture

- **Build System**: Nx workspace with independent package versioning
- **Framework**: React 18 with TypeScript
- **Component Library**: PatternFly 6
- **Testing**: Cypress (component), Jest (unit)
- **Release**: Automated via conventional commits

### Packages

- `components` - Common UI components
- `utils` - Utility functions and helpers
- `chrome` - Console chroming framework
- `notifications` - Toast notification system
- `remediations` - Remediation wizard components
- `advisor-components` - Advisor-specific components
- `rule-components` - Rule display components
- `translations` - i18n utilities
- `testing` - Testing utilities
- `config-utils` - Build configuration helpers
- `eslint-config` - Shared ESLint configuration

## Development Setup

### Prerequisites

Node and npm versions are defined in `package.json` (engines field) and `.nvmrc`.

### Installation

```bash
npm install
```

### Local Development

Start the demo sandbox (linked to all packages):

```bash
npm run serve:demo
# Open http://localhost:4200/
```

Demo source: `examples/demo/src/app/app.tsx` (do not commit changes)

## Code Conventions

### TypeScript

- Strict mode enabled
- JSX transform: `react-jsx`
- Target: ES5, module: CommonJS
- Path aliases: Use `@redhat-cloud-services/*` imports

### Linting

- ESLint with Prettier integration
- Import sorting enforced
- TypeScript strict rules
- React hooks validation

### Git Workflow

- **Commit Format**: [Conventional Commits](https://www.conventionalcommits.org/)
  - `feat:` new features
  - `fix:` bug fixes
  - `chore:` maintenance
  - `docs:` documentation
- **Requirements**:
  - 2FA enabled on GitHub account
  - GPG-signed commits mandatory
- **Husky**: Pre-commit hooks run lint/tests (use `--no-verify` to skip locally, not in CI)

### Commit Message & Release Rules (Important for AI Tools)

Releases are **fully automated** — the commit type directly controls whether packages are published to npm. Choose the commit type carefully:

- **`feat:`** → triggers a **minor** version bump for the scoped package (with `useCommitScope: true`, only the package whose scope matches the commit scope gets the full bump; changed dependents are capped at **patch**)
- **`fix:`** → triggers a **patch** version bump for the scoped package (same scope-filtering applies)
- **`<type>!:` (any type with `!`) or `BREAKING CHANGE:` footer** → triggers a **major** version bump for the scoped package
- **`chore:`, `docs:`, `ci:`, `refactor:`, `test:`** → **no release triggered**

**Rules when committing:**
1. Use `feat:` when implementing a new feature or enhancement that changes package source code — this is expected and will correctly trigger an npm release.
2. Use `fix:` when fixing a bug in package source code.
3. Use `chore:` or `docs:` for changes that do **not** affect package source code — documentation (`CLAUDE.md`, `README.md`), CI config, linting config, or tooling setup — even if the files live inside a `packages/*/` directory.
4. If a commit includes both source code changes and non-code changes (e.g., a new feature plus updated docs), use `feat:` or `fix:` — the release is warranted by the code change.
5. **Commit scope rules** (`useCommitScope: true` in `nx.json`):
   - For **`feat` and breaking change commits** (`!` marker or `BREAKING CHANGE:` footer): scope **must** be the full Nx project name (e.g., `feat(@redhat-cloud-services/frontend-components-utilities): ...`). Short scopes like `feat(utils):` or area scopes like `feat(deps):` will be rejected by commitlint.
   - For **`fix` commits**: any scope or no scope is fine (e.g., `fix(deps): ...`, `fix(ci): ...`, `fix: ...`). All `fix` commits result in patch bumps regardless of scope.
   - For **non-versioning commits** (`chore`, `docs`, `ci`, `refactor`, `test`): any scope or no scope is fine (e.g., `chore(utils): ...`, `chore(deps): ...`).

### Package Dependency Updates (config ↔ config-utils)

When changes to `config-utils` are required by `config` package:

1. **Initial PR**: Release both packages together (commit triggers releases for both)
2. **Follow-up PR**: Pin config-utils version in `packages/config/package.json` after release

**Note**: Don't pin version in initial PR — CI install happens before release, causing npm install failure for unreleased versions.

### Package Publishing (.npmignore)

All packages use a **single root `.npmignore` file** that is automatically copied to each package's `dist/` folder during build. This ensures consistent npm package contents across all packages.

**How it works:**
1. Root `.npmignore` defines patterns for files to exclude from published packages (test files, source maps, etc.)
2. Each package's `project.json` includes: `"assets": [{ "input": ".", "glob": ".npmignore", "output": "." }]`
3. On build, Nx copies root `.npmignore` → `dist/@redhat-cloud-services/{package}/.npmignore`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RedHatInsights/frontend-components](https://github.com/RedHatInsights/frontend-components) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
