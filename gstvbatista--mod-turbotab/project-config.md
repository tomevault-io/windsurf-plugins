---
trigger: always_on
description: Guidance for AI agents (and humans) working in this repository.
---

# AGENTS.md

Guidance for AI agents (and humans) working in this repository.

## Project

`mod_turbotab` is a pure-Python telecom and contact-center capacity library,
exposed primarily through the `turbotab` CLI. It implements Erlang-style
queueing formulas, staffing calculations, queue metrics, and trunk sizing
helpers, with `--json` output designed for scripts and AI agents as much as
humans.

Key facts:

- Repository: `https://github.com/gstvbatista/mod_turbotab`, default branch `main`
- Runtime: Python >= 3.9, zero runtime dependencies (intentional)
- Published on PyPI as `turbotab` (`pip install turbotab`); console script
  `turbotab -> mod_turbotab.cli:main`
- The package root maps to the repo root (`package-dir` in `pyproject.toml`),
  so `import mod_turbotab` resolves from the repo's *parent* directory

## Repository map

| Path | Purpose |
|---|---|
| `cli.py` | `turbotab` CLI entry point: argument parsing, command groups, `--json` output |
| `calculations/erlang.py` | Erlang B, extended Erlang B, Engset B, Erlang C, Erlang A |
| `calculations/traffic.py` | Inversion/search helpers for traffic intensity |
| `agents/capacity.py` | Staffing, ASA, call capacity, fractional staffing |
| `queues/queues.py` | Queued percentage, queue size, wait time, service time, achieved SLA |
| `trunks/trunks.py` | Telephony trunk sizing |
| `utils.py` | Numeric helpers: clamping, ceiling, interval conversion |
| `exceptions.py` | Project-specific exception classes |
| `tests/test_cli.py` | CLI behavior tests |
| `skills/mod-turbotab/SKILL.md` | Agent-facing skill definition for the CLI |
| `README.md` | Primary user-facing API and mathematical model documentation |

Roadmap items are tracked as [GitHub issues labeled `roadmap`](https://github.com/gstvbatista/mod_turbotab/issues?q=is%3Aissue+label%3Aroadmap),
not as files in this repo. Each one is a full spec following the structure in
[`.github/ISSUE_TEMPLATE/roadmap_spec.md`](.github/ISSUE_TEMPLATE/roadmap_spec.md)
(Problem → What It Solves → How It Works → API Surface → Complexity →
Dependencies). When picking one up, read the issue for the full
problem/approach writeup before implementing; when adding a new roadmap item,
create an issue from that template.

## Workflow

- **Branches**: code changes go through a `feat/*` (or `fix/*`, `docs/*`)
  branch merged into `main` via pull request. Committing directly to `main`
  is acceptable only for trivial docs/config touches.
- **CI**: `.github/workflows/ci.yml` runs the test suite on Python 3.9-3.13
  for every PR and push to `main`. The `tests` summary job is a required
  status check — PRs merge only when it's green. Merges are squash-only.
- **Review findings**: after pushing a fix for a reviewer finding (Codex or
  human), reply in that thread with what changed and the commit hash (for
  traceability), then mark the conversation as resolved and re-request
  review (`@codex review`). Don't leave addressed threads open or resolve
  them silently.
- **Commits**: use conventional commits — `feat:`, `fix:`, `docs:`, `test:`,
  `refactor:`, `chore:`.
- **External PRs**: this repository does not accept external pull requests
  (see `CONTRIBUTING.md`); they are auto-closed by
  `.github/workflows/close-prs.yml`. PRs from the repository owner are exempt.
- **Releases**: bump `version` in `pyproject.toml` in a `chore: release X.Y.Z`
  commit, then create a GitHub Release; `.github/workflows/publish.yml`
  builds and publishes to PyPI automatically via trusted publishing. PyPI
  versions are immutable — never reuse a version number.
- **Release format**: tag and title are both `vX.Y.Z`, always created with
  `--generate-notes` so the "What's Changed" PR list and Full Changelog link
  are appended below the hand-written summary:

  ```bash
  gh release create vX.Y.Z --title "vX.Y.Z" --generate-notes --notes "<summary>"
  ```

  The hand-written summary is not a one-liner — it tells the story of the
  release. MINOR (feature) releases follow this structure:

  ```markdown
  ## <Feature name>

  <2-3 paragraphs: the operational problem this solves (speak to the
  WFM/planning audience, not just developers), what shipped, how it
  behaves, and the defaults/edge cases worth knowing about.>

  ### Usage

  <CLI example with real numbers and expected output; Python example
  when the API surface is the point.>

  ### Changes

  - Added: <new functions, flags, modules>
  - Changed: <behavior changes, even compatible ones>
  - Fixed: <fixes that rode along>

  ### Compatibility

  - <breaking changes with migration notes, or "No breaking changes —
    defaults preserve previous behavior.">
  - Supported Python: 3.9-3.13

  Closes #<issue> via #<PR>
  ```

  PATCH releases use `## Fixes` with one detailed bullet per fix —
  symptom, cause, and what changed: `- <symptom>: <what was wrong and
  what changed> (#<PR>)`.
- **Versioning (semver)**: feature PRs do NOT touch `version`; the bump
  happens in a release commit on `main` right after the merge:
  - **Every merged `feat:` PR ships as its own MINOR release**
    (`0.1.0 -> 0.2.0`): merge, bump `version`, tag, GitHub Release. No
    batching — one feature, one release.
  - **PATCH** (`0.2.0 -> 0.2.1`): `fix:` merges; these may batch — cut the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gstvbatista/mod_turbotab](https://github.com/gstvbatista/mod_turbotab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
