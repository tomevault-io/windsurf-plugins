---
trigger: always_on
description: Guidance for AI agents and engineers working in **wait-on**. This file is the single
---

# AGENTS.md

Guidance for AI agents and engineers working in **wait-on**. This file is the single
source; `CLAUDE.md` only imports it.

## What wait-on is

`wait-on` is a cross-platform CLI + Node.js API that waits for files, ports, TCP
sockets, and http(s) resources to become available — or, with `--reverse` / `reverse`,
to go away. It is a widely-used build/test utility (e.g. wait for a dev server before
running e2e tests).

Two front doors to the same behavior — **keep them in sync**:

- **API** — `waitOn(opts[, cb])` in `lib/wait-on.js` (package `main`, `lib/wait-on`).
  Omit the callback and it returns a Promise; pass `cb(err)` and it uses the callback
  form. Options are validated against `WAIT_ON_SCHEMA`.
- **CLI** — `bin/wait-on` (help text in `bin/usage.txt`). A new user-facing option needs
  the schema entry *and*, when exposed on the CLI, a flag in `bin/wait-on`, an entry in
  `bin/usage.txt`, and a `README.md` update.

## Architecture

An rxjs polling pipeline in `lib/wait-on.js`:

- `waitOn` → `waitOnImpl`: validate `opts` against the joi `WAIT_ON_SCHEMA`, fail fast on
  malformed resources (`validateResources`), build one observable per resource with
  `createResource$`, then `combineLatest` them and `merge` in a `timer(timeout)` error
  observable. The stream runs `takeWhile(states => states.some(x => !x))` until every
  resource reports ready, then `cleanup` fires the callback once.
- Each resource polls with `timer(delay, interval)` → `mergeMap` (concurrency =
  `simultaneous`) → check → `startWith(false)` → `distinctUntilChanged()` → `take(2)`.
- **Resource types by prefix** (`PREFIX_RE`, dispatched in the `createResource$` switch):
  - `file:` (default when no prefix) — waits for the file to exist and its size to
    stabilize over `window`.
  - `http:` / `https:` — HEAD returns 2xx.
  - `http-get:` / `https-get:` — GET returns 2xx.
  - `tcp:` — `tcp:host:port` connects (bare port defaults to localhost; `[ipv6]:port` ok).
  - `socket:` — connects to a unix-domain socket.
  - `http://unix:<sock>:<path>` — http over a unix socket / Windows named pipe.
- **Reverse mode** (`reverse: true` / `--reverse`): each check is inverted (async checks
  via `negateAsync`; the `file:` check flips to "size is -1"), so `waitOn` succeeds when
  the resources are *un*available.
- **Validation**: `WAIT_ON_SCHEMA` (joi) defines every option and its default;
  `validateResource` rejects syntactically bad http/tcp resources up front with a clear
  error instead of polling until timeout.
- **HTTP**: requests go through axios with the http adapter forced
  (`axios.create({ adapter: 'http' })`), which avoids xhr/jsdom log pollution.

**Add a new resource type:** extend `PREFIX_RE`, add a `case` in the `createResource$`
switch, write a `create<Type>$` factory following the existing
`timer → mergeMap → startWith(false) → distinctUntilChanged → take(2)` shape (honor
`reverse` via `negateAsync`), and add a `case` in `validateResource` when the resource
has syntax worth failing fast on.

## Stack

- Node `>=20`, plain CommonJS (`"type": "commonjs"`, `'use strict'`), no build step.
- Runtime deps (current on master): `axios` (http), `rxjs` (polling/merge), `joi`
  (`WAIT_ON_SCHEMA`), `lodash` (via `lodash/fp`).
- CLI args are parsed with Node's built-in `util.parseArgs` (minimist was removed, #233).
- **Upcoming:** jeffbski/wait-on#238 raises the engines floor to `>=22.19` and #238/#239
  propose dropping `axios`/`lodash`. Describe the current state above; do not assume those
  have merged.

## Commands

- `npm test` — the full check: `npm run lint && npm run test:mocha`.
- `npm run lint` — eslint over `lib/**/*.js`, `test/**/*.js`, `bin/wait-on`
  (flat config `eslint.config.mjs`).
- `npm run test:mocha` — `mocha --exit "test/**/*.mocha.js"` (`--exit` is required: spun-up
  test servers leave open handles).
- `npm run test:coverage` — nyc + mocha.
- Node engines floor is `>=20.0.0` on master.

## Conventions

- CommonJS throughout; keep it (no ESM, no build step).
- Tests: mocha + chai, files `test/*.mocha.js` (`api.mocha.js`, `cli.mocha.js`,
  `validation.mocha.js`); shared fixtures `test/config-http-resources.js` and
  `test/config-headers.js`. How to write them: see
  [Test-Driven Development](#test-driven-development-mandatory).
- CI runs on **ubuntu + windows** (matrix node 20/22/24, `npm ci --engine-strict`). No
  POSIX-only assumptions: mind Windows named pipes and path separators, and don't rely on
  unix-only tooling (e.g. `openssl speed`) or shell.
- Conventional Commit messages (semantic-release + commitlint are proposed in #241).
- Keep `README.md` (and `bin/usage.txt`) in sync whenever options or CLI flags change.
- `.npmignore` hygiene: exclude new top-level dev/tooling files from the published package.
- CLI headers: `-H` / `--header "Name: value"` is repeatable and merges with config-file
  headers, CLI winning on conflict (#234).

## Test-Driven Development (Mandatory)

**All executable code is written test-first. No exceptions.** That covers `lib/`, `bin/`,
`index.d.ts`, scripts, test helpers, and configuration, tooling, and CI changes; "trivial",
"just wiring", and "just a rename" are not exemptions. The one carve-out is docs-only

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jeffbski/wait-on](https://github.com/jeffbski/wait-on) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-04 -->
