---
trigger: always_on
description: Git workflow, commit message format, branching strategy, and semantic release process
---


# Git Workflow and Release Process

## Commit Message Format

We use **Conventional Commits** for automated semantic versioning and changelog generation.

### Format

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Type

Must be one of:

- **feat**: New feature (triggers minor version bump)
- **fix**: Bug fix (triggers patch version bump)
- **docs**: Documentation changes
- **style**: Code style changes (formatting, no logic change)
- **refactor**: Code refactoring (no features or bugs)
- **perf**: Performance improvements
- **test**: Adding or updating tests
- **chore**: Build process, dependencies, tooling
- **ci**: CI/CD configuration changes

### Scope (Optional)

The scope specifies the area of the codebase:

- `core`: Core functionality (camouflage.ts)
- `config`: Configuration management
- `decorators`: Decorator functions
- `patterns`: Pattern matching
- `tests`: Test files
- `docs`: Documentation

### Subject

- Imperative mood ("add" not "added" or "adds")
- Lowercase first letter
- No period at the end
- Max 50 characters

### Breaking Changes

Breaking changes trigger a major version bump:

```
feat(core): redesign pattern matching API

BREAKING CHANGE: matchPattern now returns object instead of boolean
```

### Examples

```bash
# Feature (bumps 1.0.0 → 1.1.0)
feat(core): add support for custom hiding characters

# Bug fix (bumps 1.0.0 → 1.0.1)
fix(patterns): handle special regex characters correctly

# Documentation
docs: update README with new configuration options

# Refactoring
refactor(core): extract decoration logic into separate method

# Performance
perf(patterns): cache compiled regexes for pattern matching

# Breaking change (bumps 1.0.0 → 2.0.0)
feat(core): redesign configuration API

BREAKING CHANGE: Configuration keys changed from camelCase to snake_case
```

### Body (Optional)

- Detailed explanation of the change
- Motivation for the change
- Contrast with previous behavior

```
feat(core): add selective hiding by pattern

This allows users to hide only specific environment variables
by providing wildcard patterns like *API*, *SECRET, etc.

Previously, all values were hidden or none were hidden.
```

### Footer (Optional)

- Reference issues
- Note breaking changes
- Credit co-authors

```
fix(patterns): escape special regex characters

Fixes #45
```

## Branching Strategy

### Main Branches

- **`main`**: Production-ready code, protected branch
- **`develop`**: Integration branch (optional for larger teams)

### Feature Branches

```bash
# Create feature branch from main
git checkout main
git pull origin main
git checkout -b feature/add-custom-characters

# Work on feature
git add .
git commit -m "feat(core): add custom hiding characters"

# Push to remote
git push origin feature/add-custom-characters
```

**Naming**:

- `feature/*` - New features
- `fix/*` - Bug fixes
- `docs/*` - Documentation
- `refactor/*` - Code refactoring
- `test/*` - Test improvements
- `chore/*` - Tooling, dependencies

### Bug Fix Branches

```bash
# Create from main
git checkout -b fix/regex-escape-issue

# Fix and commit
git commit -m "fix(patterns): escape special characters in patterns"

# Push
git push origin fix/regex-escape-issue
```

### Hotfix Branches

For urgent production fixes:

```bash
# Create from main
git checkout -b hotfix/security-vulnerability

# Fix and commit with appropriate type
git commit -m "fix(security): sanitize user input in pattern matching"

# Push
git push origin hotfix/security-vulnerability
```

## Pull Request Process

### Creating a PR

1. **Push branch to GitHub**

```bash
git push origin feature/my-feature
```

2. **Create PR on GitHub**

   - Base: `main`
   - Compare: `feature/my-feature`
   - Fill in PR template

3. **PR Title**: Should match commit message format

```
feat(core): add custom hiding characters
```

4. **PR Description**: Include:
   - What changed
   - Why it changed
   - How to test
   - Screenshots (if UI changes)
   - Related issues

### PR Template

```markdown
## Description

Brief description of changes

## Type of Change

- [ ] Bug fix (fix)
- [ ] New feature (feat)
- [ ] Breaking change (BREAKING CHANGE)
- [ ] Documentation update (docs)

## Testing

- [ ] All tests pass (`npm test`)
- [ ] No linting errors (`npm run lint`)
- [ ] Manual testing completed

## Checklist

- [ ] Code follows project style guidelines
- [ ] Self-review completed
- [ ] Comments added for complex logic
- [ ] Documentation updated
- [ ] Tests added/updated
- [ ] No console.log statements left
```

### PR Review Guidelines

**For Authors**:

- Keep PRs small and focused (< 400 lines)
- Write clear commit messages
- Add tests for new features
- Update documentation
- Respond to feedback promptly

**For Reviewers**:

- Check code quality and style
- Verify tests are adequate
- Look for potential bugs
- Suggest improvements
- Be constructive and respectful

### Merging

- **Squash and merge** is preferred for feature branches
- Ensure final commit message follows conventional format
- Delete branch after merge

## Semantic Release Process

### Automated Releases

We use `semantic-release` for automated versioning and releases.

**Workflow**:

1. PR merged to `main`
2. GitHub Actions runs `semantic-release`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zeybek/camouflage](https://github.com/zeybek/camouflage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
