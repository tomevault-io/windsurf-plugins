---
trigger: always_on
description: - **Never push directly to main.** Always use a feature branch and open a PR.
---

# AGENTS.md — Home Keeper

## Workflow

- **Never push directly to main.** Always use a feature branch and open a PR.
- Wait for CI (tests, HACS validation, code review) and approval before merging.
- **Always squash merge PRs.**
- **CHANGELOG.md** — update for every user-facing change before tagging a release.
  Developer-only changes (CI config, AGENTS.md, IDEAS.md) don't need entries.
- **A stable release's `## [X.Y.Z]` notes describe what changed since the last
  _stable_ release — not since its betas.** When cutting `X.Y.Z` from an `X.Y.ZbN`
  line, write the section for someone upgrading from the previous stable version and
  roll the beta work into Added/Changed/Fixed as they'd perceive it. A feature
  introduced over the betas is **Added** (even if a later beta changed how it worked
  mid-stream); don't carry beta-to-beta framing — e.g. a `### Changed` for something
  that didn't exist in the last stable — into the stable section. **Include a `###
  Fixed` section listing every GitHub issue closed by commits since the last stable**
  — check the git log for `(Fixes #N)` / `(#N)` references and link each issue number
  so they auto-close on merge (`Fixes #N` in the CHANGELOG entry).
- **Beta versioning — always use the next release number.** After every stable
  `X.Y.0` ships, immediately bump `manifest.json` and `const.py` (`PANEL_VERSION`)
  to `X.(Y+1).0b1` on `main`, and rename the `## [Unreleased]` CHANGELOG section to
  `## [X.(Y+1).0b1]`. Beta iterations go `b1 → b2 → …` until the stable
  `X.(Y+1).0` is cut. **Never use `X.Y.0bN` after `X.Y.0` has shipped** — PEP 440
  sorts those below the stable version, so HACS would offer the stable as an
  "upgrade" to anyone on the beta, which feels like a downgrade.
- **Always cut a beta release for a new feature.** A PR that adds a user-facing
  feature must bump to the next beta in the same change — `manifest.json` +
  `const.py` (`PANEL_VERSION`) to the next `bN`, with a matching `## [X.Y.0bN]`
  CHANGELOG section — so the work ships to beta testers via HACS rather than waiting
  on the floor. (If the current top CHANGELOG section is an already-released beta,
  open the next `bN`; if it's an unreleased beta still being iterated, fold the
  feature into it.) Bug-fix-only / developer-only PRs don't need a fresh beta.
- **Always add the `preview-release` label to a new-feature PR.** As soon as the PR
  is open, apply the `preview-release` label so `preview-release.yml` publishes an
  installable ephemeral pre-release (`X.Y.Z.dev<pr>`) from the PR head — testers can
  try the feature via HACS *before* merge. The build is ephemeral and auto-deletes
  when the PR closes (see RELEASE.md → "Preview releases"). Bug-fix-only /
  developer-only PRs don't need it.
- **Always run tests locally before pushing.** Never use CI as the test runner.
  - Pure-logic unit tests need only `pip install pytest`: `pytest tests/unit -v`.
  - Full unit suite uses `pip install pytest-homeassistant-custom-component`.
- **Mutation testing gates every PR** at an 80% mutation score on the code the PR
  changed — see "Mutation testing" below. It is too slow for the
  run-before-you-push loop; run it when you touch the mutable surface.
- **User-facing prose is linted for AI-tell phrasing.** `lint.yml`'s `vale` job runs
  the [vale-ai-tells](https://github.com/tbhb/vale-ai-tells) Vale style (pinned in
  `.vale.ini`) over `README.md`, `CHANGELOG.md`, the canonical `docs/*.md` (not the
  scratch `*_PLAN.md`/research docs), `website/docs/intro.md`, `strings.json`,
  `services.yaml`, and the English frontend locale (`locales/en.json`), catching
  things like "delve", "it's important to note", em-dash overuse, and other
  AI-writing tells. **It's diff-scoped** (`filter_mode: added`): only lines your PR
  touches are checked, so it's a real gate on new prose without failing on the
  existing backlog. Run it locally with `vale sync && vale <paths>` (Vale CLI from
  [github.com/errata-ai/vale releases](https://github.com/errata-ai/vale/releases)).
  For an accepted false positive, either disable the rule for that file in
  `.vale.ini` (`ai-tells.RuleName = NO`) or wrap the exception inline with
  `<!-- vale ai-tells.RuleName = NO -->` / `<!-- vale ai-tells.RuleName = YES -->`.
  Diff-scoping only checks added/changed lines, so a wholesale rewrite of a file's
  prose (not just a small edit) can surface pre-existing hits on lines that just
  moved. Run `vale <file>` on the whole file yourself before a rewrite-style PR to
  catch those ahead of CI. There's no automated version-bump for the pinned
  `ai-tells.zip` release in `.vale.ini` (Dependabot/Renovate don't track raw
  GitHub release URLs), so bump it by hand occasionally, e.g. alongside the next
  full-corpus cleanup pass.
- **Every PR that touches the panel UI MUST include screenshots — no exceptions.**
  This is a hard gate: a UI change is not reviewable (or mergeable) until the PR
  body embeds current screenshots of the changed surface. The capture harness is
  `tests/e2e/screenshots.capture.ts` (the test) driven by `screenshots.config.ts`
  (the config — **pass this one to `--config`**, not the test file itself).
  Step-by-step:
  ```bash
  # 1. Start HA and leave it running (from repo root)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [prestomation/ha-home-keeper](https://github.com/prestomation/ha-home-keeper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
