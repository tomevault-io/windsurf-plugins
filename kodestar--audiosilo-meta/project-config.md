---
trigger: always_on
description: Guidance for working in this repository. Keep this file updated as the codebase
---

# CLAUDE.md - AudioSilo Meta

Guidance for working in this repository. Keep this file updated as the codebase
evolves. This is the sixth repo in the AudioSilo workspace (`~/dev/audiosilo`) -
read the workspace [CLAUDE.md](../CLAUDE.md) and
[META-FEASIBILITY.md](../META-FEASIBILITY.md) (the research + design basis for
this project) before working here.

## What this is

An **open, community-editable audiobook metadata database** - the data behind
meta.audiosilo.app (deployment pending). The GitHub repository IS the database:
JSON pack files holding many records each ([PACK-SPEC.md](PACK-SPEC.md)),
contributed via pull requests and issue forms, validated by Go tooling in CI,
and compiled into a SQLite artifact published as a GitHub Release. The Go API server (`metaserve`) serves that artifact plus the
static site in `site/` (Astro, the audiosilo.app design system); the AudioSilo player
integration (Phase 1.5) is the priority consumer and shipped **before** the
Audiobookshelf-provider facade (`GET /abs/search`), which has since landed on top
of it (the player was always the defining AudioSilo feature; the ABS facade is a
bonus for a direct competitor's users).

Module path: `github.com/kodestar/audiosilo-meta`. Code is AGPL-3.0; the data
is CC0-1.0 (factual core) with a CC BY-SA 4.0 community layer (the
characters/recaps sidecars) - the full policy
in [LICENSING.md](LICENSING.md) is load-bearing, read it before touching data
handling.

**Since 2026-08-21 the CC BY-SA layer is a SECOND REPOSITORY**,
[`KodeStar/audiosilo-meta-community`](https://github.com/KodeStar/audiosilo-meta-community),
holding the `works-community` family alone (plus its own issue forms, intake and
AUTHORING/EXTRACTION guides). This repository is the CC0 core - works, people,
series and `redirects.json` - and holds no share-alike content; its tree profile
(PACK-SPEC.md) is `core`, which CI passes explicitly. All the TOOLING stays here
and the community repo consumes it, so there is one implementation of pack math,
canonical JSON and validation. The release stream is unchanged: `release.yml`
checks out both repositories and `metabuild -data data --community <dir>`
composes ONE artifact from the pair, so metaserve, the poller, the patch chain
and the webhook see exactly what they saw before. A community-side merge cannot
push here, so that repo's `notify-core.yml` sends a `community-data`
`repository_dispatch` and this repo's `release.yml` accepts it as a third
trigger. The migration plan is the maintainer's, in the untracked
`.claude/community/PLAN.md`.

## Model routing (every session follows this)

- **Fable (the main session) is the orchestrator only.** Task decomposition,
  design taste, final QA. It never writes feature code directly; it may write
  orchestration artifacts (this file, briefs, governance docs, commit messages).
  Runs at high effort.
- **Opus subagents do the implementation**, one per task, parallel when tasks
  are disjoint. Each gets a self-contained brief and must leave the gate green.
- **Token-hungry chores go to cheaper models** (Sonnet/Haiku): fact research,
  bulk data sweeps, log triage.

## Build / test / gate

```sh
cd ~/dev/audiosilo/audiosilo-meta
go build ./... && go vet ./... && go test -race ./... && golangci-lint run
# ~3min wall for the -race suite; no -timeout flag needed. pkg/check's real-data
# test skips under -race (raceEnabled, race_{on,off}_test.go) - the fixture
# suites cover the parallel loader and the real tree is data coverage, still
# validated by the non-race run and by metacheck below.
go run ./cmd/metacheck --profile core       # validate the data tree (~10s over 133k works)
go run ./cmd/metafmt --check --profile core # canonical formatting (--write to fix)
go run ./cmd/metabuild -o meta.sqlite   # build the CORE-ONLY artifact (no sidecars)
go run ./cmd/metabuild -data data --community <community-repo>/data -o meta.sqlite  # ... the REAL artifact, from both repos
go run ./cmd/metaserve --db meta.sqlite --addr :8080   # serve the read-only API
go run ./cmd/metaaudit -data data -o audit-report  # read-only data-quality audit (~30s)
go run ./cmd/metarepair -data data --community ../audiosilo-meta-community/data \
  --op merge-works --limit 50                     # DRY RUN of the repair (~30s)
```

`--profile core` is what this repository's data root now IS (PACK-SPEC.md's tree
profiles): works, people, series and `redirects.json`, with the CC BY-SA layer in
the community repo. THE DEFAULT DIFFERS BY TOOL, on purpose, and the axis is the
COST OF BEING WRONG:

- **metacheck / metafmt / metaissue default to `all`.** "A data root" with nothing
  said must keep meaning the whole database - that is what the fixtures and most
  tests are - and CI passes `--profile core` explicitly at every
  invocation site: check.yml, the intake bot's compose and normalize steps, the
  rebase sweep, and release.yml's core-side validation. A wrong reading there is a red check, or (for
  metaissue) a bot pull request a maintainer closes.
- **metaaudit / metarepair default to `core`** (approved deviation, 2026-08-21).
  They are operator tools a human points at THIS tree by typing a command, and
  metarepair DELETES RECORDS - so the dangerous case must not be reachable through

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KodeStar/audiosilo-meta](https://github.com/KodeStar/audiosilo-meta) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-19 -->
