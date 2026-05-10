---
trigger: always_on
description: > Scope: This file defines repository-level operating rules for agents working in this repo.
---

# Spec Kit Extensions Agent Guide

> Scope: This file defines repository-level operating rules for agents working in this repo.
> Keep architecture, product behavior, and domain design rules out of `AGENTS.md`.
> If `.specify/memory/constitution.md` exists, it owns project-specific architecture rules.

## Repository Shape

- This repository hosts multiple Spec Kit extensions. Each extension lives in a top-level directory such as `superpowers-bridge/` or `memorylint/`.
- Treat the extension directory name as the release/package identifier. Release workflow input `extension_id` and git tags use the directory name, for example `superpowers-bridge-v1.1.0`.
- Treat `extension.id` in `extension.yml` as the stable command namespace identifier. It does not need to equal the directory name, but once published it must remain stable.

## Extension Authoring Rules

- Every extension must keep `extension.yml` valid for `schema_version: "1.0"`.
- Keep `extension.description` at or below 200 characters.
- Command names must follow `speckit.{extension-id}.{command-name}`.
- Every command markdown file under `commands/` must include the `$ARGUMENTS` context block.
- If a command depends on scripts, declare them in Markdown frontmatter under `scripts:` instead of relying on undocumented side effects.
- Hook `command:` values must match an entry declared under `provides.commands` exactly.
- Prefer extension-local config templates such as `*-config.template.yml` and declare them under `provides.config` instead of hardcoding machine-local paths.
- Extensions in this repo must be additive. Do not directly edit `.specify/scripts/` or `.specify/templates/` from an extension implementation.
- README install examples, release download URLs, and catalog-facing links must point to real published tags only.

## Validation Commands

Run the relevant checks before claiming extension work is complete:

- `bash tests/test-review-regressions.sh`
- `bash tests/test-release-workflow.sh`
- `bash superpowers-bridge/tests/test-status-sync.sh`
- `pwsh -File superpowers-bridge/tests/test-status-sync.ps1`
- `ruby -e "require 'yaml'; YAML.load_file('.github/workflows/ci.yml'); YAML.load_file('.github/workflows/release-trigger.yml')"`
- `git diff --check`

If a change only touches one extension, still run the workflow or regression checks that guard the edited files.

## Git and Review Hygiene

- Use focused commits. Do not mix release metadata edits with unrelated refactors.
- Do not stage unrelated workspace changes when preparing a release.
- Use Conventional Commit style when practical, for example `feat(...)`, `fix(...)`, `docs(...)`, `chore(...)`.
- When review feedback questions versioning or changelog state, compare against the latest published tag before changing `extension.version`.

## Release Workflow Rules

- Releases are created through `.github/workflows/release-trigger.yml` only.
- The workflow is manually dispatched with:
  - `extension_id`: the top-level extension directory name
  - `version`: the target SemVer to publish
- The release workflow is responsible for:
  - updating `extension.version`
  - rotating `CHANGELOG.md` `## [Unreleased]` into `## [<version>] - YYYY-MM-DD`
  - refreshing README release download links when present
  - creating the extension zip
  - running `gh release create`
- Do not manually create future git tags or pre-create release sections that the workflow is supposed to generate.

## Changelog and Versioning Policy

### Source of Truth

- `extension.version` on `main` must represent the latest published version for that extension, not the next intended version.
- A version is considered published only after the git tag and GitHub Release both exist.
- All not-yet-published work belongs under `## [Unreleased]` in `CHANGELOG.md`.
- The root `README.md` version table and extension README release URLs must also reflect published versions only.

### Unreleased Work

- Keep `## [Unreleased]` present at the top of every extension changelog that participates in the release workflow.
- `## [Unreleased]` must be non-empty before triggering a release.
- Do not add speculative headings such as `## [2.0.0]` before that version is actually being published.
- If a change is breaking but unreleased, describe the breaking change under `## [Unreleased]`. Do not bump `extension.version` early just because the next release will likely be major.

### Placeholder Rules for Planned Versions

- Unpublished versions are planning metadata, not release state.
- Never use a future placeholder version in `extension.yml`, README release links, root catalog tables, or git tags.
- If you need to record the intended next bump during development, do it inside `## [Unreleased]` only.
- Preferred placeholder format:

```md
## [Unreleased]

<!-- planned-bump: major -->
<!-- next-release-version: 2.0.0 -->

### Changed

- Removed a previously supported hook and replaced it with a new bridge-native flow.
```

- `planned-bump` and `next-release-version` are advisory placeholders only. They help reviewers understand intent, but they do not mean the version has been published.
- If the intended number changes before release, update or remove the placeholder comments instead of editing `extension.version`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RbBtSn0w/spec-kit-extensions](https://github.com/RbBtSn0w/spec-kit-extensions) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
