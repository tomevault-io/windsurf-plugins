---
trigger: always_on
description: Manages modal open/close state with consistent API.
---

# CLAUDE.md - S4 Frontend Context

> **Note for AI Assistants**: This file contains AI-specific development context for the S4 React frontend. For user-facing architecture documentation, see [docs/architecture/frontend.md](../docs/architecture/frontend.md) and [docs/development/frontend.md](../docs/development/frontend.md). For project overview, see root [CLAUDE.md](../CLAUDE.md). For backend context, see [backend/CLAUDE.md](../backend/CLAUDE.md).

## Frontend Overview

**s4-frontend** - React 18 application with TypeScript, Webpack, and PatternFly 6 component library.

**Technology Stack**: React 18, PatternFly 6, React Router v7, TypeScript, Webpack
**Development**: Port 9000 with Webpack HMR (Hot Module Replacement)
**Production**: Built and served statically by Fastify backend on port 5000 (container)

**For detailed architecture**, see [docs/architecture/frontend.md](../docs/architecture/frontend.md).

## 🎨 PatternFly 6 Critical Requirements

⚠️ **MANDATORY**: Follow the [PatternFly 6 Development Guide](../docs/development/pf6-guide/README.md) as the **AUTHORITATIVE SOURCE** for all UI development.

### Context7 Warning

**DO NOT use Context7 for PatternFly components.** Context7 may contain outdated PatternFly versions (v5 or earlier) that conflict with this project's PatternFly 6 requirements.

**Instead use:**

- **Local guide**: [`docs/development/pf6-guide/`](../docs/development/pf6-guide/README.md) (authoritative, project-specific)
- **Official docs**: [PatternFly.org](https://www.patternfly.org/) (always up-to-date)

Context7 is fine for non-PatternFly libraries: React, Axios, React Router, Jest, i18next, and other dependencies.

### Essential Rules

1. **Class Prefix**: ALL PatternFly classes MUST use `pf-v6-` prefix
2. **Design Tokens**: Use semantic tokens only, never hardcode colors
3. **Component Import**: Import from `@patternfly/react-core` v6 and other @patternfly libraries
4. **Theme Testing**: Test in both light and dark themes
5. **Table Patterns**: Follow guide's table implementation (current code may be outdated)

### Common Mistakes and Token Usage

**Critical rules** - See [`docs/development/pf6-guide/guidelines/styling-standards.md`](../docs/development/pf6-guide/guidelines/styling-standards.md) for complete guide:

- ✅ **ALWAYS** use `pf-v6-` prefix for component classes
- ✅ **ALWAYS** use `--pf-t--` prefix for design tokens (semantic tokens with `-t-`)
- ✅ Choose tokens by **meaning** (e.g., `--pf-t--global--color--brand--default`), not appearance
- ❌ **NEVER** hardcode colors or measurements
- ❌ **NEVER** use legacy `--pf-v6-global--` tokens or numbered base tokens

### Component Import Pattern

```tsx
import { Button, Card, Page, PageSection } from '@patternfly/react-core';
import { Table, Thead, Tbody, Tr, Th, Td } from '@patternfly/react-table';
import { TrashIcon, UploadIcon } from '@patternfly/react-icons';
```

**Version**: PatternFly 6.2.x (NOT PatternFly 5)

## 🗃️ State Management Philosophy

- **Local State First**: Use `useState` for component-specific state
- **Context for Global**: AuthContext for authentication state
- **EventEmitter for Cross-Component**: Use emitter for decoupled communication (upload progress, notifications)
- **Reusable Hooks**: Use custom hooks (`useModal`, `useStorageLocations`) for common patterns
- **No React Query**: Direct API calls with axios and local loading/error states

## 🎯 Component Development Checklist

### Before Creating ANY Component

1. **Search for similar components first** - Use `find_symbol` and `search_for_pattern`
2. **Follow PatternFly 6 requirements** - ALWAYS use `pf-v6-` prefix, semantic tokens, v6 imports
3. **Use established patterns** - Check existing components (StorageBrowser, Buckets, Settings)

### Critical Rules for ALL Components

1. **Error Handling**: MUST use `Emitter.emit('notification', { variant, title, description })` for user-facing errors

   - Use `.catch()` with axios calls
   - Log errors with `console.error()` for debugging
   - Display user-friendly notifications via EventEmitter

2. **Data Fetching**: Use direct axios calls with local state

   - Set loading state before call
   - Handle errors in `.catch()`
   - Update component state on success

3. **Internationalization**: MUST use `t()` function - never hardcode user-facing text

   - Import from `react-i18next`
   - Wrap all strings in `t('key')`

4. **Accessibility**: MUST include ARIA labels and keyboard navigation

   - Add `aria-label` to interactive elements
   - Ensure keyboard navigation works
   - Test with screen readers when possible

5. **PatternFly 6**: MUST use `pf-v6-` prefix and semantic design tokens
   - Never hardcode colors or spacing
   - Use `--pf-t--` tokens for styling
   - Test in both light and dark themes

### Component Utilities

6. **Hooks**: MUST use reusable hooks for common patterns

   - Use `useModal()` for modal state management instead of inline useState
   - Use `useStorageLocations()` for loading storage locations

7. **Notifications**: MUST use notification utilities for user feedback

   - Import from `@app/utils/notifications`
   - Use `notifySuccess()`, `notifyError()`, `notifyWarning()`, `notifyInfo()`
   - Use `notifyApiError()` for consistent API error handling


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rh-aiservices-bu/s4](https://github.com/rh-aiservices-bu/s4) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
