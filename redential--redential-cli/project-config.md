---
trigger: always_on
description: Open-source CLI (`@redential/cli`, Apache-2.0) that produces METADATA-ONLY
---

# Redential CLI — repo rules

Open-source CLI (`@redential/cli`, Apache-2.0) that produces METADATA-ONLY
"proof bundles" from local git repos. The user's source code NEVER leaves
their machine. This repo is public and receives third-party PRs: every rule
here exists because trust is the product.

## What it is / what it isn't

- IS: a local detector. Reads `git log` retroactively, produces a JSON
  bundle validated against `schema/bundle.v1.json`, and uploads it ONLY
  with explicit user confirmation.
- IS NOT: a daemon, a real-time tracker, an automatic uploader. FORBIDDEN
  to implement watch mode, telemetry, or any upload without an explicit
  `redential submit`.

## Non-negotiable principles (see docs/principles.md)

Every principle has tests in `test/privacy/`. A PR that breaks a privacy
test does not get merged, no exceptions. Any change to WHAT data leaves
the machine requires: (1) a prior discussion issue, (2) a schema version
bump, (3) an entry in docs/schema.md and CHANGELOG.md.

## Security

- ZERO secrets in this repo. The CLI doesn't know anyone's API keys: only
  `SITE_URL` (public) and the user's token obtained via the device flow.
- User token: `~/.config/redential/credentials.json` with 0600
  permissions. Never in the cwd of the scanned repo. `redential logout`
  deletes it.
- Never log the token or the full bundle in errors. Stack traces without
  payload.
- Secret-scan of the PAYLOAD before any output/submit (mandatory): AWS key
  patterns, generic tokens, private keys, .env values.
- ZERO postinstall scripts in package.json. ZERO new dependencies without
  written justification in the PR (supply-chain surface). Allowed stack:
  commander, vitest. Everything else gets discussed first.
- `package.json` with explicit `files: ["dist"]`.
- Releases: only from GitHub Actions on tags, with `npm publish
  --provenance`. Release workflows NEVER run on `pull_request`.

## Conventions

- Strict TypeScript. Node >= 20. ESM.
- ALL code comments, docstrings, variable/function names, commit messages
  and docs in ENGLISH, always, no exceptions (public international repo).
  This is a public repo that receives third-party PRs — a single Spanish
  comment breaks the professional impression. If you catch a Spanish
  comment anywhere, translate it. The ONLY Spanish allowed is the
  "Explicación para el dueño" written in chat (never in files).
- Tests with vitest. Fixtures are git repos created programmatically in
  tmpdir (never committed fixtures with real history).
- Every feature: entry in CHANGELOG.md (Keep a Changelog, strict semver)
  + a doc in docs/ explaining how it works.
- Bundle schema changes = major or minor bump depending on compatibility.
- `submit` ALWAYS prints the exact byte-for-byte JSON immediately before
  the upload confirmation — unskippable, on every path. `scan` provides
  that same exact JSON via `--json` and piped/non-TTY output; its default
  TTY output is a human summary derived only from bundle fields.
- If the repo's remote looks like it's on a known public host
  (github.com/gitlab.com/bitbucket.org), `scan` suggests connecting the
  GitHub App as an alternative (anti-cannibalization guardrail) — but
  WITHOUT blocking the scan: a known host != actually public, and without
  network access there's no way to tell the difference. On a TTY it asks
  "Continue locally? (Y/n)" (default yes); piped/non-TTY runs are never
  prompted or blocked. The CLI's PRIMARY use case is exactly a private
  employer repo hosted on github.com; blocking there would break the
  product. Real visibility verification is left to `submit` (which does
  have network access).
- When closing out each milestone or large task, BEFORE the commit: write
  in the chat a short "Explicación para el dueño" (Explanation for the
  owner) — max 10 lines, in simple Spanish, no technical jargon. It must
  answer: (1) what we built and what it's for, (2) the 2-3 important
  decisions and WHY, (3) what would break if someone mishandles each key
  piece. Meant so a non-programmer running the product can re-explain it
  to someone else. This is NOT technical documentation — that always goes
  in docs/ as usual.
- After every push to `main`, check the CI result (`gh run watch`, or
  `gh run list`/`gh run view` on the run the push triggered) before
  considering the milestone closed. "Green locally" does not close a
  milestone if the matrix is red — CI runs a Windows/macOS/Linux × Node
  20/22 matrix that local runs don't cover, and cross-platform-only
  failures (e.g. resource-constrained CI runners exposing timing/load
  issues local machines don't hit) are real bugs, not noise to ignore.

## Limits for agents

- INVIOLABLE RULE — zero network in scan: `scan` makes NO network calls
  whatsoever. Skill detection is deterministic diff matching (read locally
  with `git show`/`git diff`) against `signatures/*.json` (a versioned
  signature database in this repo: imports, config files, per-library API
  patterns). No LLMs, no remote inference, in any variant.
- INVIOLABLE RULE — closed vocabulary: the bundle only admits skill slugs
  present in `taxonomy.json` (public, in this repo). A slug outside that
  list invalidates the bundle. New slugs come in via PR to `taxonomy.json`,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Redential/redential-cli](https://github.com/Redential/redential-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-17 -->
