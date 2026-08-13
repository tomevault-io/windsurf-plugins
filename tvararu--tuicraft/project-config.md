---
trigger: always_on
description: Use `mise` to run tasks (not `bun` directly, not `mise run`):
---

# AGENTS.md

## Commands

Use `mise` to run tasks (not `bun` directly, not `mise run`):

- `mise test` — run all tests (`bun test`)
- `mise typecheck` — type-check (`tsc --noEmit`)
- `mise format` — check formatting (`biome format`)
- `mise format:fix` — fix formatting (`biome format --write`)
- `mise bundle` — install dependencies (`bun install`)
- `mise ci` — run typecheck, test, and format; then `gh signoff ci`
- `mise test:live` — run live server tests (`bun test ./src/test/live.ts`)
- `mise build` — compile single binary (`bun build --compile`)
- `mise test:slowest` — show 10 slowest tests via junit XML
- `mise worktree <branch>` — create a feature worktree under `.worktrees/`
- `mise worktree:clean <branch>` — remove a worktree and delete the branch

## Code Style

- Follow `/typescript-style` skill for structure and taste decisions
- Strict TypeScript — `noUncheckedIndexedAccess`, `noUnusedLocals`,
  `noUnusedParameters`, all strict flags on (see tsconfig.json)
- Never write comments
- Use Bun APIs over Node.js equivalents (`Bun.file` over `node:fs`, `WebSocket`
  built-in, etc.)
- `node:os` (tmpdir/homedir), `node:fs/promises` (mkdir/appendFile) are fine — no
  Bun equivalents exist
- `Bun.write` has no permission mode option — use `writeFile` from
  `node:fs/promises` with `{ mode: 0o600 }` when writing files containing
  secrets
- `Bun.file().exists()` only works on regular files — use `fs.access()` for
  unix sockets and other special files
- Bun automatically loads `.env`, so don't use dotenv

## Testing

- Always trend towards 100% test coverage — every new function, branch, and
  edge case should have a corresponding test
- **Always run `mise test:live` yourself after protocol or
  daemon changes.** The live server is always available. Do not ask the user to
  run it — run it. If it fails for infrastructure reasons (server down, env vars
  missing), then defer to the user.
- Tests are colocated: `foo.ts` → `foo.test.ts` in the same directory
- Import from `bun:test`: `import { test, expect, describe } from "bun:test"`
- Run with `mise test`
- `mise test src/file.test.ts` runs a single file (args pass through to `bun test`)
- Use `jest.useFakeTimers()` / `advanceTimersByTime()` from `bun:test` for
  timer-dependent tests (wrap in `try/finally` with `jest.useRealTimers()`)
- Prefer promise-based waiting over `Bun.sleep()` — await the event, not a
  hardcoded delay
- Use `./tmp/` for scratch files, not `/tmp/` (gitignored)
- `bun test` scans `./tmp/` for test files — never leave `.test.ts` files there
- macOS `tmpdir()` returns `/var/folders/.../T/`, not `/tmp/` — don't hardcode
  `/tmp/` paths
- Live tests read `WOW_LANGUAGE` env var (default: 1/Orcish for Horde accounts)
- `Bun.connect()` returns a promise — connection errors escape `new Promise`
  constructors. Chain `.catch(reject)` on the returned promise, not try/catch
- `Bun.listen` server-side `socket.end()` doesn't reliably trigger client
  `close` — detect the protocol terminator in `data` handler instead
- Use unique socket paths per test (counter + timestamp) to avoid cleanup races
- `mock.module()` leaks across test files in Bun — only mock `"paths"` (safe via
  dynamic imports), never mock `"config"` or `"session-log"` in shared test runs.
  For stdlib modules like `node:readline`, use dependency injection instead
- `Bun.sleep(0)` yields one microtask tick (enough for `.then()` chains);
  `Bun.sleep(1)` yields one full event loop turn (needed for filesystem I/O like
  `unlink` to complete) — prefer the minimum needed in tests
- `bun test` suppresses per-test lines when piped — use `mise test:slowest` or
  `--reporter=junit --reporter-outfile=<file>` for timing data
- Bun counts implicit class constructors (from field initializers) and
  `.catch(() => {})` callbacks as separate functions — use explicit
  constructors and test rejection paths to reach 100% function coverage

## Mise Task Authoring

- Use `'''` (TOML literal strings) for tasks with regex/backslashes — `"""`
  processes escapes and breaks sed/awk patterns

## Plans and Design Docs

- Design docs and implementation plans live in
  `docs/plans/YYYY-MM-DD-<topic>-{design,plan}.md`
- Use the `/entire` skill and the `entire` CLI to investigate why past changes
  were made — it recovers full agent session context (prompts, decisions,
  trade-offs) from checkpoints linked to commits

## Reference Codebases

- `../wow-chat-client` — Node.js WoW chat client, primary protocol reference
- `../azerothcore-wotlk-playerbots` — AzerothCore server source (C++). Key files:
  `src/server/game/Entities/Object/Updates/UpdateFields.h` (complete field index
  for 3.3.5a build 12340), `src/server/game/Handlers/SpellHandler.cpp`
  (CMSG_CAST_SPELL handling), `src/server/game/Handlers/GroupHandler.cpp`
  (SMSG_PARTY_MEMBER_STATS construction)
- `../wowser` — browser-based WoW 3.3.5a client (ES2015/React/WebGL). Useful for
  cross-referencing opcodes, auth error codes, and realm parsing. Key files:
  `src/lib/auth/` (challenge opcodes, reconnect), `src/lib/game/opcode.js` (40+
  world opcodes), `src/lib/realms/handler.js` (realm list parsing),
  `src/lib/crypto/srp.js` (SRP-6 reference — uses insecure Math.random, ours is
  better)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tvararu/tuicraft](https://github.com/tvararu/tuicraft) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
