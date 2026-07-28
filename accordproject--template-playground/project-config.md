---
trigger: always_on
description: This repository is the **Accord Project Template Playground** - a web-based IDE for creating, testing, and sharing Accord Project templates that combine TemplateMark (natural language), Concerto (data models), and TypeScript (business logic).
---

# GitHub Copilot Instructions for Template Playground

This repository is the **Accord Project Template Playground** - a web-based IDE for creating, testing, and sharing Accord Project templates that combine TemplateMark (natural language), Concerto (data models), and TypeScript (business logic).

## Project Architecture

### Tech Stack
- **Framework**: React 18 with TypeScript (strict mode enabled)
- **Build Tool**: Vite 4.5+ with 8GB memory allocation (`NODE_OPTIONS=--max-old-space-size=8192`)
- **State Management**: Zustand with immer middleware (single store pattern)
- **UI Libraries**: 
  - Ant Design 5.7+ (primary UI framework)
  - TailwindCSS 3.4+ with scoped preflight (`.twp` class, isolated from Ant Design)
  - Styled Components 6.1+ for dynamic theming
- **Editors**: Monaco Editor with custom Concerto language definition
- **Testing**: 
  - Unit: Vitest with jsdom and Testing Library
  - E2E: Playwright with 30s timeouts and localStorage presets
- **Routing**: React Router v6 with hash-based routing for shareable links
- **AI Integrations**: OpenAI, Anthropic, Google Genai, Mistral AI SDKs
- **Core Dependencies**: @accordproject/concerto-core, @accordproject/template-engine, @accordproject/markdown-template, @accordproject/markdown-transform

### File Structure
```
src/
  ├── components/        # Reusable React components
  ├── pages/            # Page-level route components
  ├── store/            # Zustand state management
  ├── types/            # TypeScript type definitions
  ├── utils/            # Pure utility functions
  ├── ai-assistant/     # AI provider integrations
  ├── editors/          # Monaco editor components
  ├── samples/          # Template examples (MODEL, DATA, TEMPLATE exports)
  └── tests/            # Unit tests (mirrors src structure)
e2e/                    # Playwright end-to-end tests
public/                 # Static assets and content
```

---

## Critical Requirements

### 1. Commit Hygiene (BLOCKING)

**Every commit MUST include Developer Certificate of Origin (DCO) sign-off:**
```bash
git commit --signoff -m "feat(scope): description"
```

**Commit Message Format (REQUIRED):**
Follow [Accord Project commit conventions](https://github.com/accordproject/techdocs/blob/master/DEVELOPERS.md#commit-message-format):
```
type(scope): description

[optional body]

Signed-off-by: Your Name <your.email@example.com>
```

**Types**: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`, `ci`  
**Scopes**: Component or area name (e.g., `ConcertoEditor`, `UI`, `CI`, `sharable`, `AI`)

**Examples from merged PRs:**
- ✅ `feat(ConcertoEditor): Added Syntax Highlighting`
- ✅ `fix(sharable): agreementData preview screen bug fixed`
- ✅ `chore(CI): fix build and deployment issues`
- ❌ `update readme` (missing type, scope, and DCO)
- ❌ `Add new feature` (missing scope and DCO)

**Reminder**: Missing DCO sign-off will block PR merging. Configure `git config alias.c 'commit --signoff'` for convenience.

---

### 2. Testing Requirements (VITAL)

**All user-facing features MUST have test coverage before merging.**

#### Unit Tests (Vitest)
- **Location**: `src/tests/` (mirrors source structure)
- **Setup**: Import `@testing-library/jest-dom` for matchers
- **Conventions**:
  - Wrap components using `useNavigate` in `<MemoryRouter>`
  - Mock `window.matchMedia` for responsive components
  - Mock `HTMLCanvasElement.getContext` for lottie-web animations
  - Use Zustand `getState()` for store assertions

**Example Pattern:**
```typescript
import { describe, it, expect, beforeEach, vi } from 'vitest';
import { render, screen } from '@testing-library/react';
import '@testing-library/jest-dom';
import { MemoryRouter } from 'react-router-dom';

describe('ComponentName', () => {
  beforeEach(() => {
    // Reset store state
    useAppStore.setState(initialState);
  });

  it('should render correctly', () => {
    render(
      <MemoryRouter>
        <ComponentName />
      </MemoryRouter>
    );
    expect(screen.getByRole('button', { name: 'Submit' })).toBeInTheDocument();
  });
});
```

#### E2E Tests (Playwright)
- **Location**: `e2e/` directory
- **Conventions**:
  - Set `localStorage.setItem('hasVisited', 'true')` to skip tour
  - Wait for `.app-spinner-container` to be hidden (30s timeout)
  - Use semantic selectors: `getByRole`, `getByLabel`, `getByText`
  - Sort test files by feature area

**Example Pattern:**
```typescript
test.describe('Feature Name', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/');
    await expect(page.locator('.app-spinner-container')).toBeHidden({ timeout: 30000 });
  });

  test('should complete user workflow', async ({ page }) => {
    const button = page.getByRole('button', { name: 'Generate' });
    await expect(button).toBeVisible();
    await button.click();
    await expect(page.getByText('Success')).toBeVisible();
  });
});
```

**PR Review Pattern**: PRs lacking tests for vital features will be asked to add coverage (see PR #60, #90).

---

## Code Style and TypeScript Conventions

### TypeScript Strictness

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [accordproject/template-playground](https://github.com/accordproject/template-playground) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
