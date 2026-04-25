---
trigger: always_on
description: This project uses **[Changesets](https://github.com/changesets/changesets)** for version management and publishing in a monorepo with two packages:
---


# Release and Changeset Process

## Overview

This project uses **[Changesets](https://github.com/changesets/changesets)** for version management and publishing in a monorepo with two packages:

- `@signalis/core`
- `@signalis/react`

## When to Create a Changeset

Create a changeset for any changes that will be released to users:

- ✅ New features
- ✅ Bug fixes
- ✅ Breaking changes
- ✅ Documentation updates that affect API usage
- ❌ Internal refactoring with no user-facing changes
- ❌ Test-only changes
- ❌ Build/tooling changes

## Creating a Changeset

### Option 1: Interactive (Recommended for Humans)

```bash
pnpm changeset
```

This will prompt you to:

1. Select which packages changed
2. Choose change type (major, minor, or patch)
3. Write a description

### Option 2: Non-Interactive (Recommended for AI)

Create a markdown file directly in `.changeset/` with YAML frontmatter:

```bash
# Create .changeset/descriptive-name.md
---
'@signalis/react': minor
'@signalis/core': patch
---

Brief summary of changes

- Detailed change 1
- Detailed change 2
- Detailed change 3
```

**File naming**: Use kebab-case, descriptive names (e.g., `mixed-deps-usesignaleffect.md`)

## Semantic Versioning Guidelines

### Major (Breaking Changes)

- Removing or renaming public APIs
- Changing function signatures in non-backward-compatible ways
- Removing support for a feature

Example: `'@signalis/core': major`

### Minor (New Features)

- Adding new functions, hooks, or features
- Adding optional parameters to existing APIs
- Deprecating (but not removing) APIs

Example: `'@signalis/react': minor`

### Patch (Bug Fixes)

- Fixing bugs without changing public APIs
- Performance improvements
- Documentation fixes

Example: `'@signalis/core': patch`

## Commit Message Format

Use conventional commit format:

```bash
git commit -m "type(scope): subject

- Detailed bullet point 1
- Detailed bullet point 2"
```

**Types:**

- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation only
- `refactor`: Code change without feature/fix
- `test`: Adding or updating tests
- `chore`: Tooling, dependencies, config

**Scopes:**

- `core`: Changes to @signalis/core
- `react`: Changes to @signalis/react
- `root`: Changes to root config/tooling

**Examples:**

```bash
# New feature
git commit -m "feat(react): add deps parameter to useSignalEffect

- Add optional deps parameter for mixed dependencies
- Fix stale closure issue with props/state
- Add comprehensive tests
- Update README with examples"

# Bug fix
git commit -m "fix(core): prevent memory leak in effect cleanup

- Properly dispose observers when effect is removed
- Add test for cleanup behavior"

# Breaking change
git commit -m "feat(core)!: change Signal constructor API

BREAKING CHANGE: Signal constructor now requires explicit type parameter

- Remove type inference from constructor
- Update all internal usages
- Update documentation"
```

## Pull Request Workflow

### 1. Create Feature Branch

```bash
git checkout -b username/feature-description
```

### 2. Make Changes & Create Changeset

```bash
# Make your changes
# ...

# Create changeset (if user-facing changes)
# Create .changeset/descriptive-name.md manually

# Stage and commit
git add .
git commit -m "feat(scope): description"
```

### 3. Push & Create PR

```bash
git push origin username/feature-description

# If using gh CLI
gh pr create --title "feat(scope): description" --body "Description of changes"
```

### 4. PR Requirements

Before merging:

- ✅ All tests pass (`pnpm test`)
- ✅ Linting passes (`pnpm lint`)
- ✅ Build succeeds (`pnpm build`)
- ✅ Changeset created (for user-facing changes)
- ✅ Documentation updated (if needed)

## Publishing a Release

### Automated Release Process (Recommended)

This project uses **[Release It](https://github.com/release-it/release-it)** orchestration with **[Changesets](https://github.com/changesets/changesets)** for version management:

```bash
pnpm release
```

This single command will:

1. **Apply changesets** - Run `pnpm changeset version` to bump workspace package versions
2. **Commit version changes** - Automatically commit the version bump with "chore: version packages"
3. **Install dependencies** - Run `pnpm install` to update lockfile
4. **Build packages** - Run `pnpm build` to compile TypeScript
5. **Create git tag** - Tag based on workspace package version (e.g., `v0.2.7`)
6. **Push to remote** - Push commits and tags to GitHub
7. **Publish to npm** - Publish all workspace packages with `--access public`

**Important**: The root `package.json` version is **not used** for releases. Tags and versions are derived from the workspace packages (`packages/*/package.json`).

### Testing Releases

⚠️ **Warning**: Do NOT use `pnpm release:dry` as it will consume changesets without publishing.

Instead, manually verify:

```bash
# Check what changesets exist
ls .changeset/*.md

# Preview what versions would be bumped
pnpm changeset status

# Verify build works
pnpm build

# Check tests pass
pnpm test
```

### Legacy Manual Process

If needed, the old manual process is still available:

```bash
# Step 1: Version bump
pnpm version

# Step 2: Build packages  
pnpm build


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cafreeman) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
