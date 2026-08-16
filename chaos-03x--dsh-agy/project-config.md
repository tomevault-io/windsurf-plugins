---
trigger: always_on
description: dsh-agy: DeepSeek Harness plugin + standalone CLI providing Google Antigravity (agy)
---

# AGENTS.md

dsh-agy: DeepSeek Harness plugin + standalone CLI providing Google Antigravity (agy)
OAuth, multi-account pool rotation, device fingerprinting, and the `agy` LLM provider.
To be published as an npm package. Product positioning is a grey-zone
tool (reusing the official client's quota machinery); Development uses pnpm (lockfile
`pnpm-lock.yaml`).

## Repository Layout

```
src/adapter/    DSH adapter: request translation (translate), SSE parsing (parse), model catalog (catalog/models)
src/oauth/      OAuth: authorize/PKCE, code exchange, refresh, paste-blob codec, endpoint constants
src/runtime/    State machine: 429 classification (classify), rotation decisions (rotation), fingerprints (fingerprint),
                session/request id (identity), thoughtSignature caching, version freshness (version)
src/store/      Account storage: encrypted JSON file (accounts, proper-lockfile), master key / keyring doc (keyring)
src/session.ts  Shared runtime glue: token caching, rotation execution, fingerprint lifecycle, verify/test/export
src/web/        /agy dashboard route + web plugin entry (only registered on loopback bindings)
src/cli/        Standalone CLI (login/status/import/verify/logout) + loopback callback server
tests/          Vitest test suite: fixture-driven, zero network
docs/           Architecture & API facts (EN + zh); maintenance-oriented, not bundled in npm package
scripts/        Developer tools (record:fixtures / e2e / debug / verify) requiring live accounts or network
```

The two plugin entry points (`src/index.ts` main plugin, `src/web/plugin.ts` web plugin)
share the same store/session/adapter instances via `createAgyRuntime` (`plugin-common.ts`).

## Non-negotiable Invariants (Violations are Regressions; all have test/code anchors)

- **Security (Loopback Trust Model)**:
  - The `/agy` route has no authentication and is ONLY allowed to register on loopback host bindings (`web/plugin.ts` gate).
  - OAuth exchanges MUST bind to the exact PKCE verifier issued for that authorization attempt (`pendingAuth` Map in web, local verifier in CLI); relaxing this verification is a security regression.
  - No request field or telemetry payload may ever transmit a raw refresh token — `sessionId` must be a derived identifier.
  - `~/.dsh/.credentials.yaml` can ONLY be modified via **append + atomic replacement** (`persistMasterKey`); rewriting the entire file wipes other credentials stored by DSH services.
  - Read-only CLI commands (`status`, `verify`, `logout`) must NEVER create a master key or credential document if one does not exist.
- **Classification Semantics**:
  - HTTP 403 responses containing quota / `RESOURCE_EXHAUSTED` phrasing MUST be classified as rate-limit (cooldown). Treating all 403s as auth-failures would permanently disable healthy accounts. Only true auth failures trigger account revocation, and a successful `verify` automatically re-enables the account.
- **Upstream Wire Facts** (`docs/ANTIGRAVITY-API.md`, verified empirically):
  - The endpoint fallback order `daily -> prod -> daily-sandbox -> autopush` is load-bearing.
  - HTTP 403 on the autopush endpoint for consumer accounts indicates "no license" (not credential failure).
  - `Client-Metadata` must only transmit `ideType`.
- **Version Freshness**:
  - The fingerprint User-Agent version is resolved dynamically via `version.ts` (750ms timeout ceiling + 6-hour cache + warm-up on boot).
  - `fingerprint-data.json` is compiled into the bundle; user hot-updates use the `$DSH_HOME/agy-fingerprint-data.json` override file.

## Commands

```sh
pnpm install                 # Development install; CI uses --frozen-lockfile
pnpm test                    # Vitest: fixture-driven, zero network
pnpm run typecheck           # tsc --noEmit
pnpm run build               # tsdown -> lib/ (three entrypoints: index / cli / web)
npm pack --dry-run           # Verify packaged files before release (npm ships with Node)
```

## Testing Conventions

- **Zero Network in Tests**: Any `fetch` call in unit tests MUST be stubbed (`vi.stubGlobal`). Real HTTP requests are only permitted in `scripts/` validation scripts. Any test touching the real network is invalid.
- `tests/*.test.ts` mirrors modules in `src/`. **Behavioral changes must update tests in lockstep** ("tests describe behavior, not correctness"). When adjusting classification, rotation, or fingerprint semantics, review what is pinned in runtime/session tests first.
- `tests/fixtures/recorded/` is gitignored. Re-recording uses `pnpm run record:fixtures` (requires a real account), and resulting diffs must be reviewed carefully.

## Scripts and CI/CD

- `scripts/` contains developer tools (`record:fixtures`, `e2e`, `debug:request`, `verify:tools`, `verify:blocks`), **all requiring real accounts or network access**. They are not part of routine dev loops, are not packaged into npm, and are not run in CI.
- `ci.yml` (runs on every PR and push to `main`): 3 OS (Ubuntu / Windows / macOS) × 2 Node versions (22 / 24; pnpm 11 requires 22.13+) -> pnpm install -> test -> typecheck -> build -> `npm pack --dry-run` -> **tarball smoke test** (installs into a clean temp directory and verifies CLI `--help`, `import('dsh-agy')`, and `import('dsh-agy/web')`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chaos-03x/dsh-agy](https://github.com/chaos-03x/dsh-agy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
