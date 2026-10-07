---
trigger: always_on
description: This package has **two** Vitest configs (plus a self-skipping live-Postgres mode under the first). Run the right one for the change.
---

# `stash` — agent notes

## Two test suites

This package has **two** Vitest configs (plus a self-skipping live-Postgres mode under the first). Run the right one for the change.

| Command | Config | Scope | Needs build? |
| --- | --- | --- | --- |
| `pnpm --filter stash test` | `vitest.config.ts` | Unit tests under `src/__tests__/**` and `src/**/__tests__/**` | **Partly** — needs `@cipherstash/stack` built (see below). Turbo's `^build` supplies it in CI. |
| `pnpm --filter stash test:e2e` | `vitest.integration.config.ts` | E2E tests under `tests/e2e/**.e2e.test.ts` driving the built `dist/bin/stash.js` through a real pty (`node-pty`) | **Yes** — run `pnpm --filter stash build` first, or use the turbo `test:e2e` task which depends on `build`. One test also needs protect-ffi's native binding, which no `build` produces (see below). |

The unit config explicitly excludes `tests/e2e/**` so the default `pnpm test`
stays fast.

**Live-Postgres suites run under the unit config but self-skip.**
`src/**/__tests__/**.live.test.ts` gate on `STASH_TEST_DATABASE_URL` via
`describe.skip`, so `pnpm --filter stash test` stays green with no database and
CI reports them as skipped. To actually run them:

```bash
docker compose -f local/docker-compose.postgres.yml up -d --wait
STASH_TEST_DATABASE_URL=postgres://cipherstash:password@localhost:55432/cipherstash \
  pnpm --filter stash test
```

They need Postgres only — no CipherStash credentials. Reach for one when a
change turns on a real SQL string, a real SQLSTATE, or a driver type
conversion: a faked `pg` returns whatever the test author typed, so none of
those is observable under it. `commands/eql/__tests__/applied.live.test.ts` is
the worked example — it found that a missing *schema* raises `42P01`, not the
`3F000` the code's own comment assumed.

It is **not** fully self-contained, despite running standalone in CI. Some `src`
modules import workspace packages that publish `./dist` only, so an unbuilt
workspace fails at collection with `Failed to resolve entry for package …`
rather than at an assertion. `vitest.config.ts` aliases `@cipherstash/migrate`
to its source to remove one such coupling; `@cipherstash/stack` remains, reached
via `languages/typescript/packages/migrate/src/backfill.ts` and a direct import in
`init/lib/__tests__/introspect.test.ts`. Deleting `languages/typescript/packages/stack/dist` fails 10
files. Closing it needs `vitest.shared.ts`'s `stackSourceAlias`, which cannot be
spread into this config: its `'@/'` points at `languages/typescript/packages/stack/src` while this
package's points at `languages/typescript/packages/cli/src`, and a flat alias map admits only one —
spread it after and stack's entry clobbers the CLI's, spread it before and the
CLI's breaks stack's own source imports (#787 review).

## When to add or update an E2E test

Update `tests/e2e/**` whenever you:

- Add or rename a top-level command, subcommand, or flag (smoke tests assert
  on help text, command names, and unknown-command behavior).
- Change the user-facing string for an exit message that an existing E2E
  asserts on (e.g. cancellation text, "Unknown auth command", the
  `db migrate` stub warning). Strings that tests assert on live in
  `src/messages.ts` — update the constant there and both prod and tests
  pick it up. *Don't* hard-code the new wording in a test.
- Touch `src/bin/stash.ts` argv parsing, exit codes, or top-level error
  handling.
- Add a new clack prompt that changes the *first* prompt rendered for a
  command currently covered by E2E (the cancel test waits for a specific
  prompt label).

You do **not** need to add an E2E test for every new flag or branch — keep
E2E coverage to the highest-value flows. Unit tests still own the bulk of
behaviour coverage.

## How the harness works

`tests/helpers/pty.ts` exports `render(args, opts?)` which spawns
`dist/bin/stash.js` inside a real pseudo-terminal and returns:

- `output` — cumulative ANSI-stripped stdout.
- `raw` — same, with ANSI escapes preserved (handy when debugging).
- `waitFor(text|regex, timeoutMs?)` — polls until the match appears.
- `key(name)` — sends keystrokes (`Enter`, `Up`, `Down`, `CtrlC`, etc.).
- `write(string)` — raw stdin write.
- `exit` — promise resolving to `{ exitCode, signal? }`.
- `kill(signal?)` — terminate the pty.

A real pty is required because `@clack/prompts` switches stdin to raw mode
and renders differently when stdout isn't a TTY; piped-stdin mocks don't
exercise the same code paths.

## Gotchas

- **Build before E2E.** `dist/bin/stash.js` is the artifact under test. The
  turbo `test:e2e` task already depends on `build`, but if you invoke the
  script directly you must build first.
- **`doctor.e2e.test.ts` also needs protect-ffi's native binding, and no
  `build` produces one.** `stash doctor` probes the encryption engine by
  *calling* through `@cipherstash/stack/diagnostics` — importing it proves
  nothing, since the neon load is lazy. `@cipherstash/stack` is a devDependency
  of this package, so in the workspace the probe always resolves it and never
  takes the "not installed, that's fine" arm; and the workspace-linked

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cipherstash/stack](https://github.com/cipherstash/stack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
