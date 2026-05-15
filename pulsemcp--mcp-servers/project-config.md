---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Also review [CONTRIBUTING.md](./CONTRIBUTING.md) for context.

## Repository Overview

This is a monorepo containing Model Context Protocol (MCP) servers built by PulseMCP. Each subfolder represents a standalone MCP server with its own functionality.

## Repository Structure

- **`productionized/`**: Directory for production-ready MCP servers
- **`experimental/`**: Directory for experimental MCP servers in development
- **`libs/mcp-server-template/`**: Template structure for creating new MCP servers

## Git Workflow

- Repository: `https://github.com/pulsemcp/mcp-servers`
- Branch naming: `<github-username>/<feature-description>` (e.g., `tadasant/fix-bug`)
- Main branch has CI/CD
- Always include test coverage for changes
- PRs should have concise titles and detailed descriptions using this format:

### PR Description Format

**This overrides the default Claude Code PR template.** Do NOT use a `## Test plan` section with unchecked checkboxes. Instead, use this format:

```
## Summary
<what changed and why>

## Verification
- [x] <what you actually did to verify this works>
- [x] <proof: concrete evidence it works — not just an assertion>
```

The `## Verification` section documents how you closed the loop — what you _actually did_, not what _should be done_. Every checkbox must be checked before the PR is opened. If you can't verify something, explain why instead of leaving an unchecked box.

### Proof: Show, Don't Tell

Every verification item should include **proof** — concrete evidence that the change works. "Tested it and it works" is NOT proof. A screenshot, a test output, or a confirmation receipt IS proof.

**Proof types and when to use them:**

1. **E2E test report** — For backend/logic changes. Describe what you tested end-to-end and what happened. This is the most common type.
2. **Screenshot** — For UI changes. **UI changes MUST include screenshots. No exceptions.**
3. **External confirmation** — For tasks with external side effects (API calls, emails sent, deployments). Show the confirmation or response.

Good examples:

- `[x] E2E: created a session via the API, verified it appeared in the dashboard with correct metadata`
- `[x] Screenshot of updated settings page: ![settings](url)`
- `[x] Ran migration on staging, verified column exists: `SELECT column_name FROM information_schema.columns WHERE table_name = 'users';``
- `[x] Sent test email via new endpoint, confirmed delivery in Mailgun logs`
- `[x] CI green (lint + tests pass)`
- `[x] Self-reviewed PR diff — no unintended changes`

Bad examples (NEVER do this):

- `[ ] CI passes` — unchecked box, aspirational
- `[ ] Verify the server works end-to-end` — unchecked box, aspirational
- `[x] Tested it and it works` — this is an assertion, not proof. What did you test? What happened?
- `[x] Verified the feature works correctly` — says nothing. Show what you did and what you saw

Unchecked boxes in a PR description are useless — they describe aspirational work that nobody will do. Checked boxes without proof are almost as bad — they're assertions without evidence. Close the loop with concrete proof before handing the PR to a human.

### IMPORTANT: Git Branch Management

**DO NOT** create new git branches or worktrees unless explicitly asked by the user. Always:

- Stay on the current branch you're working on
- Make all changes directly on the existing branch
- Only switch branches or create new ones when specifically instructed
- Avoid using `git checkout -b`, `git switch -c`, or `git worktree add` without explicit permission

### Linting and Pre-Commit Hooks

**CRITICAL: ALL linting must be run from the repository root.** This monorepo uses centralized linting configuration.

**Always run these commands from the repo root before pushing to avoid CI failures:**

```bash
npm run lint       # Check for linting issues
npm run lint:fix   # Auto-fix linting issues
npm run format     # Format code with Prettier
```

**IMPORTANT: NEVER use `git commit --no-verify` to bypass pre-commit hooks.** If pre-commit hooks fail:

### Troubleshooting Pre-Commit Hook Failures

**🔨 Module/Dependency Issues** (Most common - "Cannot find module" errors):

```bash
# Always run from repo root
cd /path/to/repo/root
rm -rf node_modules
npm install
```

**📝 Linting Issues:**

```bash
npm run lint:fix    # From repo root only
```

**🎨 Formatting Issues:**

```bash
npm run format      # From repo root only
```

**📁 Committing from Subdirectories:**

```bash
# Instead of committing from experimental/twist/ or other subdirs:
cd /path/to/repo/root
git add .
git commit -m "Your message"
```

**Why These Issues Happen:**

- Monorepo complexity with nested workspaces
- Module resolution conflicts between subdirectories
- Stale or corrupted dependency trees

The repository uses:

- **ESLint** for code quality and style enforcement
- **Prettier** for consistent code formatting
- **Husky** for git hooks (pre-commit runs lint-staged automatically)
- **lint-staged** for running linters on staged files

## Common Development Commands

Most MCP servers in this repo follow these conventions:

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pulsemcp/mcp-servers](https://github.com/pulsemcp/mcp-servers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
