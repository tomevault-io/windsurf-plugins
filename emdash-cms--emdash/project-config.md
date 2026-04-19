---
trigger: always_on
description: This file provides guidance to agentic coding tools when working with code in this repository.
---

This file provides guidance to agentic coding tools when working with code in this repository.

## Project Status

**Beta, post pre-release.** EmDash is published to npm and in active use, with i18n, RTL, and the plugin system shipped. We're no longer in the scorched-earth pre-release phase -- real users depend on current behavior, so backwards compatibility now matters (see Rules below). All development happens inside this monorepo using `workspace:*` links. See [CONTRIBUTING.md](CONTRIBUTING.md) for the human-readable contributor guide (setup, repo layout, "build your own site" workflow).

## Repository Structure

This is a monorepo using pnpm workspaces.

`CLAUDE.md` is a symlink to `AGENTS.md`. `.opencode/skills` and `.claude/skills` are symlinks to `skills/`. Don't try to sync between them.

- **Root**: Workspace configuration and shared tooling
- **packages/core**: Main `emdash` package - Astro integration and core APIs
- **demos/**: Demo applications and examples (`demos/simple/` is the primary dev target)
- **templates/**: Starter templates (blog, marketing, portfolio, starter, blank) -- contributors copy these into `demos/` to build their own sites
- **docs/**: Public documentation site (Starlight)

# Rules

**Backwards compatibility matters now.** We're out of pre-release, but pre-1.0. Real installs depend on current behavior, schemas, and API shapes. Breaking changes are allowed in minors, but need an explicit decision, a bump on the affected package, and a changeset that calls the break out clearly. Prefer additive changes: new fields, new routes, new options with sensible defaults. If an old API is obsolete, mark the replacement as preferred and keep the old path working unless there's a reason it can't. Database migrations are forward-only -- never write one that leaves existing content inaccessible. When in doubt, open a Discussion before coding.

**TDD for bugs.** Write a failing test -> fix the bug -> verify the test passes. A bug without a reproducing test is not fixed.

**Localize everything user-facing.** All admin UI strings, aria labels, and toast messages go through Lingui. All admin layout uses RTL-safe logical Tailwind classes. See the Localization and RTL sections below.

## Contribution Rules (for AI agents and human contributors)

Read [CONTRIBUTING.md](CONTRIBUTING.md) before opening a PR. Key rules:

- **You MUST use the PR template.** Every PR must include the PR template with all sections filled out. The template is loaded automatically when you create a PR via the GitHub UI. If you create a PR via the API or CLI, copy the template from `.github/PULL_REQUEST_TEMPLATE.md` into the PR body. **PRs that do not use the template will be closed automatically by CI.**
- **Features require a prior approved Discussion.** Do not open a feature PR without one. It will be closed. Open a [Discussion](https://github.com/emdash-cms/emdash/discussions/categories/ideas) in the Ideas category first.
- **Bug fixes and docs** can be PRed directly.
- **Check every applicable checkbox** in the PR template, including the "I have read CONTRIBUTING.md" box and the AI disclosure box if any part of the code was AI-generated.
- **Do not make bulk/spray changes** (e.g., "fix all lint warnings", "add types everywhere", "improve error handling across codebase"). If you see a systemic issue, open a Discussion.
- **Do not touch code outside the scope of your change.** No drive-by refactors, no "while I'm here" improvements, no added comments or logging in unrelated files.
- **All CI checks must pass.** Typecheck, lint, format, and tests. No exceptions.

## Workflow

### Before Starting

1. Run `pnpm --silent lint:json | jq '.diagnostics | length'` and fix any issues. Non-negotiable.

### During Work

- Run `pnpm --silent lint:quick` after every edit -- takes less than a second. Returns JSON with stderr redirected to /dev/null, so it won't break parsers. Fix any issues immediately.
- Run `pnpm typecheck` (packages) or `pnpm typecheck:demos` (Astro demos) after each round of edits.
- Format regularly. pnpm format in the root uses oxfmt with tabs for indentation and is very fast. Don't let formatting pile up.
- Commit regularly, and always format and quick lint beforehand.
- Update tasks.md when completing tasks. Write a journal entry when starting or finishing significant work, or if you learn anything interesting or useful that you'd like to remember.

### Before Committing

You verified linting and types were clean before starting. If they're failing now, your changes caused it -- even if the errors are in files you didn't touch. Don't dismiss failures as "unrelated". Don't assign blame. Just fix them.

### Changesets

If your change affects a published package's behavior, add a changeset. Without one, the change won't trigger a package release.

```bash
pnpm changeset --empty
```

This creates a blank changeset file in `.changeset/`. Edit it to add the affected package(s), bump type, and description:

```markdown
---
"emdash": patch
---

Fixes CLI `--json` flag so JSON output is clean.
```

Start descriptions with a present-tense verb (Adds, Fixes, Updates, Removes, Refactors). Focus on what changes for the user, not implementation details.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [emdash-cms/emdash](https://github.com/emdash-cms/emdash) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
