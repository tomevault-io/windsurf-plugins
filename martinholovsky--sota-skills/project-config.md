---
trigger: always_on
description: Operational guidance for AI assistants (and humans) working **on** this
---

# AGENTS.md

Operational guidance for AI assistants (and humans) working **on** this
repository. This is the SOTA-skills library — Markdown skills that an AI
assistant reads to build and audit software. There is no application to run;
changes are edits to Markdown held to a few hard invariants. See
[CONTRIBUTING.md](CONTRIBUTING.md) for the full conventions.

This file is the single source of truth for every agent: tools that follow
the [AGENTS.md standard](https://agents.md) (Codex, Cursor, Copilot, …) read
it directly, while `CLAUDE.md` (Claude Code) and `GEMINI.md` (Gemini CLI) are
symlinks to it — edit only this file, never the symlinks.

## Landing a change

`main` is a protected branch and **direct pushes are rejected for everyone**
(admin enforcement is on). Every change goes through a pull request:

1. `git checkout -b <branch>`
2. make the edit, then run `./scripts/check-invariants.sh`
   (and optionally `pre-commit run --all-files`)
3. push the branch and open a PR
4. all four required checks must pass, then squash-merge — invariants, secret scan,
   shell lint, and the negative-control harness that proves the gates can still fail

## Invariants (enforced in pre-commit and CI)

`scripts/check-invariants.sh` runs **16 checks** and fails the build on any of
them. One line each below. The *rationale* — and the real incident behind every
one — lives in the script's own header, at the point of use, and the practical
"what this means for your PR" version is in
[CONTRIBUTING.md](CONTRIBUTING.md#the-invariants-enforced).

| # | The build fails when… |
|---|---|
| 1 | a **skill** file (`skills/*/SKILL.md`, `skills/*/rules/*.md`) exceeds **500 lines** |
| 2 | a `skills/*/rules/*.md` doesn't end with `## Audit checklist` |
| 3 | an internal-name denylist hits (the library must stay generic) |
| 4 | a `SKILL.md` `description` exceeds **1024 chars** (spec cap — loaders silently skip it), is unquoted YAML containing `: `, or either `name`/`description` contains an **XML tag**; also a reserved word (`anthropic`, `claude`) in `name` |
| 5 | `VERSION`, `plugin.json` and the CHANGELOG top entry disagree, or a tag is ahead of `VERSION` |
| 6 | a count-bearing surface drifts from a recount of `skills/` (the social-preview pill and README alt are **"N+" floors**) |
| 7 | a skill is missing from the router's routing table **or** its library map |
| 8 | a relative Markdown link to a `*.md` target doesn't resolve |
| 9 | `CHANGELOG.md` carries more than one `## [Unreleased]`, or it isn't the top entry |
| 10 | a `rules/*.md` isn't referenced by its own `SKILL.md` — written, capped, checklist-ed, and never loaded |
| 11 | `LAST-VERIFIED` moves without a sweep. Escapes: a sweep-shaped diff (≥ 20 skill files) or naming it in the CHANGELOG. The only **diff-based** check; skips with a note when there's no merge base |
| 12 | an `assets/*.png` is older than the `*.html` it renders — the README embeds the *image*, never the source, so an un-rendered fix reaches nobody. Escape: `[no-render]` in the commit subject |
| 13 | a scoreboard row in `evals/results/RESULTS.md` leaves its `Samples` cell empty |
| 14 | a **release** (VERSION changed) carries no `**Front door checked:**` line in its CHANGELOG section, or a declared term resolves in neither `README.md`/`docs/INDEX.md` nor the release's own entry |
| 15 | the router's **library map** omits a `rules/NN` file that exists, or names one that doesn't — checks 7 and 10 both miss this, and `rules/11` went unlisted for two releases |
| 16 | the hook `README.md` **documents** differs from the one `install.sh` **writes** (`HOOK_CMD`) — the README's is what a reader copies by hand, so a stale block is the version that spreads |

**Only instruction files are capped.** A file is capped if and only if an agent
loads it *as instructions* — `skills/*/SKILL.md` and `skills/*/rules/*.md`, and
nothing else in the repo. README, CHANGELOG, `docs/`, `evals/`, this file and every
script are **uncapped** and deliberately so (decided 2026-07-15): navigability there
comes from a table of contents and [docs/INDEX.md](docs/INDEX.md), not a line
ceiling. **If you find a line-cap claim anywhere that does not say *skill files*, it
is stale — fix it.** The 500 matches the Agent Skills guidance (*"keep `SKILL.md`
under 500 lines; move detailed reference material to separate files"*), and those
separate files are exactly what `rules/*.md` are.

**This file is the exception that proves it.** `CLAUDE.md` and `GEMINI.md` symlink
here, so this file loads into **every** session — the platform's guidance is to
*"target under 200 lines per CLAUDE.md file"*, because long always-loaded files
reduce adherence. That is a different constraint from invariant 1 and is not gated.
Keep it under 200: put detail in `CONTRIBUTING.md` and leave a pointer.

**Every file-list-driven check reports its denominator** (`ok (257 rules files)`)
and **fails closed on an empty scope**. Added 2026-07-30 after a mutation showed
checks 2 and 10 printing `ok` — and the script exiting 0 — while examining *zero*
files. `0 checked, 0 failed, exit 0` is the signature of a gate that verifies
nothing (`sota-code-security` rules/11 §2.2). Adding a check? The script's header

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [martinholovsky/SOTA-skills](https://github.com/martinholovsky/SOTA-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
