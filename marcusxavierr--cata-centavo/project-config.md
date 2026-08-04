---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`cata-centavo` is an MCP server that exposes Brazilian Open Finance data (via Pluggy) to an agent. The binary is a CLI whose default mode — no argument — is the MCP server over stdio.

`docs/adr/0001-stack-and-architecture.md` is the source of truth for every engineering decision here, including the ones still open. **Read the relevant section before implementing anything.** It is long, and it flags which phases are safe to start (0 and 0.5) and which depend on unanswered questions. When the ADR and this file disagree, the ADR wins.

## Commands

**Run `nvm use` first.** This machine's default node is v18, and the failure mode is misleading: a single test file dies with `ERR_UNKNOWN_FILE_EXTENSION`, while `npm test` reports `# tests 0` and exits 0 — a green run that executed nothing.

```bash
nvm use                  # reads .nvmrc → v24.15.0

npm run dev              # run the CLI from source (node executes .ts directly)
npm run typecheck        # tsc --noEmit — this is the linter, see below
npm run lint             # eslint as a sensor: warnings inform, errors fail
npm run deps             # dependency-cruiser — architecture rules, errors fail
npm test                 # node --test, finds tests/**/*.test.ts
npm run test:watch
npm run build            # tsc -p tsconfig.build.json → dist/
```

On demand, not part of the sequence above:

```bash
npm run mutation         # stryker over src/core + src/pluggy, then the agent report (~50s)
npm run mutation:report  # re-render the last report without re-running
```

Run a single test file or a single test:

```bash
node --test tests/cli/dispatch.test.ts
node --test --test-name-pattern="unknown command"
```

**Always run `npm run typecheck` before `npm run lint` before `npm run deps` before `npm test`.** Node strips types without checking them: `const x: number = "string"` runs fine. Without `tsc`, the project has no type checking at all. Typecheck first, then lint, then dependency rules, then tests, then build — the same order CI uses.

## The sensors sidecar

`sensors` runs all of the above on intervals in the background and answers with a summary instead of six tool invocations' worth of output. Installed globally with `uv tool install git+https://github.com/birgitta410/sensors-cli`; it is not a devDependency, and everything works without it.

**Always go through `.sensors/cli.sh`, never a bare `sensors`.** Fedora's `lm_sensors` package owns `/usr/bin/sensors`, so on this machine the name belongs to two programs and which one answers depends on PATH order. It works in an interactive shell and fails in anything with a sanitized environment. `cli.sh` resolves the right one and forwards.

```bash
.sensors/cli.sh start .          # background; the eight runners in .sensors/cata-centavo.sensors.yaml
.sensors/cli.sh start --show .   # foreground, live table
.sensors/cli.sh check .          # what to read: one table, exit 1 if a sensor failed
.sensors/cli.sh stop .
```

**Read `.sensors/cli.sh check .` instead of running the checks by hand when it is running.** A `Stop` hook in `.claude/settings.json` runs it at the end of every turn and blocks on a failure. It stays silent when the sidecar is not running, so starting it is optional.

Reading the table: `structure` and `types` describe things that fail the build. `lint` warns. `cov` and `mut_state` are informational and should not be chased. `mutation` is triggered rather than scheduled, but `start` fires it once, so starting the sidecar costs Stryker's 48 seconds.

Design and the traps found on Linux: `docs/plans/2026-07-26-sensors-sidecar-design.md`.

## Runtime constraints that bite

- **Node 24 (`.nvmrc` = v24.15.0)**, native type stripping, no `tsx`/`ts-node`, no build step in dev.
- **Development needs Node 24; the published package needs only 22.13** (`engines`). The two floors differ because the package ships compiled `.js` and nobody installing it strips types. Do not "fix" the mismatch by raising `engines` — and do not lower `.nvmrc`, because Node 22.13 cannot run `.ts` at all: `node --test` there reports `# tests 0` and exits 0. ADR §3.
- **No `enum`, no parameter properties** (`constructor(private x)`). `erasableSyntaxOnly` rejects them at tsc time because they crash at runtime with `ERR_UNSUPPORTED_TYPESCRIPT_SYNTAX`. Use a `const` object plus a derived union — see `src/cli/dispatch.ts` for the pattern.
- **Source files import `.ts` extensions** (`from "./balance.ts"`). `rewriteRelativeImportExtensions` turns them into `.js` on build.
- **Nothing but JSON-RPC may reach stdout.** In server mode stdout *is* the protocol channel. Every human-facing message, log line and error goes to stderr, always — including any fallback path in a logger.
- `noUncheckedIndexedAccess` and `exactOptionalPropertyTypes` are on deliberately. The first guards the pagination loops where a bad index means missing money; the second enforces "absence is `NULL`, never `''`".

## Architecture

```
src/
├── core/          business rules. no fetch, no sqlite, no SDK
│   └── contracts.ts    interfaces core requires of whoever serves it

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MarcusXavierr/cata-centavo](https://github.com/MarcusXavierr/cata-centavo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
