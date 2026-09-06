---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
pnpm run build          # compile TypeScript -> dist/ (src only)
pnpm run typecheck      # type-check src + ALL tests, no emit (tsconfig.typecheck.json)
pnpm test               # runs `typecheck` first, then unit tests, no credentials needed
pnpm run test:contract  # deterministic MCP/schema/workflow contracts
pnpm run test:protocol  # deterministic modern + legacy MCP protocol behavior
pnpm run evals          # build + eval harness; provider cases gate on RUN_LLM_EVALS=1 + provider key
pnpm run test:integration:live  # live tests, requires real B2 credentials in env
pnpm run start          # stdio transport (local Claude Desktop use)
pnpm run start:http     # Streamable HTTP transport, add --port 3000
pnpm run docs           # strict TypeDoc API reference -> gitignored api-docs/
pnpm run docs:watch     # TypeDoc watch mode with the same strict validation
```

> `pnpm run build` uses `tsconfig.json`, which **excludes `tests/`** — so it does
> not catch compile errors in test files. `pnpm run typecheck` (wired into `pnpm test`)
> compiles `src` **and** `tests` via `tsconfig.typecheck.json`, so live-test
> compile errors are caught with no credentials. This closed a real gap where a
> broken live-test reference only surfaced on a credentialed run.

Run a single unit test file:

```bash
pnpm exec vitest run --config vitest.config.mts --project=unit tests/unit/auth.unit.test.ts
```

Run a single test by name:

```bash
pnpm exec vitest run --config vitest.config.mts --project=unit --testNamePattern="should cache the token"
```

Eval harness tests:

```bash
pnpm run evals
```

The deterministic harness self-tests run without provider credentials. Provider
adapters should gate real LLM-backed eval cases on `RUN_LLM_EVALS=1` plus their
own provider key env var (for example `OPENAI_API_KEY` or `ANTHROPIC_API_KEY`)
so normal CI remains key-free.

TypeDoc is strict. `typedoc.json` enumerates the public `src` entry-point
surface, writes generated HTML to gitignored `api-docs/`, requires module,
class, interface, function, method, property, enum, type alias, variable,
accessor, constructor, and signature documentation, and treats warnings as
errors. Undocumented public API members, undocumented modules, and invalid
links fail `pnpm run docs` and the docs workflow; issue #308 closed this
strict-validation ratchet.

Integration tests require env vars. The live suite exercises native, S3, key
management, event notifications, and the Partner API with no opt-in skips, so it
needs the full set: a non-master application key (`B2_APPLICATION_KEY_ID` /
`B2_APPLICATION_KEY`) for native, S3, and key management; the account master key
(`B2_MASTER_KEY_ID` / `B2_MASTER_KEY`) for the Partner API (which rejects
non-master keys); `B2_REGION` for the account's S3 region; and
`B2_LIVE_NOTIFICATION_BUCKET` naming a pre-provisioned, notifications-enabled
bucket:

```bash
B2_APPLICATION_KEY_ID=appkey_id B2_APPLICATION_KEY=appkey_secret \
B2_MASTER_KEY_ID=master_id B2_MASTER_KEY=master_secret \
B2_REGION=us-east-005 B2_LIVE_NOTIFICATION_BUCKET=your-notify-bucket \
pnpm run test:integration:live
```

## Commit conventions

- Commit messages are a **single line, under 72 characters**, prefixed with a
  Conventional Commit type: `feat:`, `fix:`, `docs:`, `refactor:`, `test:`,
  `chore:`, `ci:`, `build:`, `perf:`. No body, no bullet list.
- **Never** add a `Co-Authored-By` trailer or any AI/assistant/"Generated with"
  attribution to commits or PR bodies.

## Recent behavior notes (v0.1.2)

- The outbound User-Agent product token is `b2-mcp/<version>` on a published release and `b2-mcp/dev` for source/CI/dev builds, on both the native B2 SDK and the AWS S3 SDK. The build channel is resolved in `src/version.ts` (`productVersion()` / `productToken()`, `PRODUCT_NAME`) via a publish-time `release-version.json` marker; plain `VERSION` keeps the numeric semver for the MCP handshake, `--version`, and logs. `B2_MCP_UA_SUFFIX` still appends a deployment tag. (Server self-identity and the log service name intentionally remain `backblaze-b2-mcp`.)
- S3-compatible and report tools derive their endpoint/signing region from the authorized `b2_authorize_account` `s3ApiUrl`; `B2_REGION` is only a fallback/default before authorization or when authorize is temporarily unavailable.
- The native analytics tools and `b2_list_buckets` resolve a bucket-scoped key through its authorized `allowedBuckets` scope instead of an unfiltered `listBuckets()` (which a bucket-scoped key cannot call), and reject/report out-of-scope input without enumerating the key's bucket namespace.
- Destructive-gate refusals are stable, non-500 tool outcomes: `destructive_policy_blocked` (HTTP 403), and `destructive_confirmation_required` / `destructive_confirmation_refused` (HTTP 409); `tool.call` audit logs record those codes/statuses, not `internal_error`/500.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [backblaze-labs/b2-mcp](https://github.com/backblaze-labs/b2-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
