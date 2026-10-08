---
trigger: always_on
description: `@miolamio/tg-cli` — user-account Telegram CLI (MTProto / gramjs). Binary: `tg`. JSON-first, agent-scriptable. Node >= 20, ESM only.
---

# AGENTS.md

`@miolamio/tg-cli` — user-account Telegram CLI (MTProto / gramjs). Binary: `tg`. JSON-first, agent-scriptable. Node >= 20, ESM only.

## Stack

- TypeScript 5.9, strict, `module`/`moduleResolution`: Node16
- Commander 14, `telegram` (gramjs), `conf`, `proper-lockfile`, `zod`, `@toon-format/toon`, `picocolors`
- tsup (ESM, Node 20 target), vitest (globals)
- No ESLint / Prettier

## Commands

```bash
npm run build                                 # dist/bin/tg.js + dist/lib/daemon/entry.js + dist/index.js
npm run dev                                   # tsup --watch
npm test                                      # vitest run (tests/**/*.test.ts)
npx vitest run tests/unit/<file>.test.ts      # one file
npm run test:coverage
npm run typecheck                             # tsc --noEmit
```

`tests/integration/` rebuilds the binary before asserting `--help` / `--version`.

## Feature cards and user-story acceptance (TG-55)

Every change (feature, bugfix, docs, tooling, or release) must have a Lific `TG-*`
card before implementation. Search existing cards first. Record the user's
goal, Given/When/Then scenarios, verification method, and expected result.
Use [tests/stories/README.md](tests/stories/README.md) and the scenario catalog.

- Map each automated scenario to actual test names; run `npm run test:stories`.
  Missing, failed, and skipped tests are not passes. Add a regression that
  reproduces the reported user-visible failure before claiming it fixed.
- Record revision/version, environment, command, result, and sanitized evidence
  in the card. Distinguish mocks, real processes/TTY, live Telegram, and the
  affected user's machine. A handshake or ping/pong is not a completed login;
  subscribe acknowledgement is not proof of event delivery.
- Keep the card active while any required scenario is failed or unverified.
  Do not silently remove an acceptance criterion to close a card. Preserve
  historical evidence with its original scope and date.
- Before publishing, run `npm run test:acceptance` and complete the manual
  acceptance record for the candidate. Also require typecheck, build, package
  install checks, and actually executed CI. Publishing a package does not
  prove its feature works. Report any explicit user-authorized exception in
  the card and release notes; do not infer one from a routine publish request.
- Live checks use only authorized accounts/actions. Never store phone numbers,
  codes, passwords, API hashes, session strings, or private message contents in
  acceptance artifacts. A manual check may remain pending without inventing
  evidence or asking to expose credentials in chat.

The catalog covers the recent daemon, transport, login, branding, and release
work. Older review cards keep their historical evidence; extend their scenarios
when touching that behavior. Test counts alone never establish user acceptance.

## Layout

| Path | Role |
|---|---|
| `src/bin/tg.ts` | CLI entry, global flags, `preAction`, command groups |
| `src/commands/<domain>/` | `index.ts` (Commander group) + one file per subcommand action |
| `src/lib/` | client, session, output, serialize, peer, errors, daemon |
| `src/lib/types.ts` | all output DTOs |
| `src/index.ts` | library re-exports |
| `tests/unit/` | one test file per command/lib; mocks gramjs, never live Telegram |

Domains: `auth`, `session`, `chat`, `message`, `media`, `user`, `contact`, `daemon`, `completion`.

## New command

1. `export async function fooAction(this: Command, ...args): Promise<void>`
2. `const opts = this.optsWithGlobals() as GlobalOptions & { ... }`
3. Authenticated work: `withAuth(opts, async (client) => { ... })` — except `auth/*` and `session/*` (custom flows)
4. Peers via `resolveEntity`; gramjs objects via `serialize*` / `bigIntToString`
5. Result: `outputSuccess(dto)` or `outputError(message, ErrorCode.X)`
6. Register in domain `index.ts`; a new group also in `src/bin/tg.ts` + `helpGroup(...)`
7. DTO in `types.ts`; human renderer in `format.ts` if the shape is new; code in `error-codes.ts`
8. Unit test using the existing mock set: `telegram`, `output`, `SessionStore`, `withClient`/`withAuth`, `peer`

ESM imports keep `.js` extensions. Match neighboring JSDoc.

## Conventions

- **stdout is data.** `outputSuccess` / `outputError` only. Status goes to stderr via `logStatus`.
- **JSON default:** `{ ok: true, data }` / `{ ok: false, error, code }`. `--jsonl` = one object per line, no envelope (lists). `--toon` wraps the envelope. `--fields` is dot-notation. `--toon`/`--jsonl`/`--human` are mutually exclusive (enforced in `preAction`).
- **IDs are strings** (`bigIntToString` — gramjs BigInteger JSON-serializes to `{}`). Dates are ISO 8601.
- **Client lifecycle:** `withClient` / `withAuth`. Cleanup is `client.destroy()`, never `disconnect()` (zombie `_updateLoop`). Default timeout 120s.
- **Session:** `SessionStore.withLock(profile, ...)` for the whole operation. Files: `<configDir>/sessions/<profile>.session`.
- **Credentials:** `TG_API_ID` + `TG_API_HASH` beat config. Presets: `tg auth login --client desktop` (see `presets.ts`). Profile names: `[a-zA-Z0-9_-]{1,64}` (`validateProfile`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [miolamio/tg-cli](https://github.com/miolamio/tg-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
