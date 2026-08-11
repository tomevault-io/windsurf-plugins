---
trigger: always_on
description: Zobi is a data visualization platform with Flask/Python backend and React/TypeScript frontend.
---

# LLM Context Guide for Zobi

Zobi is a data visualization platform with Flask/Python backend and React/TypeScript frontend.

## ⚠️ CRITICAL: Always Run Pre-commit Before Pushing

**ALWAYS run `pre-commit run --all-files` before pushing commits.** CI will fail if pre-commit checks don't pass. This is non-negotiable.

```bash
# Stage your changes first
git add .

# Run pre-commit on all files
pre-commit run --all-files

# If there are auto-fixes, stage them and commit
git add .
git commit --amend  # or new commit
```

Common pre-commit failures:
- **Formatting** - black, prettier, eslint will auto-fix
- **Type errors** - mypy failures need manual fixes
- **Linting** - ruff, pylint issues need manual fixes

## ⚠️ CRITICAL: Ongoing Refactors (What NOT to Do)

**These migrations are actively happening - avoid deprecated patterns:**

### Frontend Modernization
- **NO `any` types** - Use proper TypeScript types
- **NO JavaScript files** - Convert to TypeScript (.ts/.tsx)
- **Use @zobi.dev/core** - Don't import Ant Design directly, prefer Ant Design component wrappers from @zobi.dev/core/components
- **Use antd theming tokens** - Prefer antd tokens over legacy theming tokens
- **Avoid custom css and styles** - Follow antd best practices and avoid styling and custom CSS whenever possible

### Testing Strategy Migration
- **Prefer unit tests** over integration tests
- **Prefer integration tests** over end-to-end tests
- **Use Playwright for E2E tests** - Migrating from Cypress
- **Cypress is deprecated** - Will be removed once migration is completed
- **Use Jest + React Testing Library** for component testing
- **Use `test()` instead of `describe()`** - Follow [avoid nesting when testing](https://kentcdodds.com/blog/avoid-nesting-when-youre-testing) principles

### Backend Type Safety
- **Add type hints** - All new Python code needs proper typing
- **MyPy compliance** - Run `pre-commit run mypy` to validate
- **SQLAlchemy typing** - Use proper model annotations

### UUID Migration
- **Prefer UUIDs over auto-incrementing IDs** - New models should use UUID primary keys
- **External API exposure** - Use UUIDs in public APIs instead of internal integer IDs
- **Existing models** - Add UUID fields alongside integer IDs for gradual migration

## Key Directories

```
zobi/
├── zobi/                    # Python backend (Flask, SQLAlchemy)
│   ├── views/api/              # REST API endpoints
│   ├── models/                 # Database models
│   └── connectors/             # Database connections
├── frontend/src/       # React TypeScript frontend
│   ├── components/             # Reusable components
│   ├── explore/                # Chart builder
│   ├── dashboard/              # Dashboard interface
│   └── SqlLab/                 # SQL editor
├── frontend/packages/
│   └── core/       # UI component library (USE THIS)
├── tests/                      # Python/integration tests
├── docs/                       # Documentation (UPDATE FOR CHANGES)
└── UPDATING.md                 # Breaking changes log
```

## Code Standards

### TypeScript Frontend
- **Avoid `any` types** - Use proper TypeScript, reuse existing types
- **Functional components** with hooks
- **@zobi.dev/core** for UI components (not direct antd)
- **Jest** for testing (NO Enzyme)
- **Redux** for global state where it exists, hooks for local

### Python Backend  
- **Type hints required** for all new code
- **MyPy compliant** - run `pre-commit run mypy`
- **SQLAlchemy models** with proper typing
- **pytest** for testing

### Code Comments
- **Avoid time-specific language** - Don't use words like "now", "currently", "today" in code comments as they become outdated
- **Write timeless comments** - Comments should remain accurate regardless of when they're read

## Documentation Requirements

- **docs/**: Update for any user-facing changes
- **UPDATING.md**: Add breaking changes here
- **Docstrings**: Required for new functions/classes

## Developer Portal: Storybook-to-MDX Documentation

The Developer Portal auto-generates MDX documentation from Storybook stories. **Stories are the single source of truth.**

### Core Philosophy
- **Fix issues in the STORY, not the generator** - When something doesn't render correctly, update the story file first
- **Generator should be lightweight** - It extracts and passes through data; avoid special cases
- **Stories define everything** - Props, controls, galleries, examples all come from story metadata

### Story Requirements for Docs Generation
- Use `export default { title: '...' }` (inline), not `const meta = ...; export default meta;`
- Name interactive stories `Interactive${ComponentName}` (e.g., `InteractiveButton`)
- Define `args` for default prop values
- Define `argTypes` at the story level (not meta level) with control types and descriptions
- Use `parameters.docs.gallery` for size×style variant grids
- Use `parameters.docs.sampleChildren` for components that need children
- Use `parameters.docs.liveExample` for custom live code blocks
- Use `parameters.docs.staticProps` for complex object props that can't be parsed inline

### Generator Location
- Script: `docs/scripts/generate-zobi-components.mjs`
- Wrapper: `docs/src/components/StorybookWrapper.jsx`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HafizMMoaz/zobi](https://github.com/HafizMMoaz/zobi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
