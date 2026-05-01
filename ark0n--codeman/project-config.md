---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Quick Reference

| Task | Command |
|------|---------|
| Dev server | `npm run dev` (or `npx tsx src/index.ts web`) |
| Type check | `tsc --noEmit` |
| Lint | `npm run lint` (fix: `npm run lint:fix`) |
| Format | `npm run format` (check: `npm run format:check`) |
| Single test | `npx vitest run test/<file>.test.ts` |
| Production | `npm run build && systemctl --user restart codeman-web` |

## CRITICAL: Session Safety

**You may be running inside a Codeman-managed tmux session.** Before killing ANY tmux or Claude process:

1. Check: `echo $CODEMAN_MUX` - if `1`, you're in a managed session
2. **NEVER** run `tmux kill-session`, `pkill tmux`, or `pkill claude` without confirming
3. Use the web UI or `./scripts/tmux-manager.sh` instead of direct kill commands

## CRITICAL: Always Test Before Deploying

**NEVER COM without verifying your changes actually work.** For every fix:

1. **Backend changes**: Hit the API endpoint with `curl` and verify the response
2. **Frontend changes**: Use Playwright to load the page and assert the UI renders correctly. Use `waitUntil: 'domcontentloaded'` (not `networkidle` — SSE keeps the connection open). Wait 3-4s for polling/async data to populate, then check element visibility, text content, and CSS values
3. **Only after verification passes**, proceed with COM

The production server caches static files for 1 year (`maxAge: '1y'` in `server.ts`). After deploying frontend changes, users may need a hard refresh (Ctrl+Shift+R) to see updates.

## COM Shorthand (Deployment)

Uses [Semantic Versioning](https://semver.org/) (`MAJOR.MINOR.PATCH`) via `@changesets/cli`.

When user says "COM":
1. **Determine bump type**: `COM` = patch (default), `COM minor` = minor, `COM major` = major
2. **Create a changeset file** (no interactive prompts). Write a `.md` file in `.changeset/` with a random filename:
   ```bash
   cat > .changeset/$(openssl rand -hex 4).md << 'CHANGESET'
   ---
   "aicodeman": patch
   ---

   Detailed description of ALL changes since last release (not just the most recent commit — review full git log since last version tag)
   CHANGESET
   ```
   Replace `patch` with `minor` or `major` as needed. Include `"xterm-zerolag-input": patch` on a separate line if that package changed too.
3. **Consume the changeset**: `npm run version-packages` (bumps versions in `package.json` files and updates `CHANGELOG.md`)
4. **Sync CLAUDE.md version**: Update the `**Version**` line below to match the new version from `package.json`
5. **Commit and deploy**: `git add -A && git commit -m "chore: version packages" && git push && npm run build && systemctl --user restart codeman-web`

**Version**: 0.6.0 (must match `package.json`)

## Project Overview

Codeman is a Claude Code session manager with web interface and autonomous Ralph Loop. Spawns Claude CLI via PTY, streams via SSE, supports respawn cycling for 24+ hour autonomous runs.

**Tech Stack**: TypeScript (ES2022/NodeNext, strict mode), Node.js, Fastify, node-pty, xterm.js. Supports both Claude Code and OpenCode AI CLIs via pluggable CLI resolvers.

**TypeScript Strictness** (see `tsconfig.json`): `noUnusedLocals`, `noUnusedParameters`, `noImplicitReturns`, `noImplicitOverride`, `noFallthroughCasesInSwitch`, `allowUnreachableCode: false`, `allowUnusedLabels: false`.

**Requirements**: Node.js 18+, Claude CLI, tmux

**Git**: Main branch is `master`. SSH session chooser: `sc` (interactive), `sc 2` (quick attach), `sc -l` (list).

## Additional Commands

`npm run dev` = dev server. Default port: `3000`. Commands not in Quick Reference:

| Task | Command |
|------|---------|
| Dev with TLS | `npx tsx src/index.ts web --https` |
| Continuous typecheck | `tsc --noEmit --watch` |
| Test coverage | `npm run test:coverage` |
| Production start | `npm run start` |
| Production logs | `journalctl --user -u codeman-web -f` |

**CI**: `.github/workflows/ci.yml` runs `typecheck`, `lint`, `format:check` on push to master/main and on PRs (Node 22). Tests excluded (they spawn tmux).

**Code style**: Prettier (`singleQuote: true`, `printWidth: 120`, `trailingComma: "es5"`). ESLint flat config (`config/eslint.config.js`) allows `no-console`, warns on `@typescript-eslint/no-explicit-any`. Ignores: `app.js`, `scripts/**/*.mjs`, `src/web/public/vendor/**`, `scripts/remotion/**`.

## Common Gotchas

- **Single-line prompts only** — `writeViaMux()` sends text+Enter separately; multi-line breaks Ink
- **ESM only** — Never `require()`, use `await import()`. `tsx` masks CJS/ESM issues in dev but production breaks
- **Package ≠ product name** — npm: `aicodeman`, product: **Codeman**. Release renames tags accordingly
- **Global regex `lastIndex`** — Use `createAnsiPatternFull/Simple()` factories, not shared `g`-flag patterns in loops

**Import conventions**: Utils from `./utils`, types from `./types` (barrel), config from specific `./config/*` files.

## Architecture

### Core Files (by domain)

| Domain | Key files | Notes |
|--------|-----------|-------|
| **Entry** | `src/index.ts`, `src/cli.ts` | |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ark0N/Codeman](https://github.com/Ark0N/Codeman) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
