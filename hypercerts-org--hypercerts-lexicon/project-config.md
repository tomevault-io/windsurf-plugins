---
trigger: always_on
description: This file provides guidance to AI coding assistants which are
---

# AGENTS.md

This file provides guidance to AI coding assistants which are
modifying code in this repository.

If however you are building on _top_ of this repository rather than
_in_ it, then instead see
[`building-with-hypercerts-lexicons/SKILL.md`](.agents/skills/building-with-hypercerts-lexicons/SKILL.md)
and the [`README.md`](README.md).

## ⚠️ CRITICAL REMINDER FOR AI AGENTS

**ALWAYS CREATE A CHANGESET** when making changes that affect the public API:

- Adding or modifying lexicon files
- Changing generated TypeScript exports
- Renaming constants, types, or functions
- Any change that users of this package will need to know about

Use the `writing-changesets` skill. **DO NOT skip this step!**

## Branch Strategy

**`main` is the only evergreen branch** and the default branch on
GitHub. Normal releases are published from `main`. Note that `main`,
the latest npm package release, and the lexicons published on ATProto
may not always be perfectly in sync due to the multiple moving parts
in the publishing process.

- **`main` branch**: Preparation for stable releases, which will be
  tagged and published from this branch.
- **`prerelease/*` branches**: Ephemeral branches for beta/prerelease
  versions (created from `main`, merged back when done; see
  [docs/PUBLISHING.md](docs/PUBLISHING.md))
- **`feature/*` (or `fix/*`) branches**: Short-lived branches for
  development work, targeting and merged to `main` or a
  `prerelease/*` branch via PR depending on whether a beta or
  prerelease is required.

> **Do not open PRs against `develop`** — it is a stale branch left
> over from a previous workflow and is no longer used.

## Overview

This repository contains ATProto lexicon definitions for the Hypercerts
protocol. Lexicons define record types that can be stored on the ATProto
network using the ATProtocol (ATProto) standard.

The codebase consists of:

- **Lexicon definitions**: JSON files in `lexicons/` that define record
  schemas
- **Generated TypeScript types**: Auto-generated in `generated/` directory
  (gitignored, do not edit manually)
- **Built output**: Compiled bundles in `dist/` directory (gitignored)
- **Documentation**: Markdown files including README.md and ERD.md.

## Guidance for development of downstream applications

As already mentioned above, downstream applications should _not_
depend on this document for guidance, which is intended for usage when
modifying this repository.

However for the sake of clarity, consume these lexicons **NOT** by
reading from `main` or other development branches of the repository,
but instead via the following published releases:

- **For TypeScript / JavaScript code** — use [the npm package
  `@hypercerts-org/lexicon`](https://www.npmjs.com/package/@hypercerts-org/lexicon),
  which includes generated types, validation helpers, and schema
  constants.
- **For other languages** — use the [tagged
  releases](https://github.com/hypercerts-org/hypercerts-lexicon/releases)
  published in this GitHub repository.

Both npm releases and git tags follow [SemVer](https://semver.org/).
For npm, you can depend on a version range to receive compatible
updates automatically. For GitHub releases/tags, pin a specific tag
or upgrade manually to a newer compatible SemVer release.

The raw lexicons published on ATProto can also be used, but they are
(unavoidably) missing useful context such as full documentation
(including changelogs), TypeScript type definitions, SemVer
guarantees, git history, and other tooling provided by the packaged
releases.

## Critical Build System Detail

**The `generated/` and `dist/` directories contain auto-generated code.
Do not edit files in these directories directly.**

To regenerate code after modifying lexicon JSON files:

```bash
npm run gen-api
```

This runs `lex gen-api` on all lexicon JSON files and:

1. Generates TypeScript types in `generated/` (including vendored
   external lexicons from `lexicons/pub/` and `lexicons/app/bsky/`)
2. Auto-generates `generated/exports.ts` with clean exports

Then to build the distributable bundles:

```bash
npm run build
```

This runs Rollup to compile TypeScript and bundle into ESM, CommonJS,
and type declaration files in `dist/`.

## Development Commands

**⚠️ IMPORTANT FOR AI AGENTS**: Always run scripts through npm scripts
(e.g., `npm run gen-schemas-md`) rather than executing Node.js files
directly (e.g., `node scripts/generate-schemas.js`). This ensures
proper environment setup and consistency with the project's workflow.

### Code Generation

```bash
# Regenerate TypeScript API types from lexicons and auto-generate generated/exports.ts
npm run gen-api

# Manually regenerate just generated/exports.ts
npm run gen-index

# Build distributable bundles
npm run build

# Generate SCHEMAS.md from lexicon definitions
npm run gen-schemas-md

# Generate markdown documentation from lexicons
npm run gen-md

# List all lexicon JSON files
npm run list
```

### Code Quality

```bash
# Check code formatting
npm run lint
# or
npm run format:check

# Auto-fix formatting issues
npm run format
```

### Validation

```bash
# Validate lexicon definitions, regenerate types, typecheck, and build
npm run check

# Type-check TypeScript without building
npm run typecheck
```

### Changesets and Versioning


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hypercerts-org/hypercerts-lexicon](https://github.com/hypercerts-org/hypercerts-lexicon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-23 -->
