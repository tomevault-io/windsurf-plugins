---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Lolly is a constraint-first, template-driven platform for generating on-brand creative assets (PNG/SVG/PDF/video/etc.) from simple inputs. A single platform-agnostic **engine** runs the same render path across multiple **shells** (web PWA, Tauri desktop/mobile, CLI). Tools are **data, not bundled code** - a manifest + template + optional hooks - synced to clients so new tools ship without app updates.

The package name, repo, and working directory are all `lolly`.

## Commands

```bash
# This repo is split into submodules - community/ (tools), brands/suse/ (PRIVATE brand pack),
# services/{mcp,ca}, docs/, and every shells/* live in github.com/lolly-tools/*.
# Clone with --recurse-submodules, or:
git submodule update --init --recursive   # REQUIRED before npm install (workspaces need every package.json)
                                          # brands/suse is `update = none` (private) - SUSE devs opt in:
git submodule update --init --checkout brands/suse

npm install                  # install workspace deps; postinstall builds the tools/ + catalog PROFILE VIEWS

# Content profiles - tools/ and catalog/ at the repo root are gitignored VIEWS of the
# active profile (profiles.json), built by scripts/use-profile.ts. NEVER commit them.
npm run profile              # show active + available profiles
npm run profile:suse         # community + SUSE tools, SUSE catalog (needs brands/suse mounted)
npm run profile:start        # blank brand: community tools + a single neutral tokens asset (brands/lolly-start)
npm run ingest:brand -- <src> --name <brand> [--register|--activate]  # hydrate a brand pack from DTCG/Tokens-Studio/Penpot tokens (scripts/ingest-brand.ts)

npm run dev:web              # run the web shell (Vite) + live-rebuild the /info site on docs changes
npm run build:web            # production build of the web shell - builds the /info site first

# Run a tool headlessly via the CLI shell (jsdom + same engine path as web)
npm run cli                                              # list available tools
npm run cli -- qr-code                                   # show a tool's inputs
npm run cli -- qr-code --url=https://suse.com --output=./qr.svg
npm run cli -- qr-code --url=https://suse.com --export=png > qr.png

npm run validate:catalog     # validate every tool.json + asset against schemas & invariants
npm run build:catalog        # regenerate catalog/tools/index.json + asset checksums

# ...but both of the above only see the ACTIVE profile's view. `catalog/tools/index.json` is
# generated PER BRAND, so editing a community tool (which every brand's index lists) updates
# the active brand and leaves the others stale - and `validate:catalog` can't see that either,
# because it validates the active view too. The drift surfaces with no context on a public
# clone (no brands/suse → falls back to lolly-start) or in CI. After ANY community tool.json
# edit, use:
npm run build:catalog:all    # rebuild every mounted profile's catalog, then restore the active one
npm run validate:catalog:all # validate every mounted profile; exits 1 on drift. CI runs plain validate:catalog on the lolly-start view (public clone), so run this one locally before pushing a community tool.json edit
npm run build:info           # build the docs/info site once (docs/build.ts → shells/web/public/info/). Add --watch to rebuild on change; dev:web runs it in --watch, build:web runs it once. Plain `npm run dev` in shells/web does NOT build /info.
```

### Tests

The engine contract test suite lives at the repo root (`tests/`, node:test, no framework); `npm test` runs it together with the co-located suites in `engine/src/**`, `packages/{core,node-shell,docs-render}/test/`, `shells/web/src/**`, `shells/tui/src/**` and `services/mcp/test/` (the roots are `TEST_ROOTS` in `scripts/run-test-suite.ts`; see `tests/README.md` for layout + gated tests). To run just the repo-root suite:

```bash
node --test "tests/**/*.test.ts"
```

Use the quoted glob, not `node --test tests/` - on current Node the bare directory form tries to load `tests` as a module instead of discovering test files. The tests import engine modules across the workspace boundary via `../engine/src`, so the repo root owns the run.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lolly-tools/lolly](https://github.com/lolly-tools/lolly) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-07 -->
