---
trigger: always_on
description: **byre** is a small Go binary that runs an AI coding agent in a throwaway,
---

# CLAUDE.md

## Project Overview

**byre** is a small Go binary that runs an AI coding agent in a throwaway,
project-scoped container — a local-first, inspectable, Docker-native harness.
`cd ~/project && byre develop` generates a Dockerfile from a config cascade,
builds it (Docker/Podman), and runs the agent in a sandbox that sees only the
project and what you explicitly grant. Mechanics: `docs/ARCHITECTURE.md`. User
docs: `README.md`.

**Vocabulary is canonical in `docs/GLOSSARY.md`** (the domain-modeling skill's
`CONTEXT.md`, renamed -- treat it as that skill's glossary file). It is binding
for prose, docs, and user-facing strings; vocabulary only, never behavior. When
another doc disagrees with it on naming, one of them is wrong -- reconcile.
**The TUI is the gold (PRINCIPLES.md P0).** Without it byre is a fusty
one-of-many sandbox -- containerising an agent is table stakes; the screen
over it is the differentiator. So: a config key with no reachable row in the
editor is a hole in the product -- a widget where the editor owns the key, a
read-only row naming the writer where a flow owns it -- not a deferred
nicety; a TUI bug ranks with an engine bug; the
demo casts are product, not decoration; and "expert vocabulary, hand-edit it"
is not an answer byre gives.
**Design principles live in `docs/PRINCIPLES.md`** (standing commitments);
**doc/site placement rules in `docs/PLACEMENT.md`** (cited as `PLACEMENT.md Pn`);
**point-in-time decisions live in `docs/adr/`** and cite principles as
rationale. Litmus: could it be "superseded by ADR-NNNN"? ADR. Would changing
it re-litigate the project? Principle.

**`docs/` holds settled references only** -- ALL-CAPS files, plus `adr/`
(decision records) and `marketing/`. Anything with a lifecycle -- designs in
flight, research drafts, parked options -- lives in `wip/` at the repo root
and is DELETED when the work ships (absorbed into an ADR and/or the docs;
git history keeps it -- see `wip/README.md`). Never start a working document
in `docs/`.

**byre runs on the host** (where Docker is). The dev *container* is where the
agent writes Go, runs `go build`, and runs unit tests; the actual `byre develop`
/ integration runs happen host-side.

## Dev environment (self-hosted)

byre develops itself. `byre develop` in this repo (config applied from `byre.preset`) builds a
**Go + Claude** box with these skills:

- **codex** — installs the `codex` binary (the independent reviewer; not launched
  as the agent). Authenticate once per box with `codex login`.
- **pjlsergeant/codereview** — ships **`byre-codereview`** (on `PATH`) and the
  review-loop conventions.
- **pjlsergeant/devlog** — the dev-workflow conventions (diary, commit
  discipline, the scratch volume). Both skills' conventions are placed in the
  box as agent memory, so the workflow rules below are reinforced automatically.
- **pjlsergeant/inttest** — ships **`byre-inttest`** (on `PATH`): sync the tree
  to the sacrificial Lima VM and run the gated `BYRE_DOCKER_TESTS=1` suite
  there. Lives IN this repo (`skills/inttest/`); the VM template rides the
  package. The dev-environment mechanics — this skill, the `skills/` dir's
  packed-manifest edit loop, VM setup — are in `docs/BYRE-DEVELOPMENT.md`.

**One-shot bootstrap (fresh machine):** `codereview` and `devlog` moved out of
the byre binary (2026-07-13) into
[pjlsergeant-byre-skills](https://github.com/pjlsergeant/pjlsergeant-byre-skills).
On a fresh clone, `byre preset apply` here reviews this repo's `byre.preset`
and chauffeurs the installs (once per machine); the preset's `[sources]`
block pins their URIs and digests. A config that references the qualified ids
without the installs fails loudly at develop with those exact commands.
(`pjlsergeant/inttest` rides the same flow but installs from a path source —
this repo's own `skills/inttest/skill.toml` — so run the apply from the repo
root; see `docs/BYRE-DEVELOPMENT.md`.)

## Workflow

- **Autonomy.** Keep going through the work; don't stop to ask "should I
  continue?" after each step. Stop only when genuinely blocked.
- **Commit frequently** — after each coherent unit (a function + tests, a fix, a
  green refactor). Small, well-described commits.
- **Code review (mandatory after a feature/fix).** Run `byre-codereview`
  yourself, read every finding, fix or consciously defer each, then re-run with
  `byre-codereview --continue "..."`. Stop when clean.
- **Substantial work gets TWO reviewers, not one reviewer twice.** Run the
  other one too (`--reviewer grok` when codex is the default, and the
  reverse), and run it INDEPENDENTLY: no briefing on what the first found.
  Neither reviewer is told the other's findings, so agreement between them
  is evidence rather than an echo, and a shared blind spot shows up as a
  gap in both. Concurrent is fine, and cheaper in wall-clock than waiting
  for the first to come back clean. `--continue` deepens one perspective; it
  does not add another. Type specimen, 2026-07-28: codex ran five rounds on a batch and
  called it clean; grok then found three real defects it had never seen,
  two of them the sibling-bound class — including a concurrency hole that
  protected the first save and not the next one.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pjlsergeant/byre](https://github.com/pjlsergeant/byre) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
