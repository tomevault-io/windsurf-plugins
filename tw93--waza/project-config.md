---
trigger: always_on
description: This file is the canonical agent guide for the Waza repository. `CLAUDE.md` is a symlink to it, so Claude Code and Codex see identical content. Edit this file; do not edit `CLAUDE.md`.
---

# Waza Agent Guide

This file is the canonical agent guide for the Waza repository. `CLAUDE.md` is a symlink to it, so Claude Code and Codex see identical content. Edit this file; do not edit `CLAUDE.md`.

## Project

Waza is a skill collection for engineering workflows. The repository contains eight skills: `think`, `ui`, `check`, `hunt`, `write`, `learn`, `read`, and `health`.

## Repository Map

- `VERSION` - single source of truth for the lock-step version. Marketplace entries, README install URLs, and installer `WAZA_REF` defaults must agree with this file (codegen-enforced). Per-skill frontmatter carries no version field; the verifier rejects a stale `metadata.version`.
- `skills/RESOLVER.md` - trigger and routing table for the skill set.
- `skills/*/SKILL.md` - individual skill entrypoints.
- `skills/*/agents/` - specialist reviewer or inspector prompts.
- `skills/*/references/` - supporting references loaded only when needed.
- `skills/*/scripts/` - deterministic helper scripts.
- `rules/` - shared writing and behavior rules used by install and validation flows. `rules/durable-context.md` is the shared Durable Context Preflight preamble; codegen copies it into each referencing skill as `skills/<name>/references/durable-context.md` (direct installs get only the skill directory), and the six skills with optional memory context link to that skill-local copy.
- `.claude-plugin/marketplace.json` - **generated**. Edit `VERSION` or per-skill `SKILL.md` frontmatter and run `make regenerate`; never hand-edit.
- `.agents/plugins/marketplace.json` - **generated** Codex repo marketplace. Points Codex at `plugins/waza` for plugin installs; never hand-edit.
- `plugins/waza/` - **generated** Codex plugin tree. Mirrors `skills/` and `rules/` plus `plugins/waza/.codex-plugin/plugin.json`; edit source files and run `make regenerate`.
- `packaging.allowlist` - default-deny list of paths that ship in `waza.zip`. New shippable assets must be added here explicitly; everything else is excluded.
- `.github/workflows/` - public test and release automation. `release.yml` runs `make test` before `make package` so the tagged commit is gated by the same suite as PRs.
- `scripts/build_metadata.py` - codegen for Claude and Codex marketplace metadata, README install URLs, Codex plugin mirror files, skill-local shared assets (update checkers, durable-context copies), installer-script `WAZA_REF` defaults, and update-checker `LOCAL_VERSION`. Run via `make regenerate`; CI checks drift via `make verify-generated`.
- `scripts/verify_skills.py` - the only validator entrypoint; a driver over the check inventory in `scripts/skill_checks.py` (content, distribution, and routing checks). The facade's import list is the canonical inventory; do not re-enumerate it here.
- `scripts/package-skill.sh` + `scripts/packaging_filter.py` - build `dist/waza.zip` from `packaging.allowlist`.
- `scripts/setup-rule.sh` + `scripts/setup-statusline.sh` - public install helpers; `WAZA_REF` defaults are codegen-pinned to the current release tag.
- `Makefile` - smoke discovery and packaging entrypoints. Adding a `tests/test_<name>.sh` file is enough to create a `smoke-<name>` target automatically.
- `tests/test_*.sh` - one smoke per surface; sources `tests/test_helpers.sh` for tmpdir / repo-copy / stub-curl / instruction-file fixture factories. `tests/python/` holds the pytest unit layer (`make verify-unit`).

## Commands

```bash
make test             # verify-docs + verify-generated + verify-routing + verify-scripts + verify-unit + all smokes
make regenerate       # rewrite marketplace.json, README install URLs, update checker copies
make verify-generated # drift check used by CI; non-zero if regenerate would change anything
make package          # build dist/waza.zip from packaging.allowlist
```

Run `make test` before meaningful changes to skill behavior, packaging, scripts, marketplace metadata, or anything generated. If you edited only frontmatter or VERSION, also run `make regenerate` and commit the resulting `.claude-plugin/marketplace.json` / `README.md` / installer changes together with your source edits.

## Skill Design Rules

Before adding a capability, decide the layer deliberately:

| Question | Yes | No |
|---|---|---|
| Does the user need judgment, adaptation, or follow-up questions? | Skill | Script or rule |
| Does the same input always produce the same output? | Script or rule | Skill |
| Is it a lookup, list, status check, or invariant check? | Script or rule | Skill |
| Does behavior shift with conversation context? | Skill | Script or rule |

Examples: `verify_skills.py` is a script; `rules/english.md` and `rules/chinese.md` are rules; `/think`, `/hunt`, `/check`, and `/health` are skills.

- Put adaptive, judgment-heavy workflows in skills.
- Put deterministic checks, lookups, and table-driven validation in scripts.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tw93/Waza](https://github.com/tw93/Waza) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
