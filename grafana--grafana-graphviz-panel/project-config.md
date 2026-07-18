---
trigger: always_on
description: This document provides guidance for AI agents working on the Grafana Graphviz Panel codebase.
---


# AI Agent Guidelines

This document provides guidance for AI agents working on the Grafana Graphviz Panel codebase.

## Quick Reference

### Essential Documentation

- **[ARCHITECTURE.md](./ARCHITECTURE.md)** - Code organization, design principles, and where to add new features
- **[README.md](./README.md)** - Development setup, testing, and build commands

### Key Commands

```bash
npm run dev          # Development mode with watch
npm run build        # Production build
npm run test         # Run tests with watch
npm run test:ci      # Run all tests with coverage
npm run lint         # Check code style
npm run lint:fix     # Auto-fix code style issues
npm run server       # Spin up local Grafana instance
```

## Code Organization Principles

### Directory Structure Decision Tree

**Q: What am I building?**

1. **Business Logic (Graphviz diagramming operations)**

   - Pure functions, no Grafana dependencies
   - Location: `src/core/`
   - Examples: DOT parsing, graph manipulation, validation

2. **Grafana Integration**

   - Uses `@grafana/*` packages
   - Location: `src/integrations/`
   - Examples: DataFrame extraction, theme styling, AI Assistant

3. **UI Components**

   - **Builder modals?** → `src/components/modals/`
   - **Panel configuration editors?** → `src/components/panel-options/`
   - **Empty/error states?** → `src/components/states/`
   - **AI Assistant features?** → `src/components/assistant/`
   - **Main panel logic?** → `src/components/Panel.tsx` or `BuilderModeOverlay.tsx`

4. **React Hooks**

   - Location: `src/hooks/`
   - Use barrel export: `src/hooks/index.ts`

5. **Type Definitions**
   - Shared types: `src/types.ts`

### The Litmus Test

**"If Grafana releases a major version update, would this file need changes?"**

- **YES** → `src/integrations/` (platform-coupled)
- **NO** → `src/core/` (framework-agnostic)

## Common Patterns & Best Practices

### Import Rules

```typescript
// ✅ Good - Use barrel exports
import { NodeFormModal, EdgeFormModal } from './modals';
import { useGraphvizRenderPipeline } from '../hooks';

// ❌ Bad - Individual imports when barrel exists
import { NodeFormModal } from './modals/NodeFormModal';

// ✅ Good - Core stays pure
import { fromDot, toDot } from 'ts-graphviz';

// ❌ Bad - Never import Grafana in core
import { PanelData } from '@grafana/data';
```

### File Placement Examples

**Example 1: Adding a new Graphviz layout algorithm**

```
Location: src/core/
Why: Pure business logic, no Grafana dependencies
```

**Example 2: Adding a new panel configuration option**

```
Component: src/components/panel-options/MyNewEditor.tsx
Registration: src/module.ts (add to setPanelOptions builder)
Export: Add to src/components/panel-options/index.ts
```

**Example 3: Adding a new builder mode tool**

```
Component: src/components/BuilderModeOverlay.tsx (or new modal)
Logic: src/core/builderMode.ts
Types: src/types.ts (add to BuilderTool enum)
```

## Testing Guidelines

### Running Tests

```bash
npm run test:ci      # Jest unit tests with coverage
npm run e2e          # Playwright E2E tests in Docker (HTML report)
npm run e2e:llm      # E2E tests with verbose list output (LLM-friendly)
npm run e2e:ui       # Playwright UI mode (local only, not in Docker)
npm run coverage     # Combined Jest + Playwright coverage report
```

### When to Run Which Tests

**After modifying source code (`src/`):**

- ✅ Run `npm run test:ci` - Validates your changes don't break unit tests
- ✅ Run `npm run build` - Ensures code compiles without errors
- ⚠️ Run `npm run e2e` - Only if you modified code that E2E tests depend on

**After adding/modifying E2E tests (`e2e/specs/`):**

- ✅ Run `npx tsc --noEmit e2e/specs/your-test.spec.ts` - Quick TypeScript check
- ✅ Run `npm run build` - Build the plugin
- ✅ Run single test during development:
  ```bash
  npm run e2e:llm e2e/specs/your-test.spec.ts
  ```
- ✅ Run `npm run e2e:llm` before committing - All tests with verbose output
- ⚠️ Do NOT run only `npm run test:ci` - This won't validate E2E tests!

**After modifying dashboard JSON (`provisioning/dashboards/`):**

- ✅ Run `npm run e2e` - Dashboard changes require full E2E validation
- ❌ Do NOT assume unit tests validate dashboard changes

**Quick validation workflow for E2E work:**

```bash
# 1. TypeScript check (fast, catches syntax errors)
npx tsc --noEmit e2e/specs/your-test.spec.ts

# 2. Build source (E2E tests run against built plugin)
npm run build

# 3. Run only your test with verbose output (fastest iteration)
npm run e2e:llm e2e/specs/your-test.spec.ts

# 4. Before committing: Run all E2E tests
npm run e2e:llm
```

### Test Strategy: Pure Unit Tests vs E2E

This project has **merged coverage reporting** combining Jest unit tests and Playwright E2E tests. Because of this:

**Prefer PURE unit tests:**

- ✅ Test `core/` business logic (no Grafana dependencies)
- ✅ Test utility functions with simple inputs/outputs
- ✅ Test React hooks in isolation
- ✅ Minimal mocking required

**Use E2E tests when:**

- ⚠️ Heavy mocking is required (many `jest.mock()` statements)
- ⚠️ Brittle mocks that break with minor changes
- ⚠️ Testing complex UI interactions
- ⚠️ Testing Grafana integration points
- ⚠️ Testing full user workflows


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [grafana/grafana-graphviz-panel](https://github.com/grafana/grafana-graphviz-panel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-18 -->
