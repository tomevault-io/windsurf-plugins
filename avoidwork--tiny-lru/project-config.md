---
trigger: always_on
description: Rules and principles for agents working on **this** project.
---

# AGENTS.md

Rules and principles for agents working on **this** project.

---

## 1. Core Rules

### 1.0 Document Conventions

When updating this document, append new information or sections. Do NOT delete or overwrite existing content unless explicitly directed. Always ask before making structural changes. When in doubt, keep it.

### 1.1 Forbidden Patterns

The following are **strictly prohibited**:

- Hardcoded secrets, API keys, or credentials.
- `eval()`, `exec()` at any level.
- Star imports (`import * as foo from 'bar'`).
- Mutating a list while iterating over it.
- `new Array()` — use `Array()` or `Array.from()` instead (oxlint will warn).

### 1.2 Security Rules

Follow the [OWASP Top 10](https://owasp.org/www-project-top-10/) for every piece of code written:

- Avoid unsafe operations, validate inputs, prevent injection risks.

### 1.3 Git Operations

- **Never rebase under any circumstance without explicit agreement from the user.** Never assume your decision is correct.
- Never force push.

### 1.4 Core Principles

- **DRY (Don't Repeat Yourself)**: Extract common logic into reusable functions; avoid duplication.
- **YAGNI (You Ain't Gonna Need It)**: Implement only what's needed; avoid over-engineering.
- **SOLID**: Follow single responsibility, open/closed, and interface segregation principles.

---

## 2. Project Context

### 2.0 Expected Project Layout

`tiny-lru` is a high-performance, lightweight LRU (Least Recently Used) cache library for JavaScript with O(1) operations and optional TTL support.

```
├── src/lru.js          # Main LRU cache implementation
├── tests/              # Test files
├── benchmarks/         # Performance benchmarks
├── dist/               # Built distribution files
│   ├── tiny-lru.js     # ES Modules
│   ├── tiny-lru.cjs    # CommonJS
│   └── tiny-lru.min.js # Minified ESM
├── types/              # TypeScript definitions
├── docs/               # Documentation
├── rollup.config.js    # Build configuration
└── package.json        # Project configuration
```

### 2.1 Quick Commands

| Command           | Purpose                                           |
|-------------------|---------------------------------------------------|
| `npm install`     | Install dependencies                              |
| `npm run build`   | Lint and build (runs lint then rollup)            |
| `npm run rollup`  | Build with rollup                                 |
| `npm run test`    | Run lint and tests                                |
| `npm run coverage`| Run tests with coverage reporting                 |
| `npm run fix`     | Fix linting and formatting issues                 |
| `npm run lint`    | Lint code with oxlint                             |

---

## 3. Code Conventions

### 3.1 Language & Tooling

- **JavaScript/TypeScript**: CommonJS, ESM, and minified ESM outputs via Rollup
- **Package manager**: `npm`
- **Linting**: `oxlint`
- **Formatting**: (via `npm run fix`)
- **Testing**: Node.js built-in test runner (`node --test`)

### 3.2 Style

- Indentation: Tabs
- Quotes: Double quotes
- Semicolons: Required
- Array constructor: Avoid `new Array()` (oxlint will warn)

### 3.3 Error Handling

Not applicable — internal LRU operations should not surface errors; handle gracefully.

### 3.4 Testing

- Framework: Node.js built-in test runner (`node --test`)
- Tests: 149 tests across 26 suites
- Coverage: 100% lines, 99.28% branches, 100% functions
- Test pattern: `tests/**/*.js`
- All tests must pass with 100% line coverage before merging
- Run: `npm test` (lint + tests) or `npm run coverage` for coverage report

---

## 4. API Conventions

### 4.1 API Reference

- `lru(max, ttl, resetTtl)` - Factory function to create cache
- `LRU` class - Direct instantiation with `new LRU(max, ttl, resetTtl)`
- Core methods: `get()`, `set()`, `peek()`, `delete()`, `has()`, `clear()`, `evict()`
- Array methods: `keys()`, `values()`, `entries()`
- Utility methods: `forEach()`, `getMany()`, `hasAll()`, `hasAny()`, `cleanup()`, `toJSON()`, `stats()`, `onEvict()`, `sizeByTTL()`, `keysByTTL()`, `valuesByTTL()`
- Properties: `first`, `last`, `max`, `size`, `ttl`, `resetTtl`
- `peek(key)` - Retrieve value without moving it (no LRU update, no TTL check)

---

## 5. Git Conventions

### 5.1 Commit Messages

Follow [Conventional Commits](https://www.conventionalcommits.org/):

```
feat: add TTL support to LRU cache
fix: correct LRU eviction order for edge case
docs: update AGENTS.md with new conventions
test: add coverage for keysByTTL method
chore: update rollup configuration
```

### 5.2 Branching

- Main branch is `main`.
- Feature branches: `feat/<short-desc>` or `fix/<short-desc>`.
- Never commit directly to `main`. Always create a feature branch first, then open a PR targeting `main`.

### 5.3 Code Review

- All changes require tests to pass and maintain coverage requirements.
- 100% line coverage is required before merging.

### 5.4 Pull Request Templates

If a `.github/PULL_REQUEST_TEMPLATE.md` file exists, it MUST be used when creating PRs. Fill out every section — do not leave any section blank. If a section does not apply, write `N/A` rather than skipping it.

---

## 6. Operational Rules

### 6.1 Coverage


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [avoidwork/tiny-lru](https://github.com/avoidwork/tiny-lru) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
