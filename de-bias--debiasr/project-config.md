---
trigger: always_on
description: - keep project-specific guidance close to the repository
---

# debiasR Codex Instructions

Purpose:
- keep project-specific guidance close to the repository
- reduce repeated setup and task-board discovery work
- protect in-progress migration and documentation edits

Project scope:
- `debiasR` is an R package for origin-destination mobility bias correction and validation.
- The repository is public on GitHub as of 2026-06-04. Treat tracked files,
  docs, vignettes, workflows, issues, and pull requests as public-facing.
- Public adjustment functions must start with `adjust_`, validation functions
  must start with `validate_`, and measurement functions must start with
  `measure_`.
- Stable deterministic helpers use the `adjust_*` and `validate_flow_*` naming pattern.
- `adjust_multilevel_bayes()` is the main Bayesian path. The default
  `coverage_offset` model variant is approved for observed-flow LAD S1-S4
  workflows; `latent_two_level` is an approved advanced observed-row
  repeated-source S3/S4 variant with diagnostic guardrails.

Before substantial work:
- Read `notes/project-management/TASK_BOARD.md` and `notes/project-management/STATUS.md`.
- Check `git status --short` and avoid overwriting unrelated modified or untracked files.
- If the task touches validation, also check relevant notes in `notes/project-management/`.

Git and GitHub controls:
- All contributors and automation agents must work on a branch and open a pull
  request into `main`.
- Direct pushes to `main` are not part of the project workflow, including for
  maintainers and automation agents.
- Francisco Rowe (`fcorowe`) and Carmen Cabrera (`carmen-cabrera`) can review
  and merge accepted pull requests.
- PRs have two acceptable merge routes:
  - Review route: the default route. Wait for an approving review from an
    eligible reviewer before merging.
  - Francisco maintainer-bypass route: Francisco Rowe (`fcorowe`) may
    explicitly request that his own PR, or a PR opened on his behalf, bypass
    the review wait and be merged or squash-merged directly when deadlines or
    reviewer availability require it. This exception belongs only to Francisco
    as the main developer of `debiasR`; do not infer it for other contributors.
- The maintainer-bypass route still requires a branch, a pull request, a clear
  statement in the PR body or PR discussion that Francisco requested the
  bypass, and passing required checks unless Francisco explicitly accepts the
  remaining risk.
- If a direct push to `main` appears necessary, stop and ask Francisco to
  approve a PR-based route instead.

Coding defaults:
- Prefer existing package patterns in `R/`, `tests/testthat/`, and roxygen documentation.
- Keep patches narrow and traceable.
- Add or update focused tests for new exported behavior.
- Update `NAMESPACE` and generated `man/` docs when exports or roxygen docs change.
- Do not commit confidential material, credentials, personal local paths beyond
  necessary reproducibility notes, restricted raw data, or development-only
  artifacts that are not appropriate for a public repository.

Validation:
- Use the existing `validate_flow_*` API style.
- Keep validation functions deterministic and tidy-output friendly.
- Document metric interpretation clearly, especially sign conventions and scale.

Testing:
- Prefer the curated fast deterministic test runner when validating broad changes:
  `Rscript scripts/run_fast_tests.R`
- The hosted fast deterministic workflow is path-aware. It runs
  `scripts/run_fast_tests.R` for package-relevant changes under `R/`,
  `tests/`, `scripts/`, `inst/`, `data/`, `data-raw/`, `man/`, `src/`,
  `tools/`, or to `DESCRIPTION`, `NAMESPACE`, `.Rbuildignore`,
  configure/cleanup scripts, and the fast/Bayesian workflow files.
  Documentation, website-text, and governance-only PRs should let the required
  fast-tests job pass through its explicit skip step instead of forcing a full
  package test run; pkgdown still builds website-facing changes.
- For broader local development checks that should load the package with
  `devtools::load_all()`, use `Rscript scripts/run_dev_tests.R`.
- For narrow validation changes, targeted `testthat` runs are acceptable before the full fast tier.
- Optional Bayesian checks use `Rscript scripts/run_bayesian_tests.R <scope>`.
  Local scopes include `smoke`, `rstanarm-smoke`, `rstanarm`,
  `latent-smoke`, `latent-stress`, and `all`; the manual GitHub Actions
  Bayesian workflow exposes `smoke`, `latent-stress`, and `all`.

Documentation:
- Keep README, NEWS, status notes, and task board synchronized when user-facing scope changes.
- After changes that affect exported package behavior, public function
  interfaces, vignettes, README guidance, validation interpretation,
  dependency/data requirements, or methodological status decisions, update the
  project notes and software versioning before the change is considered done.
  At minimum, check whether `DESCRIPTION`, `NEWS.md`,
  `notes/project-management/STATUS.md`, and
  `notes/project-management/TASK_BOARD.md` need synchronized updates. Small
  typo fixes, hidden refactors, and test-only edits do not need a version bump
  unless they change user-facing behavior or project status.
- Keep adding release/development notes to `NEWS.md`, but do not display the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [de-bias/debiasR](https://github.com/de-bias/debiasR) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
