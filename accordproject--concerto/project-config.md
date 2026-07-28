---
trigger: always_on
description: This repository is **Concerto** - a lightweight schema language and runtime for business concepts, maintained by the Accord Project under the Linux Foundation. Concerto provides object-oriented modeling capabilities with powerful serialization, validation, and code generation features.
---

# GitHub Copilot Instructions for Concerto

This repository is **Concerto** - a lightweight schema language and runtime for business concepts, maintained by the Accord Project under the Linux Foundation. Concerto provides object-oriented modeling capabilities with powerful serialization, validation, and code generation features.

## Project Architecture

### Tech Stack
- **Language**: JavaScript (ES6+) transitioning to TypeScript
- **Runtime**: Node.js 18+ with npm 10+
- **Build System**: npm workspaces with interdependent packages
- **Testing**: Mocha, Chai, Chai-as-promised, Sinon, Istanbul (99% coverage target)
- **Linting**: ESLint with strict configuration
- **Documentation**: JSDoc (required for all public APIs)
- **CI/CD**: GitHub Actions with multi-platform testing (Linux, Windows, macOS)

### Monorepo Structure
```
concerto/
├── packages/
│   ├── concerto-core/        # Core model management, parsing, validation, serialization
│   ├── concerto-util/         # Utility functions (independent from Concerto)
│   ├── concerto-cto/          # Parser for .cto syntax files
│   ├── concerto-vocabulary/   # Model vocabularies and localization
│   ├── concerto-analysis/     # Model comparison tools
│   ├── concerto-types/        # TypeScript type definitions
│   └── concerto-linter/       # Linting rules for Concerto models
└── scripts/                   # Build and maintenance scripts
```

### Package Dependencies
- **Build Order Critical**: Must run `npm run build` before `npm test` due to workspace interdependencies
- **Example**: `concerto-linter` depends on compiled `concerto-core`
- **Version Pinning**: All workspace packages use exact versions (enforced by `syncpack`)

---

## Critical Requirements

### 1. Developer Certificate of Origin (BLOCKING - ABSOLUTE)

**Every commit MUST include DCO sign-off. PRs cannot be merged without this.**

```bash
# Use one of these for EVERY commit:
git commit --signoff -m "feat(scope): description"
git commit -s -m "fix(scope): description"

# Configure alias for convenience:
git config alias.c 'commit --signoff'
```

**Format Added to Commit**:
```
Signed-off-by: Your Name <your.email@example.com>
```

**Why**: This is a Linux Foundation requirement for all contributions. Missing DCO sign-off will block PR merging.

---

### 2. Commit Message Format (STRICTLY ENFORCED)

**Required Format** (from [Accord Project conventions](https://github.com/accordproject/techdocs/blob/master/DEVELOPERS.md#commit-message-format)):

```
type(scope): description

[optional body explaining the WHY, not the what]

Signed-off-by: Your Name <your.email@example.com>
```

**Types** (from actual merged PRs):
- `feat`: New features (e.g., "feat(concerto-core): migrate package to typescript")
- `fix`: Bug fixes (e.g., "fix(concerto-util): restore legacy constructor argument order")
- `docs`: Documentation changes (e.g., "docs(concerto-analysis): fix incorrect npm install command")
- `chore`: Build/tooling (e.g., "chore(deps): add dependency linting and pin versions")
- `ci`: CI/CD workflows (e.g., "ci: upgrade outdated GitHub Actions in publish workflow")
- `test`: Test additions/modifications
- `refactor`: Code restructuring without behavior changes
- `style`: Formatting changes (whitespace, semicolons, etc.)

**Scopes**: Package name or area affected:
- Package names: `concerto-core`, `concerto-util`, `concerto-cto`, `concerto-vocabulary`, `concerto-analysis`, `concerto-linter`, `concerto-types`
- Areas: `deps`, `ci`, `docs`, `build`

**Key Principles**:
- ✅ Lowercase type and scope
- ✅ Imperative mood: "add feature" not "added feature"
- ✅ No period at end of subject line
- ✅ Explain WHY in body for non-obvious changes
- ❌ Never: "update readme", "fix stuff", "WIP"

**Examples from Merged PRs**:
```
✅ feat(concerto-core): migrate package to typescript
✅ fix(concerto-util): restore legacy constructor argument order
✅ docs(concerto-analysis): fix incorrect npm install command
✅ ci: upgrade outdated GitHub Actions in publish workflow
✅ chore(deps): add dependency linting and pin versions
❌ update readme (missing type, scope, DCO)
❌ Add new feature (missing scope, DCO, wrong case)
```

---

### 3. Testing Requirements (VITAL - 99% Coverage Target)

**Coverage Standards** (enforced in nyc configuration):
- Statements: 99%
- Branches: 97%
- Functions: 98%
- Lines: 99%

**Test Requirements by Change Type**:
- **New Features**: Must have comprehensive unit tests
- **Bug Fixes**: Must include regression tests demonstrating the fix
- **TypeScript Migrations**: Must maintain or improve existing coverage
- **Refactoring**: All existing tests must continue to pass

**Build-Then-Test Pattern (CRITICAL)**:
```bash
# ALWAYS build before testing in workspace repos:
npm run build    # Compile all packages
npm test         # Run all tests

# Or for specific package:
npm run build
npm test -w @accordproject/concerto-core
```

**Why**: Workspace packages depend on compiled versions of other packages. Tests will fail if dependencies aren't built first.

**Test Framework Patterns**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [accordproject/concerto](https://github.com/accordproject/concerto) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
