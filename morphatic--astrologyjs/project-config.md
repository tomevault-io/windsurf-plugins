---
trigger: always_on
description: An astrological charting library for JavaScript/TypeScript, backed by the Morphemeris ephemeris API.
---

# astrologyjs

An astrological charting library for JavaScript/TypeScript, backed by the Morphemeris ephemeris API.

## Project state — read this first

This is a **revival**, not a greenfield project. `astrologyjs@1.3.1` is published on npm with 154
GitHub stars and 45 forks, and it is broken: its ephemeris backend
(`http://www.morphemeris.com/ephemeris.php`) died years ago, and four open issues going back to 2017
all report the same failure. People are still starring it and still hitting the wall.

Two consequences that shape every decision:

- **There are real users on the other side of this.** Error messages, README copy, and issue replies
  matter as much as the code. The register is plain and unspun: this was a part-time
  project that outran its author's bandwidth, the backend went away, and there is a real service
  behind it now. Do not oversell, and do not apologize at length.
- **The domain model is worth keeping.** `Chart`, `Planet`, `Aspect`, `Person`, `ChartFactory`, and
  the aspect math in `src/` are fine — pure computation that does not care where the numbers came
  from. What must change is the data layer and the build.

Start with `plinth/specs/astrologyjs.nlspec.index.md`, then load the contract. It is the binding
spec and it supersedes any earlier planning document.

### Decisions already made

- **ESM-only.** `2.x` ships no CJS build. Node 20.19+/22.12+ support `require(esm)`, Node 18 is EOL,
  and dropping CJS avoids the dual-package hazard on a class-based domain model.
- **Callers supply their own Morphemeris API key.** No shipped shared key, no free proxy — both
  recreate the failure being fixed. See spec §4 and §11.
- **Local computation stays local.** Morphemeris provides ephemeris and houses; aspects, synastry,
  and composites are derived in-library. Server-side equivalents are optional extras, not the
  default path. See spec §1.2 and §1.4.
- **Trunk-based branching**, `main` only. (`master` was renamed to `main` on 2026-08-09.)
- **Toolchain:** tsup + vitest + TypeScript, matching `@morphemeris/mcp`. The 2016 gulp/rollup/
  jasmine/istanbul/Babel/Travis stack has been removed; recover from git history if ever needed.

## Working in this project

- **`plinth/`** holds all agent-facing context: specs, research, and planning. Load from here when
  you need background.
- **Specs use the tiered NLSpec format** (index + contract + rationale). Start with
  `plinth/specs/astrologyjs.nlspec.index.md` to orient, then load the contract (`.nlspec.md`) for
  implementation work. Pull the rationale (`.nlspec.rationale.md`) only when you hit a judgment call.
- **Slash commands:** `/spec-quick` to draft an NLSpec from a freeform description, `/spec-interview`
  for the structured interview flow, `/spec-audit` for completeness audits.

### Local context not in this repository

- `plinth/private/` — untracked and gitignored. Holds planning, positioning, and strategy notes that
  do not belong in a public repository. Present in Morgan's working tree only; absent from a clone.
  Anything commercially or legally sensitive goes here, not in `plinth/specs/`.
- `~/dev/morphemeris/` — the API this library consumes. Its NLSpec is the authority on endpoint
  shapes, auth, error codes, and rate limits, and wins wherever this repo restates it. `mcp/` there
  is the toolchain precedent.

## Git workflow

Branching model: trunk-based (`main` only). Feature branches merge directly into `main`.

Rules (enforced by lefthook + CI, not just convention):

- **Never commit directly to `main`.** The pre-commit hook blocks this. Create a working branch:
  `git checkout -b feat/<slug>` or `fix/<slug>`, `chore/`, `docs/`, `refactor/`, `test/`, `perf/`.
- **Commit messages use Conventional Commits.** The commit-msg hook enforces the format
  `<type>[(<scope>)][!]: <subject>`. See `.gitmessage` for the reference.
- **Never push directly to `main`.** Pre-push hook blocks this; open a PR with
  `gh pr create --base main --fill`.
- **Merge via squash only.** Configured at the repo level by `.github/setup-github.sh`.
- **Wait for CI after EVERY push, not just the first of a session.** `gh pr checks --watch <num>`.
  Never use `--admin` to bypass.
- **After merge, branches are auto-deleted** (repo setting).
- **Releases go through release-please.** It reads Conventional Commits on `main` and maintains a
  release PR; merging that PR tags the release and publishes it. See "Releasing and publishing".
- **A breaking change must be marked in a commit that lands on `main`** — `feat!:` in the subject, or
  a `BREAKING-CHANGE:` footer. Putting `!` only in a *pull request title* is not enough unless the PR
  is squash-merged, because a merge commit preserves the branch's individual commit messages and
  discards the PR title. This has already bitten once: see the note in "Releasing and publishing".

## Releasing and publishing

One workflow, `.github/workflows/publish.yml`, is the whole pipeline:

1. Land Conventional Commits on `main` (the commit-msg hook already enforces the format).
2. release-please maintains a release PR with the version bump and generated `CHANGELOG.md`.
3. Merging that PR tags the release **and publishes to npm in the same run**.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [morphatic/astrologyjs](https://github.com/morphatic/astrologyjs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
