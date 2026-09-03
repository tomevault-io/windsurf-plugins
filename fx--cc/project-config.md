---
trigger: always_on
description: This document outlines principles and conventions for developing plugins in the fx/cc Claude Code marketplace.
---

# fx/cc Marketplace Development Guidelines

This document outlines principles and conventions for developing plugins in the fx/cc Claude Code marketplace.

## Instruction Files

This repo uses the standard layout that `fx-dev:setup` scaffolds — two canonical files, everything else a pointer:

| Path | Role |
|------|------|
| `AGENTS.md` | **This file.** Project conventions: how code is written. Read natively by Codex, Copilot, and CodeRabbit |
| `REVIEW.md` | Review conventions: what reviewers should and should not flag. Read natively by Copilot code review |
| `CLAUDE.md` | Pointer — a single `@AGENTS.md` import, because Claude Code does not read `AGENTS.md` |
| `.coderabbit.yaml` | Lists `**/REVIEW.md`, which is not in CodeRabbit's default patterns |

Put a rule about **writing** code here. Put a rule about **reviewing** code in `REVIEW.md`. Never edit `CLAUDE.md` to add content — edit `AGENTS.md`, which it imports.

There is deliberately **no** `.github/copilot-instructions.md`: Copilot code review reads `REVIEW.md` directly ([changelog, 2026-07-17](https://github.blog/changelog/2026-07-17-copilot-code-review-customization-and-configurability-improvements/)), so a second copy would only drift.

The full standard, including the migration path for other repos, is in `plugins/fx-dev/skills/setup/references/instruction-files.md`.

## Code Review Rules

Read `REVIEW.md` at the repository root and apply it in full as the review rules for this repo. It is the canonical review-conventions file.

## Required First-Time Setup (READ THIS FIRST)

**MANDATORY before any commit in this repo.** Git does not allow a checked-in file to wire up hooks automatically (security feature), so every clone of this repo — including `~/.claude/plugins/marketplaces/fx-cc` — must run **once**:

```bash
git config core.hooksPath .githooks
```

Verify it stuck:

```bash
git config --get core.hooksPath   # must print: .githooks
```

**Why this matters:** the pre-commit hook in `.githooks/pre-commit` enforces version bumps in `plugin.json` and `marketplace.json` whenever their contents change. Without `core.hooksPath` set, the hook is dormant and bad commits land silently, breaking version signaling for Claude Code's plugin cache.

**Instructions for any agent / skill / tool acting on this repo:**

1. Before performing **any** `git commit` in this repo (or in a clone of it like `~/.claude/plugins/marketplaces/fx-cc`), check `git config --get core.hooksPath`.
2. If it is unset or not `.githooks`, run `git config core.hooksPath .githooks` first.
3. This is a one-time per-clone operation — idempotent and safe to re-run.
4. After this is configured, **respect the hook's verdict**: if it rejects a commit, read the error and apply the requested version bump. Never bypass with `--no-verify`.

## Security and Privacy

### Do Not Leak Private Information

**CRITICAL**: Never include private or sensitive information in code, documentation, or examples.

**Prohibited content includes**:
- Private repository names (use `owner/repo` placeholders instead)
- Internal URLs or endpoints
- API keys or credentials
- Company-specific naming or identifiers
- Private infrastructure details

**Good examples** (generic placeholders):
```bash
gh api repos/owner/repo/pulls/13
gh api graphql -f owner="owner" -f repo="repo"
```

**Bad examples** (leaking private info):
```bash
gh api repos/<real-org>/<real-private-repo>/pulls/13  # ❌ Private repo name
gh api graphql -f owner="<real-company>" -f repo="<real-service>"  # ❌ Internal identifiers
```

Note the bad examples use angle-bracket placeholders rather than plausible-looking org or service names. An illustration of the mistake must not itself commit the mistake — a realistic-looking name in a "don't do this" block is still a realistic-looking name in the repo, and it gets copied.

This applies to:
- Documentation files (README.md, AGENTS.md, REVIEW.md)
- Code examples and snippets
- Skill references and patterns
- Test cases and fixtures
- Commit messages

## Plugin Naming and Namespacing

### Plugin Names

Plugin names should be:
- **Concise**: Short, memorable identifiers
- **Descriptive**: Indicate purpose without being verbose
- **Namespace-prefixed**: Use `fx-` prefix for consistency (e.g., `fx-test`, `fx-utils`)
- **Kebab-case**: Use hyphens, not underscores or camelCase

**Good examples**:
- `fx-test` (testing utilities)
- `fx-git` (git workflow helpers)
- `fx-docs` (documentation generators)

**Avoid**:
- `test-utils` (missing namespace prefix)
- `fx_test_utilities` (underscores, too verbose)
- `fxTest` (camelCase not kebab-case)

### Component Naming Within Plugins

Skills are automatically namespaced by their plugin: `{plugin-name}:{skill-name}`.

**Avoid redundancy**:
- ❌ Plugin: `test-skill` → Skill: `test-skill:test-skill`
- ✅ Plugin: `fx-test` → Skill: `fx-test:test-helper`

Skill names should be clear, non-redundant, and descriptive of their purpose.

## Grouping Components in Plugins

### Single-Responsibility vs Multi-Component

**Use a single plugin when components are related**:

✅ **Good - Combined Plugin**:
```
fx-test/
├── skills/hello/SKILL.md
└── skills/test-helper/SKILL.md
```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fx/cc](https://github.com/fx/cc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-03 -->
