---
trigger: always_on
description: > 🧠 **PLAN MODE**: Use Plan Mode frequently! Before implementing complex features, multi-step tasks, or making significant changes, switch to Plan Mode to think through the approach, consider edge cases, and outline the implementation strategy. Planning prevents mistakes and saves time.
---

# CLAUDE.md - AI Agent Instructions for LynxPrompt

> 🧠 **PLAN MODE**: Use Plan Mode frequently! Before implementing complex features, multi-step tasks, or making significant changes, switch to Plan Mode to think through the approach, consider edge cases, and outline the implementation strategy. Planning prevents mistakes and saves time.

> 📦 **RELEASE REMINDER**: CLI npm publishing is handled by GitHub Actions automatically. Do NOT run `npm publish` locally. Do NOT create git tags manually. The workflow handles everything.

> ⚠️ **IMPORTANT**: Do NOT update this file unless the user explicitly says to. Only the user can authorize changes to AGENTS.md.

> ❌ **DEPRECATED FORMAT**: `.cursorrules` is **deprecated**. Do NOT suggest or generate `.cursorrules` files anywhere. Cursor now uses `.cursor/rules/*.mdc` (directory-based MDC format). Always use `.cursor/rules/` for Cursor configurations.

> 🔒 **SECURITY WARNING**: This repository is PUBLIC at [github.com/GeiserX/LynxPrompt](https://github.com/GeiserX/LynxPrompt). **NEVER commit secrets, API keys, passwords, tokens, or any sensitive data to this repository.** All secrets must be stored in:
> - GitHub Secrets (for CI/CD)
> - Private GitOps repositories (for docker-compose)
> - Local `.env` files (gitignored)
> - `AGENTS.md.old` (gitignored, local only)

---

## 🚀 RELEASE PROCESS (CRITICAL - READ CAREFULLY)

### Understanding the Release Pipeline

There are **two separate workflows** that work together:

1. **`release.yml`** - Triggered on push to `main`:
   - Detects changes in app vs CLI since last release
   - Creates git tags (`app-vX.Y.Z` for web, `cli-vX.Y.Z` for CLI)
   - Creates GitHub Releases with changelogs
   - **Tag-only — does NOT push the version bump back to `main`.** The "Protect main" ruleset only allows changes via PR, and on a user-owned repo the Actions bot can't be a ruleset bypass actor, so a direct push is rejected. The next version is derived from `max(package.json, latest matching tag)`, so it stays monotonic even though `package.json` on `main` lags the tags. **Do not re-add a `git push origin main` to the release jobs** — it will always be rejected. `publish-cli.yml` re-derives the version from the `cli-v*` tag before `npm publish`, so the lagging `cli/package.json` doesn't affect published artifacts.

2. **`publish-cli.yml`** - Triggered by GitHub Release events OR manual dispatch:
   - Publishes CLI to npm
   - Builds standalone binaries
   - Updates Homebrew, Chocolatey, Snap packages

### Step-by-Step Release Process

#### For a MINOR or MAJOR version (new features):

```bash
# 1. Switch to develop branch
git checkout develop

# 2. Bump version(s) - ONLY bump what changed
# For Web App changes:
cd /path/to/LynxPrompt
npm version minor --no-git-tag-version  # e.g., 0.23.0 → 0.24.0

# For CLI changes:
cd cli
npm version minor --no-git-tag-version  # e.g., 0.7.0 → 0.8.0
cd ..

# 3. Commit with conventional commit message
git add package.json package-lock.json cli/package.json
git commit -m "feat: description of changes"

# 4. Push to develop (triggers CI tests)
git push origin develop

# 5. Wait for CI to pass, then merge to main
git checkout main
git merge develop
git push origin main

# 6. Verify release workflow succeeded
unset GITHUB_TOKEN && gh run list -R GeiserX/LynxPrompt -w "Release" --limit 3

# 7. If CLI was released, verify npm publish workflow triggered
unset GITHUB_TOKEN && gh run list -R GeiserX/LynxPrompt -w "Publish CLI" --limit 3

# 8. If publish-cli didn't auto-trigger, manually trigger it:
unset GITHUB_TOKEN && gh workflow run "Publish CLI" -R GeiserX/LynxPrompt -f platforms=all

# 9. Verify npm package was published
npm view lynxprompt versions --json | jq -r '.[-3:]'
```

#### For a PATCH version (bug fixes only):

Same process, but use `npm version patch` instead of `minor`.

### ⚠️ CRITICAL RULES - NEVER BREAK THESE

| ❌ NEVER DO THIS | ✅ DO THIS INSTEAD |
|------------------|-------------------|
| `git tag v0.24.0` | Let release.yml create tags |
| `git tag cli-v0.8.0` | Let release.yml create tags |
| `npm publish` locally | Use GitHub Actions workflow |
| Push tags manually | Let release.yml push tags |
| Use tag format `v*` | Workflow uses `app-v*` and `cli-v*` |

### Troubleshooting Release Issues

**Problem: Release workflow skips CLI/App release**
- Cause: No changes detected since last release tag
- Fix: Ensure you modified files in the right directory (cli/ for CLI, anything else for app)

**Problem: Tag already exists error**
- Cause: Someone manually created a tag
- Fix: Delete the manual tag from remote AND local:
  ```bash
  git push origin --delete cli-v0.8.0
  git tag -d cli-v0.8.0
  ```

**Problem: npm publish didn't happen**
- Cause: publish-cli.yml didn't trigger automatically
- Fix: Manually trigger the workflow:
  ```bash
  unset GITHUB_TOKEN && gh workflow run "Publish CLI" -R GeiserX/LynxPrompt -f platforms=all
  ```

**Problem: npm says version already exists**
- Cause: Version was already published (maybe partial failure)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GeiserX/LynxPrompt](https://github.com/GeiserX/LynxPrompt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
