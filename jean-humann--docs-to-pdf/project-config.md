---
trigger: always_on
description: This document provides specific guidance for AI assistants (particularly Claude) when working on the docs-to-pdf project.
---

# Claude Development Guide for docs-to-pdf

This document provides specific guidance for AI assistants (particularly Claude) when working on the docs-to-pdf project.

## Project Overview

docs-to-pdf is a TypeScript-based tool that generates PDFs from documentation websites (like Docusaurus). It uses Puppeteer to crawl pages, combine them into a single HTML document, and generate a PDF.

## Development Environment Setup

### Using mise (Recommended)

This project uses [mise](https://mise.jit.su/) to manage the Node.js version:

```bash
# Install mise (if not already installed)
# See: https://mise.jit.su/getting-started.html

# Initialize the project - this will:
# - Read mise.toml
# - Install the specified Node.js version
# - Set it up for this project
mise install

# Verify Node.js version
node --version  # Should match version in mise.toml
```

The `mise.toml` file in the project root defines:

- Required Node.js version
- Any other development tool versions

### Manual Setup

If not using mise:

```bash
# Ensure Node.js >= 20.0.0 is installed
yarn install
yarn build
```

## Pre-Commit Checklist

Before committing ANY changes, you MUST:

### 1. Run Tests

```bash
yarn test
```

**Result**: All tests must pass. If any test fails, fix the issue before committing.

### 2. Run Linter

```bash
yarn lint
```

**Result**: No linting errors. Fix any issues with:

```bash
yarn lint:fix
```

### 3. Build TypeScript

```bash
yarn build
```

**Result**: TypeScript must compile without errors. The `lib/` directory will be generated.

### 4. Docker Tests (if modifying Docker files)

If you modified any Docker-related files:

```bash
cd docker
./test-quick.sh
```

**Result**: At least the quick tests should pass.

## Commit Message Requirements

### DO NOT Update CHANGELOG.md Manually

This project uses [release-please](https://github.com/googleapis/release-please) to:

- Automatically generate CHANGELOG.md
- Bump version numbers
- Create releases
- Trigger npm/Docker publishing

**NEVER** manually edit:

- `CHANGELOG.md`
- Version numbers in `package.json` (unless explicitly required for a specific reason)

### Use Conventional Commits

All commit messages MUST follow the [Conventional Commits](https://www.conventionalcommits.org/) format:

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

#### Common Types

- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `refactor`: Code refactoring
- `test`: Test additions or modifications
- `chore`: Maintenance tasks
- `ci`: CI/CD changes

#### Examples

```bash
feat(docker): add multi-Node.js version support

Added Docker images for Node 20, 22, and 24 with both Alpine and Debian variants.

fix: handle non-clickable details elements

Improved openDetails() to gracefully handle elements that cannot be clicked,
preventing failures on complex documentation sites.

docs: update README with Docker usage examples

test(e2e): add comprehensive Docker E2E tests

Created test suite covering all Node versions and Docusaurus versions.
```

#### Breaking Changes

For breaking changes, add `!` or `BREAKING CHANGE:` in footer:

```bash
feat!: require Node.js 20 or higher

BREAKING CHANGE: Dropped support for Node.js 18. Users must upgrade to Node.js 20+.
```

## Release Process

### Automated via release-please

This project uses [release-please-action](https://github.com/googleapis/release-please-action) (GitHub Action) to automate releases.

Configuration files:

- `release-please-config.json` - Release strategy and package configuration
- `.release-please-manifest.json` - Tracks current version

Workflow:

1. Commits merged to `master` → release-please GitHub Action analyzes them
2. release-please creates/updates a Release PR
3. When Release PR is merged → automated release happens:
   - CHANGELOG.md updated
   - package.json version bumped
   - GitHub release created
   - npm package published (if configured)
   - Docker images published

**Important:** The workflow is configured to use a Personal Access Token (`RELEASE_PLEASE_TOKEN`) to trigger downstream workflows like the Publish workflow. If this secret is not configured, it falls back to `GITHUB_TOKEN`, which **will not trigger** the `publish.yml` workflow due to GitHub's security restrictions.

#### Setting up the Personal Access Token

To enable automatic publishing when releases are created:

1. **Create a Personal Access Token (Classic)**:
   - Go to GitHub Settings → Developer settings → Personal access tokens → Tokens (classic)
   - Click "Generate new token (classic)"
   - Give it a descriptive name (e.g., "release-please-workflow")
   - Select scopes:
     - `repo` (Full control of private repositories)
     - `workflow` (Update GitHub Action workflows)
   - Set an expiration date (recommended: 90 days or 1 year)
   - Click "Generate token" and copy it immediately

2. **Add the token as a repository secret**:
   - Go to your repository → Settings → Secrets and variables → Actions
   - Click "New repository secret"
   - Name: `RELEASE_PLEASE_TOKEN`
   - Value: Paste your PAT
   - Click "Add secret"

3. **How it works**:
   - When configured, release-please uses this PAT to create releases

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jean-humann/docs-to-pdf](https://github.com/jean-humann/docs-to-pdf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
