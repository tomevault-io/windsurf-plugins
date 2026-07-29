---
trigger: always_on
description: TypeScript/Svelte 4 monorepo using Rush.js (pnpm 10.15.1), Node >=20 <25, Webpack 5, Electron, Jest.
---

# Foundation Platform - AI Agent Instructions

TypeScript/Svelte 4 monorepo using Rush.js (pnpm 10.15.1), Node >=20 <25, Webpack 5, Electron, Jest.

## Interaction preferences

Respond to user using Russian language, all comments should be in English.

## Code Style

**TypeScript**: Strict types, interfaces over types, avoid `any`, export types separately
**Svelte**: Script/style/markup order, reactive `$:`, stores for state, small focused components
**Naming**: Files `kebab-case`, Components `PascalCase`, functions `camelCase`, constants `UPPER_SNAKE_CASE`

## Structure

- `models/*` - Shared types/models
  - If add resources like component: '' as AnyComponent, ensure is it added into one of api package/resources package or model package. It should be defined only once. It could be IntlString as well or any other '' as Ref<something> declaration.
- `server-*` - Server packages
- `plugins/*` - Client plugins
- `packages/*` - Reusable utilities
- Projects 2-3 levels deep, each with `package.json`

## Rush Commands

```bash
rush install         # Install deps
rush build           # Build all
rush build --to PKG  # Build specific
rush add -p PKG      # Add dependency
```

## Docker Build Workflow

**IMPORTANT**: After making changes to service code (in `services/`, `pods/`, etc.), you must rebuild Docker images:

**Note**: If you're running the UI via `rush dev` (dev-server), you don't need to restart the `front` Docker container. Changes will be picked up automatically by the dev server.

```bash
# Build Docker images for specific service
rush docker:build --to @hcengineering/pod-ai-bot
rush docker:build --to @hcengineering/love-agent

# Restart Docker containers to use new images
docker compose -f dev/docker-compose.yaml restart aibot
docker compose -f dev/docker-compose.yaml restart love-agent
```

**Workflow for service changes:**
1. Make code changes to service
2. Run `rush build --to <package>` (builds TypeScript)
3. Never Run `rushx format` in the package directory (format & lint) - MAY CAUSE FILE CORRUPTIONS. Lets user do it.
4. Run `diagnostics` to check for errors
5. Run `rush docker:build --to <package>` (builds Docker image)
6. Restart the Docker container

Example for ai-bot service:
```bash
# After editing services/ai-bot/pod-ai-bot/src/workspace/love.ts
cd services/ai-bot/pod-ai-bot
diagnostics path: "services/ai-bot/pod-ai-bot/src/workspace/love.ts"
cd ../../..
rush docker:build --to @hcengineering/pod-ai-bot
docker compose -f dev/docker-compose.yaml restart aibot
```

## Error Checking

**IMPORTANT**: Use `diagnostics` tool to check for TypeScript/Svelte errors, NOT `rush build`:

- ✅ `diagnostics()` - Check all files for errors/warnings (fast, uses language server)
- ✅ `diagnostics({ path: "plugins/tracker-resources/src/utils.ts" })` - Check specific file
- ❌ `rush build` - Don't use for error checking (runs full transpilation, slower)

`rush build` performs transpilation which may succeed even with type errors. Always use `diagnostics` to verify code correctness.

### Validation in Modified Projects

After making changes, always validate the affected packages to ensure diagnostics are accurate:

```bash
# Navigate to each modified package and run:
cd <modified-package-directory>
rushx build
rushx _phase:validate

# Examples:
cd plugins/love-resources
rushx build
rushx _phase:validate

cd models/love
rushx build
rushx _phase:validate

cd server-plugins/love-resources
rushx build
rushx _phase:validate
```

This ensures that the language server has the correct build artifacts and validation passes for the specific packages you modified.

## Formatting and Linting

**AI AGENTS: DO NOT run formatting commands automatically.** Formatting can corrupt or erase files. Let the user handle formatting.

This ensures code style consistency and catches linting errors before commit.

### ⚠️ CRITICAL: Formatting Safety Rules

**NEVER run formatting commands in parallel or concurrently.** The formatter can corrupt or completely erase file contents when run simultaneously on multiple packages.

Rules:
- ❌ **DO NOT** run `rushx format` commands
- ❌ **DO NOT** use `--force` flag with formatting - it can cause content loss
- ✅ **DO** run formatting sequentially, one package at a time
- ✅ **DO** verify file contents after formatting with `git diff` or `git status`
- ✅ **DO** restore files immediately with `git checkout -- <file>` if content is lost

If you see files becoming empty or losing content after formatting, immediately restore them:
```bash
git checkout -- <affected-files>
```

## Changelog generation

When generating changelogs (the "All commits" lists), follow these rules:

- Exclude commits whose subject contains `Merge remote-tracking` (filter them out).
- Strip `Signed-off-by:` footers from commit messages (remove the footer content and any lines that are only `Signed-off-by:`).
- Recommended pipeline (example):

```bash
git log --pretty=format:'- %h %s' <range> | grep -v -F 'Merge remote-tracking' | sed -E 's/\s*Signed-off-by:.*$//'
```

- Note: `git log --no-merges` removes all merge commits; use it only if you intentionally want to omit all merges.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [intabia-fusion/platform](https://github.com/intabia-fusion/platform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
