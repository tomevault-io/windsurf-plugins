---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run build          # Compile TypeScript → dist/
npm test               # Unit tests only (~180 tests, no SAP needed, <3s)
npm run test:coverage  # Unit tests with coverage report
npm run test:live      # Integration tests (needs SAP env vars)
npm run test:e2e       # End-to-end write lifecycle (create → write → activate → delete)
npm run dev            # Launch MCP Inspector for interactive tool testing
```

Run a single test file:
```bash
npx jest src/__tests__/unit/urlBuilder.test.ts
npx jest src/__tests__/unit/errors.test.ts
```

After any source change: `npm run build` — the MCP server runs from `dist/`, not `src/`.

## Architecture

### Request Flow

```
index.ts (AbapAdtServer)
  → handler.validateAndHandle(toolName, args)   ← centralized required-field check
    → handler.handle(toolName, args)             ← switch to concrete method
      → this.withSession(() => adtclient.xxx())  ← auto-reconnect wrapper
```

`index.ts` registers all handlers, wires elicitation/notify/sampling callbacks, and dispatches every `tools/call` to the correct handler's `validateAndHandle`. No tool routing logic lives outside `index.ts`.

### Handler Hierarchy

All 8 handlers extend `BaseHandler` (`src/handlers/BaseHandler.ts`), which provides:
- `withSession(fn)` — wraps every ADT call; detects session expiry (401, ambiguous 400) and re-logins transparently
- `validateAndHandle(toolName, args)` — checks JSON schema `required` fields before any handler code runs
- `elicitForm / elicitChoice / confirmWithUser` — MCP elicitation forms injected from `index.ts`
- `notify(msg, level)` — progress messages visible in Claude Code's UI
- `askClaude(system, user)` — sampling to ask Claude a question without interrupting the user
- `fail(msg)` — throws McpError; never returns

### Lock → Write → Unlock Pattern

**Critical invariant**: lock, write, and unlock for any object MUST happen inside a single `withSession(async () => { ... })` block — never in separate `withSession` calls.

Reason: if a session timeout fires between calls, `withSession` re-logins and retries only the one operation it wraps. A lock handle acquired in session A is invalid in session B.

The canonical pattern (from `handleSetSource`):
```typescript
const doWrite = async (): Promise<void> => {
  const r = await this.adtclient.lock(objectUrl);
  lockHandle = r.LOCK_HANDLE;
  try {
    await this.adtclient.setObjectSource(sourceUrl, source, lockHandle!, transport);
  } catch (err) {
    try { await this.adtclient.unLock(objectUrl, lockHandle!); } catch (_) {}
    lockHandle = null;
    // If unLock also failed (dead session), sleep before rethrowing so SAP's
    // session cleanup can release the orphaned enqueue entry before withSession retries.
    if (writeWasDeadSession) await new Promise(r => setTimeout(r, 3000));
    throw err;
  }
  await this.adtclient.unLock(objectUrl, lockHandle!);
  lockHandle = null;
};
await this.withSession(doWrite);
```

`handleSetSource` and `handleSetClassInclude` also retry up to 3 times (0s / 3s / 8s) on "locked by another" errors with `notify()` progress messages.

### Error Classification Pipeline

`src/lib/errors.ts`:
- `parseAdtError(error)` — classifies SAP errors into `AdtErrorInfo` fields: `isSessionTimeout`, `isLocked`, `isNotFound`, `isUpgradeMode`, `isAmbiguous400`
- `isAmbiguous400` — HTTP 400 with no meaningful body; treated as session expiry (stale CSRF token). `withSession` detects this and re-logins automatically.
- `formatError(operation, error)` — converts classified errors into actionable human-readable messages with self-correction hints (what to call next, why it failed)

**`AdtErrorException` field layout:** The library stores the HTTP status in `.err` (not `.response.status` — `.response` is often `undefined`). `parseAdtError` reads `error?.response?.status ?? error?.err` so both shapes are covered. Do not rely on `error?.response?.status` alone.

**The `.err = 500` wrapper lie:** `fromError`/`fromException` in abap-adt-api wrap errors they don't recognize as `AdtErrorException(500, …)` — a hardcoded 500 with the real HTTP status surviving only in the stringified message (`Error: Request failed with status code 400`). This was the dominant prod failure mode (~350 errors/10 days logged as session_timeout with no retry). `parseAdtError` now trusts the status parsed from the message when the numeric status is that wrapper's 500 and there is no `.response`. The real SAP response (status/headers/body) is captured at the axios layer via an interceptor (`installRawFailureCapture` in `index.ts`) and merged into the error log as `raw_url`/`raw_status`/`raw_headers`/`raw_body`.

When adding a new error condition, update `parseAdtError` first (adds detection), then `formatError` (adds the message).

### URL Construction

`src/lib/urlBuilder.ts` is the single source of truth for all ADT paths. `TYPE_PATHS` maps ABAP type strings to their ADT base paths. `buildObjectUrl(name, type)` is for lock/unlock/delete; `buildSourceUrl(name, type)` appends `/source/main` for read/write.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DassianInc/dassian-adt](https://github.com/DassianInc/dassian-adt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
