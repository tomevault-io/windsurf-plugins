---
trigger: always_on
description: This repository publishes Docker-authored knowledge skills for AI coding agents.
---

# Repository guidance

## Purpose and sources of truth

This repository publishes Docker-authored knowledge skills for AI coding agents.
Canonical skill content lives under `skills/`; discovery symlinks and plugin
manifests expose it to supported agents. Start with [README.md](README.md) for the
catalog and repository entry points; installation guidance lives on
[Docker Docs](https://docs.docker.com/ai/skills/install/), and the repository layout is
visible in the top-level tree. Follow [CONTRIBUTING.md](CONTRIBUTING.md) for
the contribution flow and DCO text. Do not duplicate those documents here.

`catalog.yaml` is the source of truth for the distribution version, canonical
manifest description, documented distribution surfaces and their manifest
mapping, products, skill IDs, per-skill versions, and status. Any
change under `skills/<id>/` must increase that skill's version in both the
catalog and its `skill.yaml`: patch for corrections, minor for new guidance,
assets, or status, and major for a material routing-contract change. Versions
never decrease, and new skills need a valid matching initial version.
`scripts/render_catalog.py` generates the marked tables and inventories in
`README.md`, `evals/README.md`, and `skills.sh.json`, plus the
versions and descriptions in plugin manifests without reformatting them. Edit
the catalog and run the renderer; never hand-edit generated output. The
top-level distribution version changes only in a release PR containing no skill
changes and only the catalog, `CHANGELOG.md`, and rendered outputs. Prepare those
files with `task release:prepare VERSION=X.Y.Z`; do not bump or rotate them by
hand. User-visible changes update the changelog's `Unreleased` section in the
same change. Maintainer release steps and the SemVer policy are in
[CONTRIBUTING.md#maintainer-releases](CONTRIBUTING.md#maintainer-releases).

## Commands

Run commands from the repository root. [Task](https://taskfile.dev/) and Docker
are required; validation runs in pinned container images.

- `task` or `task ci`: run the complete skill-validation, evaluation, link, and
  verification-script suite used by CI and release workflows.
- `task validate`: run unit tests for repository validators, then validate skill
  structure, frontmatter, catalog membership, manifests, ownership, and files.
- `task eval`: run deterministic checks against checked-in skill assets. This is
  not a live model evaluation.
- `task links`: check local Markdown destinations and heading anchors.
- `task links:external`: opt-in live HTTPS link check; runs separately from
  offline `task`/`task ci` in a pinned container.
- `task images:inventory`: count actionable skill and eval container image
  references offline; this inventory also runs in `task ci`.
- `task images:check`: opt-in live public image tag and linux/amd64 plus
  linux/arm64 check; it does not run in offline `task ci`.
- `task catalog`: regenerate all files derived from `catalog.yaml`.
- `task catalog:check`: verify generated catalog files are current without
  changing them.
- `task release:prepare VERSION=X.Y.Z`: validate a strictly increasing
  distribution version, rotate `CHANGELOG.md`, and regenerate catalog-derived
  files for a release pull request.
- `VERSION_CHECK_BASE_SHA=$(git merge-base HEAD origin/main) task`: run the
  version-policy and DCO commit comparison locally against the pull request
  base. Without a base SHA both PR-only checks are skipped so offline validation
  still works; CI always supplies the pull request base and full Git history.

Prefer the narrow command while editing, then run `task` before declaring the
change complete. `scripts/ci.sh` is the shared CI/release entrypoint; keep it and
`Taskfile.yml` aligned when validation changes.

## Validation invariants

Keep these repository-wide contracts intact:

- Every non-hidden directory under `skills/` has exactly one `catalog.yaml`
  entry, and every catalog path exists.
- Each skill contains `SKILL.md`, `skill.yaml`, and `agents/openai.yaml`; IDs and
  versions agree with the catalog and required metadata is present.
- Every catalogued skill has `evals/<skill-id>.md` and specific skill plus eval
  ownership rules in `.github/CODEOWNERS`.
- Every skill includes the required sections enforced by `scripts/validate.py`,
  uses only supported top-level frontmatter fields, and keeps `SKILL.md` at or
  below 500 lines.
- Skill content passes deterministic encoding, control-character, hidden-text,
  secret, unsafe-command, insecure-URL, symlink, file-mode, and file-size checks.
- Discovery symlinks resolve to `skills/`, and `CLAUDE.md` resolves to
  `AGENTS.md`.
- Files under `references/`, `assets/`, `checks/`, and `scripts/` are referenced
  from that skill's `SKILL.md`; referenced files exist.
- Compose YAML assets under `skills/*/assets/` parse cleanly and avoid literal
  credentials or URL passwords, unscoped datastore ports, untagged or `latest`
  images, and Docker socket mounts.
- Plugin manifest versions match the catalog distribution version and continue
  to represent the catalog.
- Every actionable container image reference in skill and eval examples is
  included in the offline image inventory; live checking runs separately.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [docker/skills](https://github.com/docker/skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
