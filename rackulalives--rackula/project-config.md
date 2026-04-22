---
trigger: always_on
description: **Project:** Rackula — Rack Layout Designer for Homelabbers
---

# CLAUDE.md — Rackula

**Project:** Rackula — Rack Layout Designer for Homelabbers
**Version:** 0.7.6

---

## Versioning Policy

We follow [Cargo semver](https://doc.rust-lang.org/cargo/reference/semver.html) conventions:

**Pre-1.0 semantics (current):**

- `0.MINOR.patch` — minor version acts like major (breaking changes allowed)
- `0.minor.PATCH` — bug fixes and small improvements only
- Pre-1.0 means "API unstable, in active development"

**When to bump versions:**

| Change Type        | Version Bump | Example                                                    |
| ------------------ | ------------ | ---------------------------------------------------------- |
| Feature milestone  | `0.X.0`      | New major capability (e.g., multi-rack, new export format) |
| Bug fixes / polish | `0.x.Y`      | Only when releasing to users, not every commit             |
| Breaking changes   | `0.X.0`      | Format changes, removed features                           |

**Workflow:**

- **Don't tag every commit** — accumulate changes on `main`
- **Tag releases** when there's a coherent set of changes worth announcing
- **Use pre-release tags** for development checkpoints: `v0.5.0-alpha.1`, `v0.5.0-beta.1`
- **Batch related fixes** into single patch releases

**Release Process:**

Use the `/release` skill to create releases with proper changelog entries:

```bash
/release patch   # Bug fixes: 0.5.8 → 0.5.9
/release minor   # Features: 0.5.9 → 0.6.0
/release major   # Breaking: 0.6.0 → 1.0.0
/release 1.0.0   # Explicit version
```

The `/release` skill will:

1. Gather changes since last release (commits, PRs, issues)
2. Draft a changelog entry in Keep a Changelog format
3. Preview and confirm with you
4. Update CHANGELOG.md, bump version, tag, and push

**Important:** CHANGELOG.md is the source of truth. GitHub releases are auto-generated
from changelog entries. The release workflow will fail if no changelog entry exists.

**Tag format:** Always use `v` prefix (e.g., `v0.5.8`, not `0.5.8`)

**Current milestones:**

- `0.5.x` — Unified type system, NetBox-compatible data model
- `1.0.0` — Production-ready, stable API

---

## Documentation

Documentation is organized by purpose:

```text
docs/
├── ARCHITECTURE.md          → High-level overview and entry points
├── deployment/              → Deployment-specific docs (auth, hosting)
├── guides/
│   ├── TESTING.md           → Testing patterns and commands
│   └── ACCESSIBILITY.md     → A11y compliance checklist
├── reference/
│   ├── SPEC.md              → Technical overview and design principles
│   ├── BRAND.md             → Design system quick reference
│   └── GITHUB-WORKFLOW.md   → GitHub Issues workflow
├── planning/
│   └── ROADMAP.md           → Version planning
├── plans/                   → Implementation plans (YYYY-MM-DD-kebab-case.md)
├── research/                → Research spikes by issue ({ISSUE}-{type}.md)
├── spikes/                  → Active spike investigations
└── superpowers/
    └── specs/               → Brainstorming design specs (created on first use)
```

**Start here:** `docs/ARCHITECTURE.md` for codebase overview.
**Reference:** `docs/reference/SPEC.md` for technical overview and design principles.

### Design Documents

Project overrides for Superpowers v5 document locations:

| Document Type          | Location                  | Naming Convention                  |
| ---------------------- | ------------------------- | ---------------------------------- |
| Specs (brainstorming)  | `docs/superpowers/specs/` | `YYYY-MM-DD-<topic>-design.md`     |
| Plans (execution)      | `docs/plans/`             | `YYYY-MM-DD-<feature-name>.md`     |
| Research spikes        | `docs/research/`          | `{ISSUE}-{type}.md`                |

Plans use `docs/plans/` (project override — v5 defaults to `docs/superpowers/plans/`).

## GitHub Issues Workflow

GitHub Issues is the source of truth for task tracking.

**Querying work:**

```bash
# Find next task
gh issue list --label ready --milestone v0.6.0 --state open

# Get issue details
gh issue view <number>
```

**After completing an issue:**

```bash
gh issue close <number> --comment "Implemented in <commit-hash>"
```

**Issue structure provides:**

- Acceptance Criteria → Requirements checklist
- Technical Notes → Implementation guidance
- Test Requirements → TDD test cases

See `docs/reference/GITHUB-WORKFLOW.md` for full workflow documentation.

## CodeRabbit Integration

CodeRabbit provides AI code review on every PR. **Claude Code must wait for CodeRabbit approval before merging.**

### PR Workflow

1. Create PR with `gh pr create`
2. **Wait for CodeRabbit review** (7-30 min) — check with `gh pr checks <number>`
3. If CodeRabbit requests changes:
   - Read the CodeRabbit comments
   - Address each issue in follow-up commits
   - Push changes and wait for re-review
4. Only merge after CodeRabbit approves

### CodeRabbit CLI (Local Review)

Run local review before pushing to catch issues early:

```bash
# Review uncommitted changes (token-efficient output for AI)
coderabbit --prompt-only --type uncommitted

# Review committed changes on current branch
coderabbit --prompt-only --type committed
```

Always use `--prompt-only` — provides concise, token-efficient output optimized for Claude Code.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RackulaLives/Rackula](https://github.com/RackulaLives/Rackula) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
