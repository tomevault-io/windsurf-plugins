---
trigger: always_on
description: This repo contains **two distinct things** that live side-by-side:
---

# CLAUDE.md

## What This Repo Is

This repo contains **two distinct things** that live side-by-side:

1. **The AWOS framework** — markdown files in `commands/`, `templates/`, `claude/`, `scripts/`, and `plugins/`. These are the actual product: AI-agent prompts, document templates, and a Claude Code plugin. They never execute as code in this repo; they get copied into a user's project.
2. **The installer** — JavaScript code in `src/` and `index.js` (entry point). Published to npm as `@provectusinc/awos` and runnable via either Node (`npx`) or Bun (`bunx`). Its only job is to copy framework files into a user's project. See `src/CLAUDE.md` for installer internals.

When working here, identify which layer your change touches. Editing a prompt under `commands/foo.md` is product work; editing `src/services/file-copier.js` is installer work.

## Critical Rule: Do Not Run the Installer Here

**Never run the installer inside this repo** — neither `npx @provectusinc/awos` / `npx ./index.js` nor `bunx @provectusinc/awos` / `bun index.js`. The installer creates `.awos/`, `.claude/`, and `context/` directories — running it here pollutes the source tree with copies of files that already exist as originals. To test installer changes, run it against a separate scratch project as described in `CONTRIBUTING.md`.

## Common Commands

```sh
# Format check — CI-enforced quality gate (pick one runner):
npx prettier . --check
bunx prettier . --check
npx prettier --write .     # auto-format before committing
bunx prettier --write .

# Run the test suite (no npm deps; node --test built-in):
npm test                   # all four layers (markdown lint, installer, fixtures, engine)
npm run test:lint          # Layer 1 — static prompt linter
npm run test:installer     # Layer 2 — installer unit tests
npm run test:fixtures      # Layer 3 — fixture-project end-to-end
npm run test:coverage      # prints per-file coverage table for src/
npm run test:coverage:gate # fails if coverage drops below env thresholds
bun test --coverage tests/ # local cross-runtime coverage (Bun version)

# Behavioral / session-log E2E lives in the awos-qa repository
# (sibling to this one). See its README for how to run.

# Test installer against a separate project (pick one runner; $AWOS_REPO is the absolute path to this repo):
cd ~/some-scratch-project
npx $AWOS_REPO/index.js
bunx $AWOS_REPO/index.js
bun $AWOS_REPO/index.js          # direct exec also works
npx $AWOS_REPO/index.js --dry-run   # preview only
```

The installer runs on **Node 22+ or any recent Bun**. It uses only standard JS built-ins (`fs`, `path`) via CommonJS `require`, which both runtimes support — do not add npm dependencies or runtime-specific APIs without strong justification, as that would break cross-runtime compatibility.

## Testing

The repo has a three-layer test suite under `tests/`, all built on Node's `node:test` built-in — no npm dependencies. See `tests/README.md` for the detailed reference.

1. **Static prompt linter** (`tests/lint-prompts.test.js`) — symmetry, frontmatter, marker presence, cross-references, dimension DAG, copy-table consistency, and grep-style checks for required substrings inside prompt bodies.
2. **Installer unit tests** (`tests/installer/*.test.js`) — exercises the installer services against temp directories.
3. **Fixture projects** (`tests/fixtures.test.js` + `tests/fixtures/<name>/`) — real installer runs against representative pre-install trees, with manifest-based assertions.

All three layers run in CI via `npm test`, which also runs the engine test layer (see "Running the engine tests" below).

### Coverage

`npm run test:coverage` runs the full suite under Node 22's built-in `--experimental-test-coverage` and prints a per-file table for `src/**` (the installer entry point `src/index.js` is excluded — it's just CLI plumbing). `npm run test:coverage:gate` adds three threshold flags that fail the run when coverage drops below the configured floor.

CI runs both: a non-blocking **coverage-report** job that just prints the table, and a **coverage-gate** job that enforces hardcoded thresholds. To raise the floor, edit `COVERAGE_LINES` / `COVERAGE_FUNCTIONS` / `COVERAGE_BRANCHES` in `.github/workflows/quality-check.yml`.

Local Bun fallback: `bun test --coverage tests/` produces an equivalent table (slightly different column set) when Node isn't installed.

Behavioral end-to-end tests — the ones that run a real Claude Code session against a seeded scratch project and assert on the actual tool-call trace — live in the separate **`awos-qa`** repository (sibling to this one). See its README for how to run them.

### Tests must narrate what they checked

Output that says `N events found` or `M pass` tells you the suite ran, not what was validated. `assert.*` failure messages should name the contract being violated, not just dump a diff. Anyone reading the test output should understand which contracts were verified without opening the test source.

### Adding tests for new contracts


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [provectus/awos](https://github.com/provectus/awos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
