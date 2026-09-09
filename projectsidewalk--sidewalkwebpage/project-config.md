---
trigger: always_on
description: Project Sidewalk is a web-based crowdsourcing tool for mapping and assessing sidewalk accessibility. Scala 2.13 +
---

# CLAUDE.md

Project Sidewalk is a web-based crowdsourcing tool for mapping and assessing sidewalk accessibility. Scala 2.13 +
Play 3.0 (Java 17) backend, Postgres + PostGIS via Slick, and a vanilla-JS frontend that Grunt concatenates (no
transpile, no minify, no module system), all run in Docker. Request flow is routes → Controller → Service → Table
(DAO). Architecture tour: `docs/architecture.md`. Setup, daily commands, troubleshooting: `docs/dev-environment.md`.

## 🚨 NEVER READ `docker-compose.override.yml` 🚨

Real live secrets, nothing enforces this but you. Never open it, never print or commit a value from it, and exclude
it from wide `grep`/`find`/`cat *` sweeps. Ask the maintainer for a value; `docker-compose.yml` has dummy equivalents.

## Starting on an issue? Assign it first

The moment you begin implementing or debugging an issue (not when filing, reading, or triaging it), assign the
developer you act for: `gh issue edit <n> --add-assignee @me`. Assignment means "someone is on this now", so it is
never done at filing time and never skipped. If someone else is already assigned, say so before adding yourself.

This file holds only cross-cutting rules. `.claude/rules/` surfaces path-scoped essentials when you touch a matching
file, and this table says which doc to read first:

| Working on… | Read first |
|---|---|
| A schema change (`conf/evolutions/`) | `docs/evolutions.md` |
| A `/v3/api` endpoint or `app/models/api/` | `docs/architecture.md` → "The public API" (and the `update-apis` skill) |
| Translations (`conf/messages/`, `public/locales/`) | `docs/internationalization.md` |
| CSS, Twirl views, any UI | `docs/style-guide.md`, `docs/accessibility.md` |
| A new or changed user interaction | `docs/logged-events.md` |
| Releases, deploys, asset caching, persistent media dirs | `docs/deployment-and-stages.md` |
| Storing uploaded media (DB row vs. media dir) | `docs/architecture.md` → "Media storage" |
| Label crops, or a marker drawn on one (`label_crop` provenance, #2660) | `docs/architecture.md` → "Media storage" |
| Tests or CI | `docs/testing-and-ci.md`, `test/e2e/README.md` |
| `scripts/*.py` | `scripts/README.md` |
| Onboarding a new city (streets, regions, schema, configs) | `docs/onboarding-a-city.md` (and the `onboard-city` skill) |
| Google Maps keys, quotas, or a Google Cloud bill | `docs/google-cloud.md` |
| The label lat/lng estimator or the labeling viewport frame | `docs/label-latlng-estimation.md` |

## Workflow

- `develop` is the main branch and the PR target; `master` is the release branch. Branch names start with the
  issue number (`1234-fix-label-popup`).
- Start a ticket by branching from an up-to-date `develop`, unless the maintainer says otherwise.
- **Never open a pull request, merge, tag, or release without the maintainer's explicit OK.** Do the work, run the
  checks, push the branch if useful, then stop and ask. Filing GitHub issues is fine. Maintainers: @jonfroehlich
  and @misaugstad.
- Prod deploys are tag-triggered (`vX.Y.Z` on `master`); pushing `develop` redeploys the test stage.
- Edit `src/` files only. Never run grunt or edit `build/` output: the developer's `npm start` runs `grunt watch`.
  A new `src/` file must match a glob in `Gruntfile.js`.
- Keep docs in sync in the same change. `docs/architecture.md` is the human-facing architecture reference; exact
  dependency versions live only in `docs/upgrading-libraries.md`.
- Never browser-test anything that needs a street-view panorama (placing labels, validating). Hand the developer a
  checklist or console snippet instead. The `test/e2e/` suite only *loads* Explore's tutorial and Validate's landing
  state; don't extend it into pano interaction.

## Before a change is done

- **Scala:** `make scalafmt-fix` (a blocking CI gate). Compile check without fighting the developer's `sbt ~ run`:
  `docker exec projectsidewalk-web bash -lc "cd /home && sbt --client compile"`. `-Xfatal-warnings` is on, so a
  success is warning-clean.
- **Frontend:** `make lint` (ESLint, Stylelint, HTMLHint, locale parity, CSS layout, asset paths, evolutions lint;
  all blocking CI gates), or scope it with `make eslint dir=…` / `make stylelint dir=…`. `make lint-fix` handles the
  mechanical fixes. The tree is lint-clean, so any finding is from your change.
- **Tests:** `sbt --client test` (same `docker exec`; needs the db container), `make test-js` (jsdom unit suite),
  `make test-e2e` against a running app, `make test-python`. Details and what CI gates: `docs/testing-and-ci.md`.

## Conventions the linters can't check

- **ES2022.** As you touch code, modernize it: constructor functions → `class` with `#private` fields, jQuery →
  `fetch` + Promises, Bootstrap → native (defer a refactor that would ripple through many callers). Build HTML with
  template literals, never `+` concatenation.
- **Comments say *why*, never what.** ScalaDoc (`@return`) / JSDoc (`@returns`, typed `@param`) on every class and
  non-trivial method, including private ones. Never describe what code *used to* do; git history has that, and a
  hook flags it. Templates: `docs/style-guide.md` → "Comments".

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ProjectSidewalk/SidewalkWebpage](https://github.com/ProjectSidewalk/SidewalkWebpage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
