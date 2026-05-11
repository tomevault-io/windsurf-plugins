---
trigger: always_on
description: This project follows the **Gitflow** branching model for organized development and release management.
---


# Gitflow Branching Strategy

This project follows the **Gitflow** branching model for organized development and release management.

## Branch Structure

### Main Branches (Long-lived)

#### `main`
- **Purpose:** Production-ready code
- **Protected:** Yes
- **Merged from:** `release/*` and `hotfix/*` only
- **Never commit directly to this branch**

#### `develop`
- **Purpose:** Integration branch for features
- **Protected:** Yes
- **Merged from:** `feature/*`, `release/*`, and `hotfix/*`
- **Base for:** All feature branches

### Supporting Branches (Short-lived)

#### `feature/*`
- **Purpose:** New features or enhancements
- **Naming:** `feature/<issue-number>-<short-description>`
- **Examples:**
  - `feature/123-add-websocket-support`
  - `feature/456-user-authentication`
- **Base:** `develop`
- **Merge to:** `develop`
- **Lifetime:** Duration of feature development

#### `release/*`
- **Purpose:** Prepare for production release
- **Naming:** `release/<version>`
- **Examples:**
  - `release/1.0.0`
  - `release/2.1.0`
- **Base:** `develop`
- **Merge to:** `main` and `develop`
- **Lifetime:** Until release is finalized

#### `hotfix/*`
- **Purpose:** Critical bug fixes in production
- **Naming:** `hotfix/<version>-<description>`
- **Examples:**
  - `hotfix/1.0.1-security-patch`
  - `hotfix/2.1.1-memory-leak`
- **Base:** `main`
- **Merge to:** `main` and `develop`
- **Lifetime:** Until hotfix is deployed

## Workflow

### Starting a New Feature

```bash
# Ensure develop is up to date
git checkout develop
git pull origin develop

# Create feature branch
git checkout -b feature/123-add-caching

# Work on feature...
git add .
git commit -m "feat: add Redis caching support"

# Push to remote
git push origin feature/123-add-caching

# Create Pull Request to develop
```

### Completing a Feature

```bash
# Update from develop
git checkout develop
git pull origin develop

git checkout feature/123-add-caching
git merge develop

# Resolve any conflicts
# Run tests
cargo test --all-features

# Push and create PR
git push origin feature/123-add-caching
```

### Creating a Release

```bash
# Create release branch from develop
git checkout develop
git pull origin develop
git checkout -b release/1.0.0

# Update version numbers
# Update CHANGELOG.md
# Final testing

# Commit release preparation
git commit -am "chore: prepare release 1.0.0"

# Merge to main
git checkout main
git merge --no-ff release/1.0.0
git tag -a v1.0.0 -m "Release version 1.0.0"

# Merge back to develop
git checkout develop
git merge --no-ff release/1.0.0

# Push everything
git push origin main develop --tags

# Delete release branch
git branch -d release/1.0.0
git push origin --delete release/1.0.0
```

### Creating a Hotfix

```bash
# Create hotfix branch from main
git checkout main
git pull origin main
git checkout -b hotfix/1.0.1-critical-fix

# Fix the issue
git commit -am "fix: resolve critical security vulnerability"

# Merge to main
git checkout main
git merge --no-ff hotfix/1.0.1-critical-fix
git tag -a v1.0.1 -m "Hotfix version 1.0.1"

# Merge to develop
git checkout develop
git merge --no-ff hotfix/1.0.1-critical-fix

# Push everything
git push origin main develop --tags

# Delete hotfix branch
git branch -d hotfix/1.0.1-critical-fix
git push origin --delete hotfix/1.0.1-critical-fix
```

## Commit Message Convention

Follow **Conventional Commits** specification:

### Format

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Types

- **feat**: New feature
- **fix**: Bug fix
- **docs**: Documentation only changes
- **style**: Code style changes (formatting, missing semicolons, etc.)
- **refactor**: Code refactoring
- **perf**: Performance improvements
- **test**: Adding or updating tests
- **chore**: Maintenance tasks, dependency updates
- **ci**: CI/CD changes
- **build**: Build system changes

### Examples

```bash
# Feature
git commit -m "feat(queue): add job retry with exponential backoff"

# Bug fix
git commit -m "fix(auth): resolve JWT token expiration issue"

# Documentation
git commit -m "docs(readme): update installation instructions"

# Breaking change
git commit -m "feat(api)!: change response format

BREAKING CHANGE: API responses now use camelCase instead of snake_case"

# Multiple changes
git commit -m "chore: update dependencies and fix linting issues

- Update tokio to 1.35
- Update serde to 1.0.195
- Fix clippy warnings in cache module"
```

## Pull Request Guidelines

### Creating PRs

1. **Base branch:**
   - Features → `develop`
   - Hotfixes → `main` (then merge to `develop`)
   - Releases → `main` (then merge to `develop`)

2. **Title format:**
   - Follow commit message convention
   - Example: `feat: add WebSocket support for real-time updates`

3. **Description must include:**
   - Summary of changes
   - Related issue numbers
   - Testing performed
   - Breaking changes (if any)

### PR Template

```markdown
## Description
Brief description of what this PR does

## Related Issues
Closes #123
Relates to #456

## Type of Change
- [ ] Bug fix (non-breaking change which fixes an issue)
- [ ] New feature (non-breaking change which adds functionality)
- [ ] Breaking change (fix or feature that would cause existing functionality to not work as expected)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [quinnjr/armature](https://github.com/quinnjr/armature) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->
