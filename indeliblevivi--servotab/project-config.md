---
trigger: always_on
description: This file is the stable project-specific contract for agents working in this repository. Read it before editing canonical methods, generated plugin files, package metadata, behavior evidence, the website, or release surfaces.
---

# Servotab Repository Contract

This file is the stable project-specific contract for agents working in this repository. Read it before editing canonical methods, generated plugin files, package metadata, behavior evidence, the website, or release surfaces.

## Product identity

- Product and plugin id: `servotab`
- Canonical domain: `https://servotab.com`
- Product form: a quiet, risk-scaled Codex engineering method layer
- Core thesis: “Method as exponent, not machinery.”
- Stable promise: keep clear changes direct, add stronger method only when risk or uncertainty requires it, close claims with fresh evidence, and preserve the complete requested outcome.

Servotab is independent and community-maintained. Do not claim it is an official OpenAI product, approved by OpenAI, submitted to the plugin directory, or available there unless current owner-authorized evidence proves that exact state.

## Truth ownership

| Surface | Authority |
|---|---|
| `methods/*.md` | Canonical bodies for the 12 engineering methods |
| `scripts/skill_catalog.py` | Canonical skill ids, descriptions, prompts, and activation metadata |
| `scripts/build_skills.py` | Canonical generation rules for plugin skills and curated package assets |
| `plugins/servotab/skills/**` | Generated projection; never edit directly |
| `assets/` | Canonical repository identity assets and usage notes; `servotab-mark-ink*` plus `skill-icons/*` own the 13 skill icon sources |
| `plugins/servotab/assets/` | Generated curated package copies, not a second asset authority |
| `plugins/servotab/skills/*/assets/` | Generated local skill-icon copies; never edit directly |
| `plugins/servotab/plugin.json` | Canonical portable plugin package metadata and public interface contract |
| `plugins/servotab/.codex-plugin/plugin.json` | Synchronized compatibility fallback for older Codex package readers; never an independent authority |
| `plugins/servotab/LICENSE` and `NOTICE.md` | Package-local functional-material and identity-asset rights boundary |
| `.agents/plugins/marketplace.json` | Repository marketplace route |
| `PACK_MANIFEST.json` | Exact generated package identity; regenerate, do not hand-edit |
| `fieldlab-pack.json` and `evals/` | Servotab-owned behavior subject and evidence surfaces |
| `site/` | Static public website source; not plugin runtime authority; the Methods catalog imports canonical leaf SVGs from root `assets/skill-icons/*` |
| `README.md` | Canonical English user-facing identity, installation, use, and limitations |
| `README.zh-CN.md` | Full Chinese reader edition; factual parity with `README.md`, not sentence-level identity |
| `docs/current-state.md` | Volatile candidate, install, GitHub, deployment, domain, and submission state |
| `docs/migration-from-softpowers.md` | Supported transition from manifest-owned legacy global layers |

Logs, screenshots, design boards, reviews, external repositories, generated files, deployments, and previous plans are evidence or projections unless the current task explicitly gives them authority. They do not silently override current user intent, accepted specifications, or canonical source.

## Current package topology

The package contains exactly one implicit-eligible router, `servotab`, and 12 explicit-only leaves:

```text
design
spec-chain
plan
execute
debug
tdd
review
review-feedback
verify
worktree
delegate
finish
```

Retired current IDs are `brainstorm`, `receive-review`, and `parallel`. Preserve them only in historical release records, provenance, and migration explanation. Do not restore them as active methods, skills, references, invocation examples, issue fields, or package paths.

Root `skills/`, `install.sh`, `uninstall.sh`, `scripts/install.py`, and `scripts/uninstall.py` are retired. Do not add a parallel global-skill installer around the plugin package.

`scripts/build_skills.py` fails closed if the retired root `skills/` path exists. The generator must never recursively delete that path: remove known tracked legacy projection files through the reviewed migration diff, and preserve any unknown or untracked content for explicit disposition.

## Editing workflow

For a method or metadata change:

1. Edit `methods/*.md` and/or `scripts/skill_catalog.py`.
2. Run `python3 scripts/build_skills.py`.
3. Inspect canonical and generated diffs together.
4. Run the package validation gate.
5. Update documentation whose current contract changed.

Workspace lifecycle guidance is owned by `methods/worktree.md`; `finish` shares its removal contract and `delegate` must not equate branch names with isolated write surfaces. Changes to these decisions require matching activation seeds and semantic canaries; disposable Git tests establish mechanisms only, not model behavior.

For a canonical package asset change:

1. Edit or replace only the intended file under `assets/` with rights and provenance understood.
2. Run `python3 scripts/build_skills.py` to update curated plugin copies, including each skill's local SVG/PNG icon pair.
3. Regenerate `PACK_MANIFEST.json` only after plugin validation passes.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [IndelibleVivi/servotab](https://github.com/IndelibleVivi/servotab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
