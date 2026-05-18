---
trigger: always_on
description: This document defines the Git workflow for the guarahooks project, including commit conventions, branches, and contribution process.
---



# Git Workflow

This document defines the Git workflow for the guarahooks project, including commit conventions, branches, and contribution process.

## 🌳 Branch Strategy

### Simplified Flow

```tree
main (production)
├── feature/add-use-toggle
├── feature/improve-docs
├── fix/use-fetch-bug
├── hotfix/critical-fix
└── docs/update-readme
```

### Branch Types

#### `main`

- Main production branch
- Always stable and deployable
- All merges via Pull Request
- Release tags created here
- **Base for all other branches**

#### `feature/*`

- New functionalities
- New hooks
- Existing improvements
- **Base: `main`**

#### `fix/*`

- Bug fixes
- **Base: `main`**
- Merge to `main`

#### `hotfix/*`

- Critical urgent fixes
- **Base: `main`**
- Direct merge to `main`

#### `docs/*`

- Documentation updates
- **Base: `main`**
- Merge to `main`

## 📝 Conventional Commits

### Basic Structure

```bash
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

### Commit Types

#### `feat` - New Feature

```bash
feat: add useToggle hook
feat(hooks): add useLocalStorage with cross-tab sync
feat(cli): add hook installation command
```

#### `fix` - Bug Fix

```bash
fix: resolve memory leak in useInterval
fix(hooks): handle SSR properly in useWindowSize
fix(docs): correct useToggle example
```

#### `docs` - Documentation

```bash
docs: add API reference for useToggle
docs(hooks): improve useLocalStorage examples
docs: update contributing guidelines
```

#### `style` - Formatting

```bash
style: format code with prettier
style(hooks): fix eslint warnings
```

#### `refactor` - Refactoring

```bash
refactor: simplify useToggle implementation
refactor(types): improve TypeScript definitions
```

#### `perf` - Performance

```bash
perf: optimize useDebounce memory usage
perf(hooks): reduce bundle size of useFetch
```

#### `test` - Tests

```bash
test: add unit tests for useToggle
test(hooks): improve coverage for useLocalStorage
```

#### `build` - Build System

```bash
build: update pnpm to v9.15.3
build(ci): add automated testing workflow
```

#### `ci` - Continuous Integration

```bash
ci: add automated hook registry validation
ci: setup semantic release workflow
```

#### `chore` - Maintenance

```bash
chore: update dependencies
chore: bump version to 1.2.0
```

### Main Scopes

#### `hooks`

```bash
feat(hooks): add useGeolocation hook
fix(hooks): resolve useLocalStorage race condition
```

#### `docs`

```bash
docs(hooks): improve useToggle documentation
feat(docs): add interactive examples
```

#### `cli`

```bash
feat(cli): add hook template generator
fix(cli): resolve installation path issues
```

#### `registry`

```bash
feat(registry): add hook categories
fix(registry): resolve dependency resolution
```

## 🔀 Development Workflow

### 1. Creating Feature Branch

```bash
# Sync with main
git checkout main
git pull origin main

# Create feature branch
git checkout -b feature/add-use-geolocation

# First implementation
git add .
git commit -m "feat(hooks): add basic useGeolocation implementation"

# Push branch
git push -u origin feature/add-use-geolocation
```

### 2. Iterative Development

```bash
# Implement hook
git add registry/hooks/use-geolocation.tsx
git commit -m "feat(hooks): implement useGeolocation with options"

# Add example
git add registry/example/use-geolocation-demo.tsx
git commit -m "feat(hooks): add useGeolocation demo component"

# Add documentation
git add content/docs/hooks/use-geolocation.mdx
git commit -m "docs(hooks): add useGeolocation documentation"

# Register in system
git add registry/registry-hooks.ts registry/registry-examples.ts
git commit -m "feat(registry): register useGeolocation hook and example"

# Update navigation
git add config/docs.ts
git commit -m "feat(docs): add useGeolocation to navigation"
```

### 3. Finalizing Feature

```bash
# Build and tests
pnpm build:registry
pnpm build:docs
pnpm lint

# Final adjustments commit
git add .
git commit -m "fix(hooks): resolve linting issues in useGeolocation"

# Final push
git push origin feature/add-use-geolocation
```

### 4. Pull Request to Main

```markdown
## 🎯 Type of Change
- [x] New feature (feature)
- [ ] Bug fix (fix)
- [ ] Documentation (docs)
- [ ] Refactoring (refactor)

## 📋 Description
Adds `useGeolocation` hook to access user location with:
- Precision options support
- Robust error handling
- Loading states
- Complete documentation

## 🧪 Tests
- [x] Hook tested manually
- [x] Functional example
- [x] Build without errors
- [x] Complete documentation

## 📚 Documentation
- [x] Hook documentation created
- [x] Example added
- [x] API documented
- [x] Navigation updated
```

## 🏷️ Versioning and Releases

### Semantic Versioning

```tree
MAJOR.MINOR.PATCH

1.2.3
│ │ │
│ │ └── Patch: Bug fixes
│ └──── Minor: New features
└────── Major: Breaking changes
```

### Release Types

#### Patch (1.0.1)

```bash
fix: resolve useToggle callback issue
fix(hooks): handle edge case in useLocalStorage
docs: fix typo in useGeolocation example
```

#### Minor (1.1.0)

```bash
feat: add useGeolocation hook
feat: add useNotifications hook
feat(cli): add hook scaffolding command
```

#### Major (2.0.0)

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [h3rmel/guarahooks](https://github.com/h3rmel/guarahooks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
