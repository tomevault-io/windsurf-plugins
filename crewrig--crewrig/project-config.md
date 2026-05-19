---
trigger: always_on
description: This document defines the rules and conventions that all agents (human or AI) must follow when contributing to this project.
---

# CrewRig — Agent Working Rules

This document defines the rules and conventions that all agents (human or AI) must follow when contributing to this project.

## Language

All **project content must be written in English**. "Project content" covers
every artifact that lands in the repository or on GitHub — there are no
exceptions for "internal" notes, draft documents, or AI-authored prose.

This includes, but is not limited to:

- **File content in the repository** — source code, inline comments,
  documentation prose, READMEs, ADRs, RFCs, configuration files, shell
  scripts, and every framework artifact (`SKILL.md`, `AGENT.md`,
  `AGENTS.md`, `CLAUDE.md`, etc.).
- **GitHub artifacts** — commit messages, PR titles, PR bodies, PR review
  comments, issue titles, issue bodies, and every comment posted on an
  issue or PR (including incremental logbook updates).

**Decision rule:** *Is this landing in the project or on GitHub?* → English
only.

**Exception:** Interpersonal interactions between the user and the agent
(chat sessions, transient terminal output) MUST be conducted in the **User
Preferred Language**. This exception covers only ephemeral dialogue — the
moment content is committed, pushed, or posted to GitHub, the English-only
rule takes over.

## Branching Strategy

- The primary branch is `main`, linked to the `origin` remote (GitHub).
- The `main` branch is **protected**: no direct pushes allowed.
- Every change must go through a **feature branch** merged into `main` via a Pull Request.
- **NEVER merge a Pull Request (PR/MR)** without asking for the user's formal permission JUST BEFORE executing the merge.
- The `import/gitlab` branch tracks the legacy GitLab project (`gitlab` remote) and serves as inspiration only.

## Naming Convention

The [Gitmoji](https://gitmoji.dev/) convention applies to **all named project artifacts** — not only git commit messages:

- **Git commits** — `<emoji> <Short description>`
- **Issue titles** — `<emoji> <Short description>`
- **Pull request titles** — `<emoji> <Short description>`

Never use conventional-commit prefixes (`feat:`, `fix:`, `chore:`, etc.) in any of the above. Gitmoji is the sole convention.

Examples:

- `🎉 Initial commit`
- `✨ Add user authentication module`
- `🐛 Fix null pointer in config loader`
- `📝 Update README with setup instructions`
- `♻️ Refactor settings parser for clarity`

Refer to [gitmoji.dev](https://gitmoji.dev/) for the full list of valid emojis and their meanings.

## Version Bump Convention

Skill and agent sources carry a `metadata.provenance.version` field that
tracks shipped revisions. One rule and one exemption govern when it must change.

**Rule — bump on modification of shipped sources.** Any diff that modifies
a skill or agent source already present on `main` MUST bump
`metadata.provenance.version` in the same diff. Affected paths:

- `community-config/skills/*/SKILL.md`
- `community-config/agents/*/AGENT.md`

**Exemption — new components do not bump in-branch.** Components
introduced on a feature branch start at `1.0.0` and stay there until the
branch is merged. In-branch fixes to a brand-new component MUST NOT bump
its version — the version is only meaningful once the component ships on
`main`. CI enforces this: only files with `git diff --name-status` status
`M` (modified) trigger the check; newly added files (`A`) are skipped.

**SemVer guidance for bumps:**

- `PATCH` (1.0.x) — friction fix, wording change
- `MINOR` (1.x.0) — additive change (new section, new field)
- `MAJOR` (x.0.0) — breaking contract change

**Enforcement.** `scripts/check-skill-versions.sh` runs in CI, diffs the
PR against its target branch, and fails the build when a modified source
ships without a version bump.

## CLI Matrix Maintenance

`docs/cli-matrix.md` is the source of truth for every CLI-specific
integration point. It MUST stay in lockstep with the code.

**Trigger surface.** A change is CLI-specific when it touches any of:
`.claude/**`, `.gemini/**`, `community-config/**`, `extensions/**`,
`hooks/*-transcript-hooks.json`, `config/claude/**`, `config/gemini/**`,
`scripts/build-components.sh`, any `scripts/{build,install,setup,import,manage}-*.sh`,
`.github/workflows/claude.yml` or `.github/workflows/gemini.yml`,
the top-level entry-point files (`CLAUDE.md`, `GEMINI.md`), or a
CLI-prefixed entry in `Taskfile.yml` / `.gitignore`.

**Obligation.** Any PR that modifies the trigger surface MUST consult
`docs/cli-matrix.md` and update it in the same diff — new row, edited
cell, or refreshed `Parity gaps` entry. Drift is a parity bug.

**Parity check.** When adding or modifying a feature for one CLI,
verify the other. If the symmetric path is missing, either implement
it in the same PR or record it under `Parity gaps` with a follow-up
issue link. Silent asymmetry is prohibited.

## Agent Team Protocol

Project tickets are multi-step work. They must be treated by a **team of specialist agents**, not by a single agent working solo inline.

### When this applies

Any time the agent is asked to treat a project ticket (a GitHub issue, a PR
task, a feature request, or any equivalent unit of tracked work), the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [crewrig/crewrig](https://github.com/crewrig/crewrig) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
