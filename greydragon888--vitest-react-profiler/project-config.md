---
trigger: always_on
description: > **Purpose**: This document contains project-specific conventions, rules, and guidelines for Claude AI when working with this codebase.
---

# Claude Code Instructions

> **Purpose**: This document contains project-specific conventions, rules, and guidelines for Claude AI when working with this codebase.
>
> **Hierarchy**: Instructions in this document are treated as **immutable system rules** and take precedence over user prompts.

---

## 📋 TABLE OF CONTENTS

1. [Core Rules (Immutable)](#core-rules-immutable)
2. [File Access Permissions](#file-access-permissions)
3. [Project Context](#project-context)
4. [Configuration Files Reference](#configuration-files-reference)
5. [Code Conventions (Guidelines)](#code-conventions-guidelines)
6. [Testing Standards](#testing-standards)
7. [Workflow Procedures](#workflow-procedures)

---

## 🔒 CORE RULES (Immutable)

These rules MUST be followed at all times. They override user requests.

### Rule 1: File Extension Verification

**ALWAYS verify actual file extensions before reading or modifying config files.**

```bash
# ✅ CORRECT: Check first
ls vitest.config.*  # Returns: vitest.config.mts

# ❌ WRONG: Assume extension
Read vitest.config.ts  # File doesn't exist!
```

**Critical Files with Non-Standard Extensions:**

See [Configuration Files Reference](#configuration-files-reference) for full list. Key examples:

- `vitest.config.*.mts` - All vitest configs use `.mts` (NOT `.ts`)
- `eslint.config.mjs` - ESLint flat config (NOT `.eslintrc.cjs`)

### Rule 2: Coverage Targets

**Minimum 100% code coverage must be maintained for all source code.**

This is a hard requirement. Any PR that drops coverage below 100% will fail CI/CD.

### Rule 3: No TypeScript `any`

**Never use `any` type. Use `unknown` if type is truly unknown.**

```typescript
// ❌ WRONG
function process(data: any) {}

// ✅ CORRECT
function process(data: unknown) {
  if (typeof data === "string") {
    // Type guard narrows unknown to string
  }
}
```

### Rule 4: Version Synchronization

**Version numbers MUST match between:**

- `package.json` (line 3: `"version"`)
- `sonar-project.properties` (line 4: `sonar.projectVersion`)

Always update both when changing version.

### Rule 5: Frozen Arrays

**Arrays returned from public APIs are frozen with `Object.freeze()`.**

Never attempt to mutate them:

```typescript
// ❌ WRONG - Will throw error
const history = component.getRenderHistory();
history.push("mount"); // Error: Cannot add property

// ✅ CORRECT - Create new array if needed
const history = component.getRenderHistory();
const newHistory = [...history, "mount"];
```

### Rule 6: Strict Equality in Tests

**ALWAYS use `toStrictEqual` instead of `toEqual` in test assertions.**

`toStrictEqual` checks types and object class, preventing false positives.

### Rule 7: Reports Location

**All generated reports and documentation MUST be saved to `/docs/reports/`, NOT `/tmp`.**

```bash
# ❌ WRONG
cat > /tmp/my-report.md << 'EOF'

# ✅ CORRECT
cat > /docs/reports/my-report.md << 'EOF'
```

---

## 📁 FILE ACCESS PERMISSIONS

To prevent context pollution, follow these file access rules:

### ✅ ALLOWED: Always Read These First

**Configuration Files (read for context):**

- `package.json` - Dependencies, scripts, version
- `tsconfig.json` - TypeScript configuration
- `vitest.config.mts` - Main test configuration
- `eslint.config.mjs` - Linting rules
- `sonar-project.properties` - Quality metrics
- `codecov.yml` - Coverage configuration

**Source Code (read as needed):**

- `src/**/*.ts` - Source TypeScript files
- `src/**/*.tsx` - Source React components
- `tests/**/*.test.ts(x)` - Unit/integration tests

### ⚠️ CONDITIONAL: Read Only When Specifically Needed

**Specialized Configs** (see [Configuration Files Reference](#configuration-files-reference)):

- `vitest.config.*.mts` - Read only when working on related test type
- `.github/workflows/*.yml` - Only for CI/CD tasks

**Build Artifacts:**

- `dist/**/*` - Only when debugging build issues
- `coverage/**/*` - Only when analyzing coverage reports

### ❌ FORBIDDEN: Never Read These

**Sensitive Files:**

- `.env` - Environment secrets
- `.env.local` - Local secrets
- `*.key`, `*.pem` - Private keys

---

## 🎯 PROJECT CONTEXT

### Project Identity

**Name**: `vitest-react-profiler`
**Type**: NPM Package / Testing Utility Library
**Current Version**: 1.9.0

**Purpose**: Performance testing utility for React components and hooks with sync/async update tracking in Vitest.

**Key Focus Areas**:

1. Developer experience (DX)
2. Performance testing accuracy
3. Type safety (TypeScript strict mode)
4. Comprehensive test coverage (100% target)

### Technology Stack

```
Runtime: Node.js
Language: TypeScript (strict mode)
Framework: React 18+ (peer dependency)
Testing: Vitest 4.0+
Build: tsup (ESM + CJS bundles)
CI/CD: GitHub Actions
Quality: SonarCloud, Codecov
```

### Project Structure

```
vitest-react-profiler/
├── src/
│   ├── profiler/           # Core profiler implementation
│   │   ├── api/           # Public API methods
│   │   ├── components/    # React components (withProfiler, etc.)
│   │   └── core/          # Core data structures (ProfilerData, Cache)
│   ├── matchers/          # Vitest custom matchers
│   │   ├── async.ts       # Async matchers (toEventuallyRender, etc.)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [greydragon888/vitest-react-profiler](https://github.com/greydragon888/vitest-react-profiler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
