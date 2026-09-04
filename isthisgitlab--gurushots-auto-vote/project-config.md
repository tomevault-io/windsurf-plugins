---
trigger: always_on
description: - **MCP Integration**: Always utilize MCP (Model Context Protocol) tools and resources first before implementing custom logic. Check available MCP servers for existing solutions.
---

# Development Guidelines

## Code Architecture

- **MCP Integration**: Always utilize MCP (Model Context Protocol) tools and resources first before implementing custom logic. Check available MCP servers for existing solutions.
- **Cross-Platform Consistency**: The application targets three platforms — Electron (GUI), CLI, and Android (Capacitor). All core business logic in `src/js/` must be shared; only the platform shell (entry points, transport, storage adapter) is platform-specific.
- **Entry points**: Electron `src/js/index.js` · CLI `src/js/cli/cli.js` · Electron preload `src/js/preload.js` · Capacitor bridge `src/js/bridge/capacitor.js`. The same React renderer (`src/js/react/`) runs under both Electron and Capacitor.
- **API surface swap**: `src/js/apiFactory.js` selects real vs mock implementations at runtime based on `settings.mock`. All callers go through the factory — do not import `src/js/api/*` or `src/js/mock/*` directly from business logic.
- **IPC** (deep reference: `docs/architecture.md`): `src/js/ipc/manifest.js` is the dependency-free single source of truth for the `window.api` surface; both shells generate from it and CI enforces name-parity (**signatures are NOT checked**). Handlers export `buildHandlers(deps)` + `register(ipcMain)`, are reused by CLI and Capacitor (no parallel impls), and **return `{success,error}` — never throw to the renderer**.
- **Persistence**: use `createJsonStore()` (`src/js/settings/storage.js`) for platform-aware JSON; don't hand-roll `fs`. Node-side platform detection = `src/js/runtime.js`; renderer-side = `globalThis.Capacitor?.isNativePlatform?.()`.

## Application Domain & Invariants

> Deep reference: **`docs/architecture.md`** (and `docs/scheduling.md` for the timer engines). These are invariants an edit must not violate.

- **Voting pass**: one shared loop `runVotingPass` (`src/js/services/votingOrchestrator.js`) for real+mock — never fork it; inject differences via `deps`. The per-challenge runners are **sequential, never parallelised** (shared challenge-object mutation). Rule precedence in `_runVotingRules` (`src/js/services/VotingLogic.js`) is load-bearing.
- **Two distinct sentinels — don't merge them**: for `exposureTarget`/`lastHourExposureTarget`, `0`/null = "target == trigger" (rule **stays active**); for `boostTime`/`emergencyFill`/`keyUnlockedBoostTime`, `0` = feature **off**.
- **Scheduler**: single recursive `setTimeout` chain, no cron; one decision point `computeNextCycleDelayMs`; invariant "never sleep past a boundary"; cancellation is a global flag that propagates by `return`, never `throw`.
- **API transport**: everything POSTs through `makePostRequest` (`src/js/api/api-client.js`), which returns the body or **`null` — it never throws**. Branch on `null`; keep retry/backoff centralised; custom (Android) adapters must call `finalizeAdapterResponse`.
- **Semantic lexicon** ranks auto-fill photos only — **never** the vote decision; `SEMANTIC_MATCH_FLOOR` is build-gated by `scripts/validate-lexicon.js`, not hand-tuned.
- **Safety**: optional-chain every per-challenge API read (an unguarded throw aborts the whole pass); new-entry detection compares ids as **sets, not positions**; mock mode must pass `cleanupStaleMetadata: null` so it never touches the real `metadata.json`.
- **Security invariants** (details/limits in `docs/architecture.md` §10): keep `contextIsolation` on, `nodeIntegration` off, and the sandboxed-preload / `window.api`-only exposure — regressing any is a severe-vuln class; keep the `isTrustedSender` frame check on every invoke; never log a raw headers/token object (redaction is key-allowlisted, not exhaustive — it misses the `x-token` key).

## File Management

- **Documentation Policy**: Do not create markdown files, README files, or other documentation unless explicitly requested by the user.
- **Prefer Editing**: Always edit existing files rather than creating new ones when possible.

## Testing Standards

- **Test Organization**: All test files must be placed in the `tests/` directory following proper Jest conventions and structure.
- **Mock Configuration**: Never use `mock: false` in any Jest or testing commands - use proper mocking strategies instead.

## UI/UX Standards

- **Styling Framework**: GUI application uses Tailwind CSS + DaisyUI for all styling components
- **Custom CSS Policy**: No custom CSS is allowed except for the predefined Latvian color theme (already defined in the project)
- **Component Consistency**: Follow DaisyUI component patterns and conventions for consistent user experience

## Renderer / UI

> Scope: `## UI/UX Standards` above governs styling policy (Tailwind/DaisyUI/theme); this section governs component & data-flow conventions. Deep reference: `docs/architecture.md` §8–9.

- **Backend access**: all backend calls go through `window.api.*` via `src/js/react/api/useIpcQuery.js` / `useAsyncIpcAction.js` — **no per-platform branching in components**, and don't call `window.api` raw.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [isthisgitlab/gurushots-auto-vote](https://github.com/isthisgitlab/gurushots-auto-vote) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-04 -->
