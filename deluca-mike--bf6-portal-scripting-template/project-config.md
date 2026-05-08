---
trigger: always_on
description: Project-specific rules for building Portal mods. These apply when editing code in this repo.
---

# Portal scripting template — Agent rules

Project-specific rules for building Portal mods. These apply when editing code in this repo.

---

## Code quality & tooling

- **Lint**: Run the linter. Code cannot be executed outside of a locked-down game server, so lint is a primary quality check.
- **No any type**: Never use explicit or implicit any types unless absolutely necessary.

---

## Logging & debugging

- **Prefer Logger over `console.log`**: Only local game servers on PC dump `console.log` logs to a file, but behavior differs from production and some user can only test on Xbox or PS5. Use the Logger module for consistent, environment-aware logging.
- **Debugging is slow**: Seeing `setLogging` output requires building, uploading, and running the experience—not a quick loop. Don’t lean on runtime logging as the first path to learning about and fixing issues; prefer lint, types, and reasoning first.
- **Logger performance**: For long messages or performance-critical paths (e.g. dynamic logger with many rows), use `logAsync()` instead of `log()` so the operation is non-blocking and doesn’t cause frame drops.
- **Conditional logging**: When using the `Logging` class, use `willLog(level)` before building expensive log strings so you don’t build strings that won’t be emitted.
- **Utility logging**: Many `bf6-portal-utils` modules expose `setLogging(logFn, logLevel, includeError?)`. Use it when you need runtime visibility into handler/callback errors; remember it requires a full build/upload/run cycle.

---

## The `mod` namespace

- **Injected API**: `mod` is an injected namespace. Many functions use overloads rather than union/optional parameters.
- **Type reference**: Use `@node_modules/bf6-portal-mod-types/index.d.ts` (and referenced files) to see what’s available. Avoid reading files under `./runtime-spawn-enums` except `./runtime-spawn-enums/common.d.ts`, which lists spawnable assets available on all maps.
- **Opaque types**: Custom `mod` types (e.g. vectors, object refs) are opaque. Compare with `mod.Equals(a, b)`, `mod.GetObjId(a) == mod.GetObjId(b)`, or by decomposing (e.g. `mod.XComponentOf(vector)`).
- **User-facing strings**: The first argument to `mod.Message` can be: (1) a string key from `mod.stringkeys` (from `string.json` at runtime), with or without `"{}"` placeholders—use `mod.Message(stringKey, ...substitutions)` when the string has placeholders; (2) a runtime `number`; or (3) a `mod.Player`. For string keys, mirror the JSON structure (e.g. `mod.stringkeys.template.notifications.deployedOnMap`). Use nested key paths (e.g. `gameMode.hud.section.label`) in `string.json` and `mod.stringkeys` for organization.

---

## Architecture & patterns

- **Event-driven**: Prefer event-driven design over polling or long-lived task handlers.
- **Exit-early**: Use early returns to reduce nesting and improve readability.
- **Abstraction**: For complex logic or state, use namespaces or classes. Avoid relying on module or file scope for anything outside of the entry point file (`src/index.ts`), since the bundler flattens imports into a single TypeScript file (import order/hierarchy is preserved).
- **Per-player state**: For game modes with per-player state, use a class (or namespace) with a registry keyed by player id (e.g. `{ [playerId: number]: GamePlayer }`). Register on join, remove and clean up (UI, timers, spawned objects) when the player is invalid or leaves.
- **Constants in one place**: Keep game constants (points, thresholds, durations, sound config, message keys) as private static readonly or a dedicated config object; avoids magic numbers and makes tuning easier.
- **Event cleanup**: When subscribing to events, store the unsubscribe callback returned by `.subscribe()` and call it when the context is done (e.g. after handling the relevant occurrence). If a subscription is meant to stay open for the entire game, it is fine to never store or use the unsubscribe callback.
- **Optional chaining**: Use `?.` when accessing module-level or shared state that may not be initialized yet (e.g. admin-only tools that exist only after a specific player joins).
- **No duplicate event handlers**: Do not implement or export any Battlefield Portal event handler functions (e.g. `OnPlayerDeployed`, `OnPlayerJoinGame`) in your codebase. The `Events` module from `bf6-portal-utils` owns all event hooking; subscribe via `Events.OnPlayerDeployed.subscribe(...)` etc. instead.
- **Event handler order**: Execution order of multiple handlers for the same event is not guaranteed. If order matters, use a single handler that calls other functions in sequence, or chain manually. For example, `OnPlayerJoinGame` may fire before `OnGameModeStarted`.
- **Event handler return values**: Handlers cannot return values to the caller. Use shared state or callbacks if you need to collect results.
- **Race conditions**: When two handlers can run in any order (e.g. assist vs kill for the same victim), capture values that might change (e.g. “before death” state) as early as possible and write logic so both orderings are handled correctly.

---

## Dependencies & utilities

- **Prefer `bf6-portal-utils`**: Use modules from `bf6-portal-utils` instead of the raw `mod` namespace when possible.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [deluca-mike/bf6-portal-scripting-template](https://github.com/deluca-mike/bf6-portal-scripting-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
