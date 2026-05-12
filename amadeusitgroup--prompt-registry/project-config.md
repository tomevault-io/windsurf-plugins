---
trigger: always_on
description: These are short, actionable notes to help an AI coding assistant be productive in this repository.
---

# Copilot Instructions for Contributors and AI Agents

These are short, actionable notes to help an AI coding assistant be productive in this repository.

**🚨 MANDATORY FIRST STEP: Read Folder-Specific Guidance BEFORE Writing Code 🚨**

Before working in any folder, **MUST READ** the corresponding AGENTS.md file:

| Working in... | Read first |
|---------------|------------|
| `docs/` | `docs/AGENTS.md` — Documentation structure, placement, and discovery |
| `test/` | `test/AGENTS.md` — Test writing patterns and helpers |
| `test/e2e/` | `test/e2e/AGENTS.md` — E2E test patterns |
| `src/adapters/` | `src/adapters/AGENTS.md` — Adapter interface and implementation |
| `src/services/` | `src/services/AGENTS.md` — Service layer patterns |

**Before writing or modifying ANY file:**
1. Identify which folder the file is in (e.g., `test/services/foo.test.ts` → `test/`)
2. Read that folder's `AGENTS.md` first
3. Apply its guidance

Skipping these guides leads to broken VS Code mocks, duplicated utilities, and deviation from established patterns.

---

## Development Methodology

### Bug Fixes: Test First

1. **Reproduce first**: Create a failing test that demonstrates the bug
2. **Confirm failure**: Run the test, verify it fails as expected
3. **Fix the code**: Make the minimal change to fix the issue
4. **Confirm fix**: Run the test, verify it passes
5. **No regression**: Run related tests to ensure nothing broke

### Debugging: Isolate the Fault Location

When tests fail, determine whether the bug is in **test code** or **production code** BEFORE iterating:

1. **Read error messages carefully**: `expected X, got Y` tells you what the code produced vs what was expected
2. **Add debug logging to production code first**: If the test setup looks correct, the bug is likely in production code
3. **Trace data transformations**: When IDs or values change unexpectedly, log at each transformation point
4. **Check for inconsistent code paths**: Different entry points (e.g., `installBundle` vs `updateBundle`) may use different logic
5. **Validate assumptions with real-world testing**: If possible, reproduce the issue in the actual extension before fixing

**Red flags that the bug is in production code:**
- Test fixtures match documented formats but validation fails
- Multiple test approaches fail with the same error pattern
- Error shows data transformation (e.g., `v1.0.0` → `1.0.0`) not present in test code

**Anti-pattern**: Repeatedly modifying test fixtures when the error message shows production code is transforming data incorrectly.

### Test-Driven Development (TDD)

Use TDD when it makes sense (most new functionality):
1. Write a failing test for the expected behavior
2. Write the minimum code to make it pass
3. Refactor if needed, keeping tests green

### E2E Testing

E2E tests must invoke actual code paths, never reimplement production code. See `test/e2e/AGENTS.md` for detailed patterns and examples.

### Test Completion Criteria

See `test/AGENTS.md` for the full test completion checklist. Key rule: if tests won't run due to setup issues **you** introduced, the task is incomplete.

### Minimal Code Principle

- Write the **absolute minimum** code to solve the requirement
- No extras, no abstractions, no "nice-to-haves"
- Every line must directly contribute to the solution—if it doesn't, delete it
- Prefer simple, direct implementations over clever ones

### Backward Compatibility

- **Do NOT** try to be backward compatible with changes just introduced in the same session or in the current changed files
- **For new features**: Ask the user if backward compatibility is required before proposing a design
- If backward compatibility is needed, document the migration path

### Discovery Before Design

Before implementing anything new:
1. Search for existing similar functionality (`grep -r "class.*Manager" src/`)
2. Check if utilities already exist in `src/utils/` or `test/helpers/`
3. Review tests for established patterns
4. Reuse before rewriting, consolidate before duplicating

---

## Big Picture

This is a VS Code extension (Prompt Registry) that provides a marketplace and registry for Copilot prompt bundles.

### Architecture Overview

```
src/
├── adapters/     → Source-specific implementations (GitHub, Local, etc.)
├── commands/     → VS Code command handlers
├── services/     → Core business logic (RegistryManager, BundleInstaller, etc.)
├── storage/      → Persistent state management
├── types/        → TypeScript type definitions
├── ui/           → UI providers (Marketplace WebView, Tree View)
├── utils/        → Shared utilities
└── extension.ts  → Entry point
```

### Key Components

- **UI surface**: `src/ui/*` (Marketplace and `RegistryTreeProvider`)
- **Orchestration**: `src/services/RegistryManager.ts` (singleton) coordinates adapters, storage, and installer
- **Installation flow**: adapters produce bundle metadata/URLs → `BundleInstaller` downloads/extracts/validates → scope services sync to target directories
- **Scope services**: `UserScopeService` (user/workspace) and `RepositoryScopeService` (repository) handle scope-specific file placement
- **Lockfile management**: `LockfileManager` manages `prompt-registry.lock.json` for repository-scoped bundles


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AmadeusITGroup/prompt-registry](https://github.com/AmadeusITGroup/prompt-registry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
