---
trigger: always_on
description: <!-- standards-version: 1.10.0 -->
---

<!-- standards-version: 1.10.0 -->

# AGENTS.md

Guidance for AI coding agents working on the Blender Developer Tools repository.

**Division of labor:** this file carries fleet-standard governance and workflow
rules — branching, commits, merge and CI-evidence policy, release automation,
authoring standards. `CLAUDE.md` carries repo-specific operational facts an
agent needs at runtime — content inventory, Blender runtime discovery, git
staging hazards, and the example-shipping quality gates. Read both; neither
repeats the other.

## Repository overview

Skills, rules, snippets, starter templates, and runnable smoke-gated examples
for Blender Python development. The repo targets **Blender 5.1** (current
stable) with a **Blender 4.5 LTS** fallback. There is no MCP server. It ships
a `.cursor-plugin/plugin.json` manifest so the ecosystem drift checker
classifies it as a `cursor-plugin`. This is content the AI loads when the user
asks Blender questions or works on Blender add-ons in Cursor or Claude Code.

The content base is 12 skills, 6 rules, 2 templates, 17 snippets, and 44
examples (counts are CI-enforced against README.md and the manifest). The full
inventory tables and per-item purposes live in `CLAUDE.md`. Example anatomy
and authoring rules: copy `examples/bmesh-gear/`; the render look is specified
in `docs/VISUAL-STYLE.md`; the canonical run prompt is
`docs/new-example-prompt.md`.

## Repository structure

```
Blender-Developer-Tools/
  skills/<skill-name>/SKILL.md   # 12 skill files
  rules/<rule-name>.mdc          # 6 rule files
  templates/<template-name>/     # 2 starter templates
  snippets/<snippet-name>.py     # 17 standalone Python snippets
  examples/<name>/               # 40 runnable smoke-gated examples (+ gallery.json)
  examples/gallery_framing.py    # shared Layer 1 framing measurement (render path only)
  scripts/build_gallery.py       # generates docs/gallery/ (stdlib only)
  scripts/site/                  # vendored landing-page build (build_site.py + template)
  docs/gallery/                  # committed generated gallery pages + hero assets
  docs/new-example-prompt.md     # canonical example-creation prompt
  .github/workflows/             # validate, blender-smoke, drift-check, release, pages, label-sync, stale
  .github/dependabot.yml
  AGENTS.md, CLAUDE.md, README.md, ROADMAP.md, CHANGELOG.md
  CONTRIBUTING.md, SECURITY.md, CODE_OF_CONDUCT.md
  VERSION                        # source of truth for the repo version
  LICENSE                        # CC-BY-NC-ND-4.0
```

## Branching and commit model

- Single `main` branch. No develop or release branches. Work on a focused
  feature branch off an up-to-date `main`. Never force-push or bypass hooks.
- Conventional commits drive the auto-release workflow. It scans the commit subjects since
  the last tag and releases only when at least one is release-worthy:
  - `feat:` triggers a minor bump
  - `fix:` triggers a patch bump
  - `feat!:` / `fix!:` / `BREAKING CHANGE` triggers a major bump
  - Other types (`chore:`, `docs:`, `ci:`, `refactor:`, etc.) do **not** cut a release: the
    workflow runs, decides there is nothing to release, and exits without a tag or version
    bump. A mixed push still releases if any commit in range is a `feat:`/`fix:`.
  - `[skip ci]` in the head commit still bypasses the workflow entirely. With the commit-type
    gate above it is now an optional override, not a requirement for non-release commits.
- Commit messages should describe the why, not the what, and carry a DCO
  `Signed-off-by:` trailer matching the commit author (see CONTRIBUTING.md).
- Stage with explicit paths only — never `git add -A` or `git add .`. The
  reason is a repo-specific hazard documented in `CLAUDE.md` § Git staging.

## Merge policy and CI evidence

- **Squash-merge with branch deletion is the standard.** The PR becomes one
  commit on `main`; delete the remote feature branch after merge, then
  fast-forward local `main`.
- **Smoke jobs do not re-run on the merge SHA.** `blender-smoke.yml` triggers
  on `pull_request` (plus a weekly schedule and manual dispatch) — there is no
  `push` trigger. The correct post-merge evidence for example changes is:
  both Blender smoke jobs (4.5 LTS and 5.1) passed on the PR head SHA that
  became the sole squash-merged commit, with the actual binary versions
  confirmed in the job logs.
- **Post-merge, verify green on `main`:** Release (`release.yml`), Validate
  (`validate.yml`), Ecosystem drift check (`drift-check.yml`), and Deploy
  GitHub Pages (`pages.yml`; paths-filtered, so it does not trigger for every
  change).
- **Socket Security checks** ("Socket Security: Project Report" and "Socket
  Security: Pull Request Alerts") run on PRs via the Socket GitHub App. No
  override policy has been decided: a pending or failing Socket check is
  unresolved — wait before merging.
- **Release-owned fields are never hand-edited:** `VERSION`, `CHANGELOG.md`,
  the CLAUDE.md `**Version:**` line, the ROADMAP.md `**Current:**` line, and
  the manifest `"version"` in `.cursor-plugin/plugin.json`. Generated gallery
  pages under `docs/gallery/` are regenerated via `scripts/build_gallery.py`,
  never hand-edited.
- **Evidence over assertion:** PR bodies must label what was proven by live

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TMHSDigital/Blender-Developer-Tools](https://github.com/TMHSDigital/Blender-Developer-Tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->
