---
trigger: always_on
description: This file orients an AI agent (or a human) making changes to the
---

# Working on this repo

This file orients an AI agent (or a human) making changes to the
**SurveyCTO Agent Skill** itself. It is not part of the skill bundle —
it is excluded from `surveycto-skill.zip` / `surveycto-skill-dev.zip`.

For end-user-facing documentation see [`README.md`](README.md). For the
skill content itself see [`SKILL.md`](SKILL.md).

## What this repo is

An [Agent Skills](https://agentskills.io) package that gives AI agents
SurveyCTO domain expertise. The deliverable is a zip
(`surveycto-skill.zip`) that hosts (Claude Cowork, Codex, etc.) extract
into their skills directory. **The zip is not committed** — it's built
and published by GitHub Actions:

- Push to `develop` → `.github/workflows/build-dev.yml` builds
  `surveycto-skill-dev.zip` as a workflow artifact (downloadable from
  the run page) for smoke-testing pre-release changes.
- Push to `main` → `.github/workflows/release.yml` builds
  `surveycto-skill.zip` and attaches it to a new GitHub Release whose
  tag/name comes from `metadata.version` in `SKILL.md`.

For local smoke-testing, you can build a zip yourself (see
[*When to rebuild the local zip*](#when-to-rebuild-the-local-zip)). It
stays in your working tree and is gitignored.

## Branching

- **`develop`** — active development. Pushes here trigger the dev-build
  workflow.
- **`main`** — stable releases. Pushes here trigger the release
  workflow.

Default working branch for new work is `develop` (or a feature branch
off `develop`). PRs target `develop` first; `develop → main` is reserved
for release.

## When to bump the version

The release workflow takes its tag from `metadata.version` in
`SKILL.md`. If `develop` is merged into `main` without a bump since the
last release, the workflow will try to re-tag the same version and the
release will fail (or, worse, silently overwrite, depending on settings).

**Procedure** before making non-trivial changes to skill content
(`SKILL.md`, `references/`, `assets/`):

1. Compare the current branch's `metadata.version` in `SKILL.md`
   against `main`'s:
   ```bash
   git show origin/main:SKILL.md | sed -n '/^metadata:/,/^---/p' | grep version
   ```
2. If the two match, the current branch hasn't been bumped since the
   last release — bump it as part of this change. If they already
   differ (current is ahead of `main`), no further bump is needed
   unless the new change crosses a different semver boundary than the
   bump that's already there.
3. If you're unsure (e.g. the change is small, or the user hasn't
   asked), ask or offer rather than skipping. A missed bump blocks
   release; an unnecessary bump is harmless.
4. When bumping, update **both** places the version appears in
   `SKILL.md` (see below). They must stay in sync — the frontmatter
   drives the release tag and the body line is what the agent reports
   at runtime, since agents don't see frontmatter.

Use [semantic versioning](https://semver.org), per the rules in
[`README.md` → Versioning](README.md#versioning):

- **Pre-release** (`1.0.0-beta`, `1.0.0-beta.N`, `1.0.0-rc.N`) — public
  beta and release-candidate builds. Increment the trailing
  `-beta.N` / `-rc.N` for each pre-release ship.
- **Patch** (`1.0.0` → `1.0.1`) — fix incorrect information, typos,
  clarify existing guidance.
- **Minor** (`1.0.0` → `1.1.0`) — add new content (new reference
  sections, new patterns, template updates, new behavior rules in the
  skill).
- **Major** (`1.x` → `2.0.0`) — structural changes that may affect how
  agents use the skill (file moves, removed sections, MCP-contract
  changes the skill depends on).

The version lives in **two** places inside `SKILL.md` that must be
kept in sync (see [`README.md` → Versioning](README.md#versioning) for
the rationale):

1. The `metadata.version` field in the frontmatter — read by the
   release workflow and by skill hosts.
2. A "Skill version" line near the top of `SKILL.md`'s body — visible
   to the agent at runtime, since agents do not see frontmatter.

```yaml
metadata:
  author: Dobility, Inc. (SurveyCTO)
  version: "X.Y.Z"
```

```markdown
**Skill version: X.Y.Z.** ...
```

## When to update the version policy

Skills do not auto-update and there is no separate update channel, so
the SurveyCTO MCP server announces the current skill-version policy to
connected agents (it serves the recommended and deprecated versions, and
agents prompt users to update when they're behind). The editorial half
of that policy lives in **`version-policy.source.json`**. Bumping
`metadata.version` in `SKILL.md` alone does **not** update what the
server broadcasts — that file is separate and must be maintained
deliberately. See [`README.md` → Broadcasting versions through the MCP
server](README.md#broadcasting-versions-through-the-mcp-server) for the
end-user-facing rationale.

Edit `version-policy.source.json` in the **same PR that bumps the
version** whenever the deprecation floor or the update summary should
change. Its fields:

- **`latest_updates`** — a newest-first, human-readable list of
  per-version summaries shown to agents. When you bump the version for a
  content change worth surfacing to users, prepend a new
  `"X.Y.Z: <one-line summary>"` entry. This is the field you'll touch
  most often.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [surveycto/surveycto-agent-skill](https://github.com/surveycto/surveycto-agent-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
