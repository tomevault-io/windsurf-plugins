---
trigger: always_on
description: Guide for agents and humans working on the Rendobar CLI. Companion doc for the monorepo's `AGENTS.md` at [rendobar/rendobar](https://github.com/rendobar/rendobar).
---

# AGENTS.md — rendobar-cli

Guide for agents and humans working on the Rendobar CLI. Companion doc for the monorepo's `AGENTS.md` at [rendobar/rendobar](https://github.com/rendobar/rendobar).

## TL;DR

- **Source, tests, workflows**: all in this repo
- **SDK source**: lives in the [rendobar/rendobar](https://github.com/rendobar/rendobar) monorepo under `packages/sdk/`. Consumed here as `@rendobar/sdk@^1.0.0` from npm.
- **Release**: conventional commits → release-please → tag → `cli-binaries.yml` builds 5 platform binaries with attestations → GitHub Release
- **No manual tags, no manual version bumps**

---

## Cross-repo brand consistency

This repo is part of the broader Rendobar platform. The canonical reference
for brand strings, URLs, OG metadata, robots policy, and title/description
rules across **every** rendobar repo lives in the apex monorepo at
`rendobar/rendobar` `.claude/rules/brand-consistency.md`:

https://github.com/rendobar/rendobar/blob/main/.claude/rules/brand-consistency.md

Critical rules embedded here so you don't need to context-switch:

### Canonical brand strings (must match apex)

| Field | Value |
|---|---|
| Display name | `Rendobar` |
| Apex URL | `https://rendobar.com` |
| Apex page URLs | `https://rendobar.com/<path>/` (always trailing slash) |
| API URL | `https://api.rendobar.com` |
| Dashboard URL | `https://app.rendobar.com` |
| CDN URL | `https://cdn.rendobar.com` |
| Twitter handle | `@rendobar` |

**Forbidden variants**: `Rendobar.com`, `the Rendobar platform`, `rendobar`
(lowercase except in URLs / package names), `https://www.rendobar.com`,
`http://rendobar.com`, apex page links without a trailing slash.

### `package.json description` — em-dash, sentence case

- ✅ `"Rendobar CLI — serverless video processing from your terminal."`
- ❌ `"Rendobar | CLI Tool"` (pipe, title case)
- ❌ `"Rendobar CLI - serverless..."` (ASCII hyphen instead of em-dash)

### README + issue templates — trailing slash on apex page links

Apex Astro is configured with `trailingSlash: "always"`. Every link to an
`https://rendobar.com/<path>` page **must end with `/`**:
- ✅ `https://rendobar.com/docs/`, `https://rendobar.com/blog/`, `https://rendobar.com/pricing/`
- ❌ `https://rendobar.com/docs`, `https://rendobar.com/blog`

The apex root (`https://rendobar.com`) and subdomain roots
(`https://api.rendobar.com`, `https://app.rendobar.com`,
`https://cdn.rendobar.com`) are file/origin URLs and stay slash-less.

The install scripts at `https://rendobar.com/install.sh` etc. are FILES, not
pages — they correctly stay slash-less.

### No AI attribution in commits / PRs

Per the user's global rule (also in apex): never add `Co-Authored-By: Claude`,
`🤖 Generated with [Claude Code]`, or any AI attribution in commit messages or
PR descriptions. Strip them silently if a template injects them.

### Anti-patterns — never do these

- Use a brand-string variant other than canonical `Rendobar`
- Use `Rendobar | ...` (pipe) or `Rendobar - ...` (ASCII hyphen) in package.json description
- Skip the trailing slash on a `https://rendobar.com/<path>` link in README, issue templates, install scripts that print URLs, or workflow files
- Reference `rendobar.com/blog`, `rendobar.com/pricing`, `rendobar.com/docs` without trailing slash
- Add a `Co-Authored-By: Claude` or "Generated with [Claude Code]" line in commit messages or PR descriptions

## Dev loop

```bash
git clone https://github.com/rendobar/cli.git
cd cli && pnpm install
pnpm test          # 48 tests via bun test
pnpm typecheck     # tsc --noEmit
pnpm dev -- --version     # run from source
pnpm build         # compile standalone rb / rb.exe
./rb --version
```

## Working on SDK + CLI simultaneously

When you need unreleased SDK changes in CLI:

```bash
# Prereq: clone the monorepo as a sibling (or set RENDOBAR_MONOREPO env var)
#   ../rendobar/packages/sdk
pnpm dev:sdk-local    # builds monorepo SDK + pnpm-links it into this repo
pnpm test             # tests now run against your local SDK
# ...iterate...
pnpm dev:sdk-npm      # restores @rendobar/sdk from package.json (npm version)
```

**A pre-commit hook blocks commits while the SDK is linked.** It's automatic — if you forget to unlink, `git commit` fails with an instruction to run `pnpm dev:sdk-npm`.

The sibling path is `../rendobar/packages/sdk` by default. Override via:

```bash
RENDOBAR_MONOREPO=/custom/path/to/monorepo pnpm dev:sdk-local
```

## Conventional commits

Full rules + decision matrix + anti-patterns: **[.claude/rules/conventional-commits.md](.claude/rules/conventional-commits.md)** — read this before any commit.

Core rules:

- **R1** — `feat:` / `fix:` mean *user-visible* change. Adding infra / tests / CI / docs is **not** a feat or fix.
- **R2** — If the shipped artifact (binary + install scripts) is byte-identical to the previous release, no release. Pick a non-bumping type (`ci`, `chore`, `docs`, `build`, `refactor`, `test`).
- **R3** — `install.sh`, `install.ps1`, `uninstall.*` are public API. Functional changes to them bump.
- **R4** — `!` in the type for breaking changes (`feat!:`), not `BREAKING CHANGE:` in the subject.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rendobar/cli](https://github.com/rendobar/cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
