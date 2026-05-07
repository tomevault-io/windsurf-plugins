---
trigger: always_on
description: Git-flow branching strategy and workflow conventions for the Prax project
---


# Git-Flow Workflow

This project follows the **Git-Flow** branching model for organized development and releases.

## Branch Structure

```
main (production)
  │
  └── develop (integration)
        │
        ├── feature/* (new features)
        ├── bugfix/* (non-critical fixes)
        ├── release/* (release preparation)
        └── hotfix/* (critical production fixes)
```

## Primary Branches

### `main`
- **Purpose**: Production-ready code only
- **Protection**: Protected, requires PR approval
- **Deploys to**: Production / crates.io releases
- **Never commit directly** - only merge from `release/*` or `hotfix/*`

### `develop`
- **Purpose**: Integration branch for features
- **Contains**: Latest delivered development changes
- **Base for**: All `feature/*` and `bugfix/*` branches
- **Merged to**: `release/*` branches

## Supporting Branches

### Feature Branches: `feature/<name>`

For new features and enhancements.

```bash
# Create feature branch from develop
git checkout develop
git pull origin develop
git checkout -b feature/query-builder

# Work on feature...
git add .
git commit -m "feat(query): implement basic query builder"

# Keep up to date with develop
git fetch origin develop
git rebase origin/develop

# Push and create PR to develop
git push -u origin feature/query-builder
```

**Naming convention**: `feature/<scope>-<description>`
- `feature/query-builder`
- `feature/postgres-connection-pool`
- `feature/schema-parser`

### Bugfix Branches: `bugfix/<name>`

For non-critical bug fixes during development.

```bash
git checkout develop
git checkout -b bugfix/connection-timeout

# Fix the bug...
git commit -m "fix(postgres): handle connection timeout gracefully"

git push -u origin bugfix/connection-timeout
```

**Naming convention**: `bugfix/<scope>-<description>`
- `bugfix/query-null-handling`
- `bugfix/migration-rollback`

### Release Branches: `release/<version>`

For preparing a new production release.

```bash
# Create release branch from develop
git checkout develop
git checkout -b release/0.1.0

# Update version in Cargo.toml
# Update CHANGELOG.md
# Final testing and bug fixes only

git commit -m "chore(release): prepare v0.1.0"

# Merge to main
git checkout main
git merge --no-ff release/0.1.0
git tag -a v0.1.0 -m "Release v0.1.0"

# Merge back to develop
git checkout develop
git merge --no-ff release/0.1.0

# Delete release branch
git branch -d release/0.1.0
```

**Naming convention**: `release/<semver>`
- `release/0.1.0`
- `release/1.0.0`
- `release/2.3.1`

### Hotfix Branches: `hotfix/<name>`

For critical production fixes that can't wait.

```bash
# Create hotfix from main
git checkout main
git checkout -b hotfix/security-vulnerability

# Fix the issue...
git commit -m "fix(security): patch SQL injection vulnerability"

# Merge to main
git checkout main
git merge --no-ff hotfix/security-vulnerability
git tag -a v0.1.1 -m "Hotfix v0.1.1"

# Merge to develop (or current release branch)
git checkout develop
git merge --no-ff hotfix/security-vulnerability

# Delete hotfix branch
git branch -d hotfix/security-vulnerability
```

**Naming convention**: `hotfix/<description>`
- `hotfix/security-patch`
- `hotfix/critical-query-fix`

## Workflow Diagrams

### Feature Development
```
develop ─────●─────────────●─────────────●───────
              \           /
feature/*      ●────●────●
               ↑    ↑    ↑
            commits on feature
```

### Release Process
```
main    ─────────────────────●────── (v0.1.0)
                            /
release/0.1.0    ●────●────●
                /
develop ───●───●─────────────●───────
```

### Hotfix Process
```
main    ────●─────────────●────── (v0.1.1)
             \           /
hotfix/*      ●────●────●
                        \
develop ─────────────────●───────
```

## Commit Message Format

All commits **MUST** follow [Conventional Commits](https://conventionalcommits.org/) with **REQUIRED scope**:

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

### ⚠️ Scope is REQUIRED

Every commit must include a scope. This is enforced by the `commit-msg` git hook.

**Valid scopes:**
- Crate names: `query`, `postgres`, `mysql`, `sqlite`, `mssql`, `mongodb`, `duckdb`, `schema`, `codegen`, `migrate`, `cli`
- Special: `deps`, `ci`, `docs`, `release`, `security`

### Valid Types

| Type | Description | Example |
|------|-------------|---------|
| `feat` | New feature | `feat(query): add nested filter support` |
| `fix` | Bug fix | `fix(postgres): handle connection timeout` |
| `docs` | Documentation | `docs(readme): add installation guide` |
| `style` | Formatting | `style(query): fix indentation` |
| `refactor` | Code refactoring | `refactor(schema): simplify parser logic` |
| `perf` | Performance | `perf(query): optimize SQL generation` |
| `test` | Tests | `test(postgres): add connection tests` |
| `build` | Build system | `build(deps): update tokio to 1.35` |
| `ci` | CI/CD | `ci(github): add benchmark workflow` |
| `chore` | Maintenance | `chore(release): bump version to 0.3.3` |
| `revert` | Revert commit | `revert(query): undo filter changes` |

### Breaking Changes

Add `!` before `:` for breaking changes:
```
feat(api)!: change query builder interface
```

### Types by Branch


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [quinnjr/prax](https://github.com/quinnjr/prax) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
