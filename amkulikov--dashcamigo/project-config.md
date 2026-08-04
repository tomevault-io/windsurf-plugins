---
trigger: always_on
description: Guidance for Claude Code working in this repo: the constraints, the working rules,
---

# CLAUDE.md

Guidance for Claude Code working in this repo: the constraints, the working rules,
and a map of where to look - not a description of how the code currently works. When
guidance and code disagree, the code wins.

## What this is

A web-based, no-backend viewer for dashcam recordings. Multi-vendor through a
capability-first parser architecture: a new camera adds techniques to per-capability
libraries, it is not a "vendor" module.

Hard constraints - non-negotiable; changing one is an architectural decision. If a
task seems to require crossing one, stop and ask - never work around it silently:

- **No backend.** Video is never uploaded. The user picks local files, JS reads them
  in-browser via the File API.
- **Analytics is optional and separable.**
- **Graceful degradation.** If any external dependency (tile server, analytics) is
  down, the functional app keeps working - video, chart, markers and the track drawn
  on the map canvas are all local.

## Branches and deployment

- One working branch: `main` -> staging (https://beta.dashcamigo.app) on every
  push (`deploy.yml`). Production (https://dashcamigo.app) deploys on a `v*`
  tag push: `release.yml` builds and uploads via wrangler; the machine-managed
  `release` branch only records what production runs and only `release.yml`
  moves it - never touch `release` by hand. Never rebase `main`; on conflict,
  stop and surface it.
- Treat pushing as an outward action: commit locally freely, but do not push
  without an explicit request - a push to `origin/main` deploys staging.
- Promotion to production is the user's call and = pushing a tag
  `v<yyyy>.<mm>.<dd>[.<n>]` (zero-padded). The same tag publishes the GitHub
  Release with the prebuilt self-host artifacts (`.github/workflows/release.yml`,
  runbook in `docs/deploy.md`). Releases are immutable: a re-release is a new
  tag, never an overwrite of a published one.

## Working rules

### The private zone (`private/`)

- Everything under `private/` is local-only: never tracked by the main repo
  (`/private/` in the root `.gitignore`), never in the docker build context
  (`.dockerignore`). This is a single public repository - no mirror, no scrub
  stage between a commit and the world. `samples/` and `incoming/` are real
  user data. **Never commit it, never send it to an external service.** Share
  only anonymized fragments (coordinates rounded to whole degrees).
  Anonymization scripts are committed as `scripts/anonymize-*.mjs`.
- Text notes (`plans/`, `research/`, `*.md` only) are versioned by the nested
  git repo rooted at `private/` - its remote must be a private one. Commit note
  edits there in the same session that made them - nothing else keeps that
  history. Layout and rules: `private/README.md`.
- Unlike everything else in this file, a violation here is irreversible - a revert
  does not fix a leak.

### Code language

- Identifiers, in-code strings, filenames, commit and PR messages - simple English.
- Comments - simple English, compact. Explain WHY (the invariant, the reason, the
  non-obvious case); HOW only when naming does not carry it. Do not restate the code
  - a comment no shorter than the line it labels, carrying no WHY, is noise. No
  attribution or quotes: never name or cite a person/user/tester ("owner-confirmed",
  "X reported") - state the fact neutrally. "State the rule, not its history" (see
  Documentation) holds for comments too: no dates, "used to be", "was X in stage N",
  "previously".
- Error messages - English, lower-case, no trailing period (Go style).
- UI copy goes through i18n, never as a literal in code.

### TypeScript

- Errors, three tiers: an expected local failure (one record/line/block does not
  parse) returns null/empty; a whole file that matched a cheap marker but is not
  that format throws `WrongFormatError` (contract in `src/parsers/types.ts`) so
  dispatch falls through; an invariant violation throws a plain `Error`.
- In a `catch`: normalize (`err instanceof Error ? err.message : String(err)`)
  into a structured log field, or rethrow unchanged after cleanup; an
  `AbortError` (user cancel) always passes through.
- `any` is a last resort for a third-party lib's unusable types and carries a
  justifying comment. Unknown-shaped data (parsed JSON, worker messages,
  third-party events) is `unknown`, narrowed at the boundary. Casts belong at
  real boundaries (DOM reads, worker protocol) - the parser core stays
  cast-free. `arr[i]!` where the index is in-bounds by construction is the
  accepted idiom under `noUncheckedIndexedAccess`, not a bug to fix.
- Named exports only, no `enum` (string-literal unions), `interface` for object
  shapes, kebab-case filenames. A barrel `index.ts` is an explicit ordered
  registry - never a re-export convenience.
- `class` only for stateful lifecycle objects (a tracker session, a loaded
  model, a worker client); everything else is functions over interfaces.
- Relative imports end in `.js`. The bundler resolves without it; the codebase
  never omits it.
- Doc comments are prose stating the contract, not `@param`/`@returns` tag
  lists. UPPER_SNAKE_CASE only for literal constants; booleans read as
  `is`/`has`/`should`/`can`.

### Commits and git history


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [amkulikov/dashcamigo](https://github.com/amkulikov/dashcamigo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-31 -->
