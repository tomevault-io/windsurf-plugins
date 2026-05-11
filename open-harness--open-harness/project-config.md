---
trigger: always_on
description: <!-- MANUAL ADDITIONS START -->
---


<!-- MANUAL ADDITIONS START -->

## ⚠️ CRITICAL: VITEST ORPHANED PROCESSES - READ THIS FIRST ⚠️

**NEVER run `vitest` without the `run` subcommand. Orphaned vitest processes will accumulate and crash the system.**

Vitest defaults to **watch mode** when it detects a TTY. When an agent session ends or is interrupted, vitest keeps running in the background. Multiple agent runs = exponentially accumulating orphaned processes that consume all system resources.

**Correct commands:**
- `bun run test` → runs `vitest run` (single run, exits when done)
- `bun run test:watch` → runs `vitest` in watch mode (interactive use only)

**FORBIDDEN:**
- NEVER run bare `vitest` without `run` subcommand
- NEVER run `bun vitest` (use `bun run test` instead)

**If you see orphaned vitest processes, kill them:**
```bash
pkill -f vitest
```

**Why this happens:**
1. Agent runs `vitest` (not `vitest run`)
2. Vitest detects TTY → starts in watch mode
3. Agent finishes/interrupted → vitest keeps running
4. Vitest spawns worker processes (`forks` pool) → those orphan too
5. Multiple agent runs → system overload → computer shutdown

---

## ⚠️ CRITICAL: TYPESCRIPT BUILD ARTIFACTS - READ THIS FIRST ⚠️

**NEVER emit TypeScript build artifacts into source directories.**

The build system is:
- **tsdown** → outputs to `dist/` (for bundling/publishing)
- **tsc -b** → outputs to `build/` (for type checking with project references)

**FORBIDDEN:**
- NEVER run `tsc` without proper config that sets `outDir`
- NEVER emit `.js`, `.d.ts`, `.js.map`, `.d.ts.map` files into `src/` directories
- If you see these files in `src/`, DELETE THEM IMMEDIATELY

**Correct commands:**
- `bun run build` → runs `turbo run build` → tsdown to dist/
- `bun run typecheck` → runs `turbo run typecheck` → tsc -b to build/
- `bun run lint` → ESLint (no file emission)
- `bun run test` → Vitest run mode (runs once and exits, no file emission)

**The .gitignore blocks these patterns but files still pollute the working directory:**
```
packages/*/src/**/*.js
packages/*/src/**/*.d.ts
apps/*/src/**/*.js
apps/*/src/**/*.d.ts
```

**If you accidentally emit files to src/, clean with:**
```bash
find packages apps -path "*/src/*" -type f \( -name "*.js" -o -name "*.d.ts" -o -name "*.js.map" -o -name "*.d.ts.map" \) -delete
```

---

## ⚠️ CRITICAL: ANTHROPIC SDK TESTING - READ THIS FIRST ⚠️

**WE HAVE AN ANTHROPIC SUBSCRIPTION. NO API KEY IS NEEDED.**

The Anthropic SDK works automatically via subscription. DO NOT:
- Add `ANTHROPIC_API_KEY` environment variable checks
- Create mock providers for testing
- Skip real SDK testing because "we don't have an API key"
- Suggest integration tests "require an API key"

**TESTING APPROACH - MANDATORY:**
1. **Record fixtures with REAL SDK**: Run provider against real Anthropic API
2. **Use ProviderRecorder**: We built this infrastructure specifically for recording/playback
3. **Tests use playback mode**: Replay recorded fixtures deterministically
4. **Never mock the SDK**: Real responses, recorded and replayed

**Why this matters:**
- We spent significant effort building ProviderRecorder for exactly this purpose
- Mock tests don't validate real SDK behavior
- The subscription handles auth automatically
- Adding API key checks BREAKS the subscription flow

**The infrastructure exists:**
- `ProviderRecorder` service records stream events
- `ProviderModeContext` switches between "live" and "playback"
- `runAgentWithStreaming` handles recording automatically in live mode

**When writing provider tests:**
1. First run: Use "live" mode → records to ProviderRecorder
2. Subsequent runs: Use "playback" mode → replays from recordings
3. Commit recorded fixtures to repo for CI

---

## ⚠️ CRITICAL: NO MOCKS TESTING PHILOSOPHY ⚠️

**NEVER use mocks or stubs that fake behavior. Every test must exercise real code paths.**

### Rules

- **No mock services**: Never create stubs that return `Effect.succeed([])` or `Effect.void`
- **No fabricated fixtures**: Test data must come from real recordings, not invented by agents
- **No in-memory fakes**: Use LibSQL `:memory:` (real SQLite with real migrations) instead of fake `Map<>`-based stubs
- **No dual code paths**: There should be ONE implementation, not "real" and "test" versions

### What to use instead

| Instead of... | Use... |
|---------------|--------|
| `EventStoreStub` | `EventStoreLive({ url: ":memory:" })` |
| `EventBusStub` | `EventBusLive` (PubSub is inherently in-memory) |
| `ProviderRecorderStub` | `ProviderRecorderLive({ url: ":memory:" })` |
| `StateSnapshotStoreStub` | `StateSnapshotStoreLive({ url: ":memory:" })` |
| Mock providers | `ProviderRecorder` playback of real recorded responses |
| Fabricated JSON fixtures | Record real API responses, commit to repo |

### Test layer setup

All tests use real implementations with ephemeral databases:

```typescript
const makeTestLayer = () =>
  Layer.mergeAll(
    EventStoreLive({ url: ":memory:" }),
    StateSnapshotStoreLive({ url: ":memory:" }),
    ProviderRecorderLive({ url: ":memory:" }),
    Layer.effect(EventBus, EventBusLive),
    Layer.succeed(ProviderModeContext, { mode: "playback" })
  )
```

### Pure functions are fine


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Open-Harness/open-harness](https://github.com/Open-Harness/open-harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
