---
trigger: always_on
description: - After code changes (not documentation changes): `pnpm check` (get full output, no tail). Fix all errors, warnings, and infos before committing.
---

# Development Rules

## Commands
- After code changes (not documentation changes): `pnpm check` (get full output, no tail). Fix all errors, warnings, and infos before committing.
- Note: `pnpm check` does not run tests.
- NEVER run: `pnpm dev`, `pnpm build`, `pnpm test`
- Only run specific tests if user instructs: `pnpx tsx ../../node_modules/vitest/dist/cli.js --run test/specific.test.ts`
- If you create or modify a test file, you MUST run that test file and iterate until it passes.
- When writing tests, run them, identify issues in either the test or implementation, and iterate until fixed.
- NEVER commit unless user asks

## Style
- Keep answers short and concise
- No emojis in commits, issues, PR comments, or code
- No fluff or cheerful filler text
- Technical prose only, be kind but direct (e.g., "Thanks @user" not "Thanks so much @user!")

## Changelog
Location: `CHANGELOG.md`

### Format
Use these sections under `## [Unreleased]`:
- `### Breaking Changes` - API changes requiring migration
- `### Added` - New features
- `### Changed` - Changes to existing functionality
- `### Fixed` - Bug fixes
- `### Removed` - Removed features

### Rules
- Before adding entries, read the full `[Unreleased]` section to see which subsections already exist
- New entries ALWAYS go under `## [Unreleased]` section
- Append to existing subsections (e.g., `### Fixed`), do not create duplicates
- NEVER modify already-released version sections (e.g., `## [0.12.2]`)
- Each version section is immutable once released

### Attribution
- **Internal changes (from issues)**: `Fixed foo bar ([#123](https://github.com/aamdmn/payload-agent/issues/123))`
- **External contributions**: `Added feature X ([#456](https://github.com/aamdmn/payload-agent/pull/456) by [@username](https://github.com/username))`

## Release

Publish, git tag, and GitHub release are a single command so they cannot drift apart (0.9.0 and 0.10.0 reached npm but were never tagged/released because this step was manual).

1. In the feature PR: bump `version` in `package.json` and move the `[Unreleased]` entries into a new `## [x.y.z] - YYYY-MM-DD` CHANGELOG section.
2. Merge to `main`, then from a clean, up-to-date `main` run:
   - `pnpm release` — releases the current `package.json` version
   - `pnpm release "short summary"` — same, with a custom GitHub release title (default title is `vx.y.z`)
   - `pnpm release --dry-run` — runs every check and prints the plan, changes nothing

`scripts/release.mjs` runs all read-only checks first (on `main`, clean tree, in sync with origin, tag does not already exist, version not already on npm, `gh` installed + authed), then `pnpm publish` (pnpm-only — see `ensure-pnpm.mjs`), then creates and pushes the annotated tag, then the GitHub release. Release notes are pulled from the matching `CHANGELOG.md` section, so the changelog is the single source of truth — do not hand-write release notes.

## **CRITICAL** Tool Usage Rules **CRITICAL**
- NEVER use sed/cat to read a file or a range of a file. Always use the read tool (use offset + limit for ranged reads).
- You MUST read every file you modify in full before editing.

## **CRITICAL** Git Rules for Parallel Agents **CRITICAL**

Multiple agents may work on different files in the same worktree simultaneously. You MUST follow these rules:

### Committing
- **ONLY commit files YOU changed in THIS session**
- ALWAYS include `fixes #<number>` or `closes #<number>` in the commit message when there is a related issue or PR
- NEVER use `git add -A` or `git add .` - these sweep up changes from other agents
- ALWAYS use `git add <specific-file-paths>` listing only files you modified
- Before committing, run `git status` and verify you are only staging YOUR files
- Track which files you created/modified/deleted during the session

### Forbidden Git Operations
These commands can destroy other agents' work:
- `git reset --hard` - destroys uncommitted changes
- `git checkout .` - destroys uncommitted changes
- `git clean -fd` - deletes untracked files
- `git stash` - stashes ALL changes including other agents' work
- `git add -A` / `git add .` - stages other agents' uncommitted work
- `git commit --no-verify` - bypasses required checks and is never allowed

### Safe Workflow
```bash
# 1. Check status first
git status

# 2. Add ONLY your specific files
git add packages/ai/src/providers/transform-messages.ts
git add packages/ai/CHANGELOG.md

# 3. Commit
git commit -m "fix(ai): description"

# 4. Push (pull --rebase if needed, but NEVER reset/checkout)
git pull --rebase && git push
```

### If Rebase Conflicts Occur
- Resolve conflicts in YOUR files only
- If conflict is in a file you didn't modify, abort and ask the user
- NEVER force push

### User override
If the user instructions conflict with rules set out here, ask for confirmation that they want to override the rules. Only then execute their instructions.


# Payload Plugin Development

This project is a **Payload CMS plugin** (`payload-agent`). All source code lives in `src/`, with a `dev/` directory for local testing.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aamdmn/payload-agent](https://github.com/aamdmn/payload-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
