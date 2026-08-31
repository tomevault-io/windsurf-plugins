---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`tailcast` — browser-to-browser screen sharing (no media audio) for a small group
inside a Tailscale tailnet. Bun + TypeScript, **no runtime dependencies**, no
build step, no `dist/`: the package ships `.ts` as written and Bun runs it
directly.

There *are* dev dependencies (`biome`, `husky`, `lint-staged`, `bun-types`), so a
clone needs `bun install` before the pre-commit hook works. Nothing they produce
is shipped — `files` in `package.json` lists 7 source entries and that is the
whole tarball.

Six shipped source files: `bin/tailcast.mjs`, `bin/cli.ts`, `server.ts`,
`stun.ts`, `public/index.html`, `public/sw.js`. Two test entry points and six
`bench/` modules on top.

### Three names, three jobs

- **Project**: `tailcast` — the repo, and every UI string.
- **Package**: `@thoth-dev/tailcast` — what `npm install`, `bun add` and `bunx`
  take, and what appears in the npmjs.com URL. `@thoth-dev/screen-share` still
  resolves as a migration alias.
- **Command**: `tailcast`, unscoped, because `bin` is keyed by the command and
  not the package. Once installed, the executable on `PATH` is `tailcast` (with
  `screen-share` kept as an alias), and the CLI's own `--help`, `--stop` and
  pidfiles all refer to itself that way.

Only the not-yet-installed, run-once-via-`bunx` case needs the package name;
everything downstream of installation uses the command name. Be consistent about
which is which when writing docs. A release touches a third org too: the **npm**
org is `thoth-dev`, the **GitHub** org is `thoth-id`, and `repository` in
`package.json` keeps `thoth-id` on purpose.

### Why there are two files in `bin/` and not one

On POSIX, npm links `node_modules/.bin/tailcast` straight at the file named in
`bin`, so the shebang picks the interpreter. With `bin/cli.ts` there (shebang
`bun`), a machine without Bun died in `env` — `env: 'bun': No such file or
directory` — before a single line of ours ran, which is why no message written
inside `cli.ts` could ever have explained it.

`bin/tailcast.mjs` has a `node` shebang and does three things: under Bun already
(`process.versions.bun`, i.e. `bunx`) it imports `cli.ts` directly, so
`process.argv` keeps the shape `argv.slice(2)` expects and no second process
appears; otherwise it looks for `bun` on `PATH`, in `$BUN_INSTALL/bin` and in
`~/.bun/bin` (that last one is the common failure: Bun installed, `npx` running
in a non-login shell that never read the rc) and spawns it; failing that it names
what is missing, why Bun and not Node (`Bun.serve`), and the install line, then
exits 1. It does **not** install anything: a published `bin` that curls a script
from another domain is the postinstall pattern nobody wants.

Being an intermediate process comes with an obligation the old layout did not
have: **the launcher relays `SIGINT`/`SIGTERM`/`SIGHUP` to the child and mirrors
its exit code.** Without the relay, `kill <pid>` on the pid the user can see
killed only the launcher and left `bun` orphaned holding the port (measured, not
assumed). Ctrl-C hides the bug, because the terminal signals the whole process
group; a targeted kill does not. When the child dies of a signal the launcher
removes its own listener for it before re-raising on itself, otherwise the
handler catches the re-raise and the launcher hangs forever trying to kill a dead
child.

Keep the launcher free of CLI logic. Flags, `--bg` and `--stop` all stay in
`cli.ts`.

### Language

**Everything written for a human to read is in English**: code comments, UI
strings, CLI help, stderr messages, docs. Comments start with a lowercase letter
and use no em-dashes. They earn their place by saying *why*, never by restating
what the line already says: the measured numbers, the bug that was reproduced,
the alternative that looks right and is not. If the code says it, delete the
comment.

Identifiers are in English (`resolveStatic`, `targetBox`, `pointUnder`,
`syncRoster`). A few Portuguese leftovers survive in `bench/scenarios.ts`
(`quadro`, `foco`) and in the `.vivo` class — rename them when you touch that
code, not as a sweep.

`localStorage` keys are `tailcast:name`, `tailcast:rooms`, `tailcast:sound` and
`tailcast:quality`.
The legacy `ss:` keys (`ss:name`, `ss:nome`, `ss:rooms`, `ss:salas`, `ss:sound`)
are still read for migration and then cleared, so existing users keep their name,
room history and sound preference without a hard cut. Every read is wrapped:
`localStorage` throws in a restricted context (cookies blocked, sandboxed
iframe).

## Commands

```bash
bun install                 # dev deps: biome, husky, lint-staged
bun run server.ts           # HTTP+WS on :3000, STUN on UDP :3478
bun bin/cli.ts --help       # authoritative flag list
bun run format              # biome check --write ./
bun run lint                # biome lint ./
```

`.husky/pre-commit` runs `npx lint-staged`, which runs `biome check --write` over
staged `.ts/.js/.json/.html/.css/.svg`. **Committing rewrites your staged
files** — tabs, width 100, double quotes, imports organized. Expect the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thoth-id/tailcast](https://github.com/thoth-id/tailcast) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
