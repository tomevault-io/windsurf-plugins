---
trigger: always_on
description: Context for AI assistants working on this repo. Keep this file factual; user-facing docs live in `README.md`.
---

# CLAUDE.md — spec-kit-discovery

Context for AI assistants working on this repo. Keep this file factual; user-facing docs live in `README.md`.

## What this is

A [spec-kit](https://github.com/github/spec-kit) extension that adds a discovery phase **before** `/specify`. Five commands (`problem` → `concept` → `clarify` → `decide` → `decompose`) produce durable artifacts on isolated `discovery/NNN-<slug>` branches in the **consumer** project. This repo *is* the extension; it is not itself a spec-kit project.

## Locked design decisions (v0.1)

Don't re-litigate these without an explicit user ask:

1. **Scope = lean + ADR.** Five commands, not three, not six. `context` and `options` are folded into `concept.md` as sections, not standalone commands.
2. **Slug strategy = AI-generated, user-confirmed, numbered branch.** Branch name `discovery/NNN-<slug>`, dir `.specify/discovery/NNN-<slug>/`. `NNN` = max(local branches, remote branches, existing dirs) + 1, zero-padded.
3. **Interaction style = draft + clarify.** Each command drafts from prose (`$ARGUMENTS`) and marks gaps with `[NEEDS CLARIFICATION: <question>]` (spec-kit's existing convention). Empty args → pure template. `clarify` resolves markers interactively.
4. **Constitution integration = opt-in promotion.** `decide` asks once: "*Does this decision establish a project-wide rule?*" Yes → append 1-3 imperative bullets to `.specify/memory/constitution.md` under `## Decisions promoted from ADRs`, each with a back-link. Never append the full ADR.
5. **Handoff = discovery branch merges to main; `/specify` runs from main per slice.** Discovery branches are conceptually a design-doc stage, not feature branches.

## Repo layout

```
extension.yml                       # manifest — 5 commands, no hooks in v0.1
commands/
  problem.md                        # branch+dir creation, 01-problem.md
  concept.md                        # 02-concept.md (options + recommendation)
  clarify.md                        # walks [NEEDS CLARIFICATION] markers
  decide.md                         # 03-adr-MMM-<slug>.md + opt-in promotion
  decompose.md                      # 04-features.md with /specify-ready prose; opt-in 05-management-summary.md, opt-in promotion of ADRs → docs/adr/ and summary → docs/management-summaries/
scripts/bash/
  discovery-context.sh              # state introspection — single source of truth
README.md                           # user-facing
CHANGELOG.md
LICENSE                             # MIT
.extensionignore                    # excluded from `specify extension add` installs
```

Each `commands/*.md` file is an AI prompt with YAML frontmatter (`description`) and a markdown body containing natural-language steps, a call to the shared script, and a template for the artifact it produces.

## Shared state script (versioned contract)

`scripts/bash/discovery-context.sh` is the **single source of truth** for repo state. All commands call it via the installed path `.specify/extensions/discovery/scripts/bash/discovery-context.sh` (spec-kit copies `scripts/` to `.specify/extensions/<ext-id>/scripts/` at install time — see `extensions/git` in spec-kit core for the reference pattern).

**Output protocol** (treat as a versioned contract — any rename is a breaking change for all 5 commands):

Header block, always emitted:

```
CURRENT_BRANCH=<name>
ON_DISCOVERY_BRANCH=yes|no
WORKING_TREE_DIRTY=yes|no
IN_GIT_REPO=yes|no
DISCOVERY_SUFFIX=<NNN-slug>          # empty if ON_DISCOVERY_BRANCH=no
DISCOVERY_DIR=.specify/discovery/<NNN-slug>   # empty if ON_DISCOVERY_BRANCH=no
NEXT_NUMBER=<NNN>                    # zero-padded, ready to use for new branches
```

Three sentinel-delimited lists follow:
```
BEGIN_EXISTING_BRANCHES_LOCAL ... END_EXISTING_BRANCHES_LOCAL
BEGIN_EXISTING_BRANCHES_REMOTE ... END_EXISTING_BRANCHES_REMOTE
BEGIN_EXISTING_DIRS ... END_EXISTING_DIRS
```

Exit code is always 0 (errors are surfaced via `IN_GIT_REPO=no`, not exit status). Bash 3.2 compatible — no `[[ ! ... =~ ... ]]`, no associative arrays. **Don't reintroduce the `[[ ! ... =~ ... ]]` pattern** — it broke once due to history-expansion escaping of `!`; `case` with glob is the established workaround.

Adding a new key is backward-compatible; renaming or removing one is not. Bump `CHANGELOG.md` accordingly when changing the protocol.

## Extension system primer (what you need to know)

- **Spec for extensions**: `extensions/EXTENSION-DEVELOPMENT-GUIDE.md` and `EXTENSION-API-REFERENCE.md` in `github/spec-kit`. Read those before changing `extension.yml` schema.
- **Manifest constraints**:
  - `extension.id` must match `^[a-z0-9-]+$` — ours is `discovery`.
  - Command names must match `^speckit\.<ext-id>\.<cmd>$`.
  - `version` is strict SemVer (`1.0.0` not `1.0` or `v1.0.0`).
- **Available hook points** (for v0.2): `before_specify`/`after_specify`, plus `_plan`, `_tasks`, `_implement`, `_clarify`, `_constitution`, etc.
- **Script path rewriting**: scripts referenced as `../../scripts/bash/foo.sh` in extension source become `.specify/scripts/bash/foo.sh` after install. We don't ship scripts yet — all helpers are inlined in command markdown.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bkuebler/spec-kit-discovery](https://github.com/bkuebler/spec-kit-discovery) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-29 -->
