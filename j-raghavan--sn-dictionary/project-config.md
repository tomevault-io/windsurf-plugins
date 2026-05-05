---
trigger: always_on
description: You are reviewing code for an open source project.
---

# Repository instructions for GitHub Copilot

You are reviewing code for an open source project.

## Review priorities
Focus on:
- correctness and edge cases
- security issues and unsafe defaults
- backward compatibility
- performance regressions in hot paths
- unnecessary complexity
- missing tests for changed behavior
- API and schema stability
- maintainability and readability

## What to avoid
Do not suggest:
- large refactors unless clearly necessary
- stylistic churn without concrete benefit
- adding dependencies unless justified
- speculative changes not grounded in the diff

## Project expectations
Prefer:
- small targeted fixes
- explicit error handling
- clear naming
- simple designs over clever abstractions
- preserving existing public behavior unless the PR explicitly changes it

## Testing expectations
Flag when:
- behavior changes without tests
- edge cases are untested
- error paths are untested
- docs should be updated because user-facing behavior changed

## Review style
Be concise and specific.
Reference the exact risk and the likely impact.
When possible, suggest a minimal fix.

---

## Engineering principles

This project enforces a principal-engineer quality bar. Apply these consistently when reviewing.

### SOLID
- **Single Responsibility** — every module / component / function should have one clear reason to change. Flag handlers that grow side concerns (logging policy, state persistence, UI decisions) into the same unit.
- **Open / Closed** — prefer extension via new modules, configuration, or strategy injection over editing core logic. Flag conditional branches that pile up to cover new cases when polymorphism or a registry would scale better.
- **Liskov Substitution** — when a function takes a `*Like` interface (the project's DI pattern, e.g. `CommAPILike`, `PluginManagerLike`), every implementation must honour the contract. Flag mocks or test doubles that diverge in observable behavior from production callers.
- **Interface Segregation** — narrow `*Like` interfaces to the methods the consumer actually calls. Flag deps objects that pull in API surface the handler never invokes.
- **Dependency Inversion** — handlers, the StarDict reader, and the popup all depend on abstractions (interfaces / event buses), not on concrete `sn-plugin-lib` symbols. Flag direct turbomodule imports inside handler / pure modules — those belong only in `index.js`.

### KISS
- Pick the simplest design that satisfies the requirement.
- A 30-line handler with one clear branch is better than a 10-line handler that needs a comment to explain.
- Flag clever one-liners, dynamic dispatch, or meta-programming when a straightforward function suffices.

### DRY
- Extract a helper when the same logic appears 3+ times *and* the abstraction names the concept naturally. Two near-duplicates are usually fine; three is the rule-of-three threshold.
- Flag copy-pasted SDK call sequences, copy-pasted try/catch blocks around the same fallback, and copy-pasted log prefixes.
- Reuse existing helpers (`unwrap`, `safeClosePluginView`, `resolveIconUri`, `tryAcquire/release`, `t()`, `decodeBase64`, `decodeUtf8`) rather than re-rolling them inline.

### DDD-flavored boundaries
The codebase is small but follows clear ubiquitous-language boundaries — keep them.
- `src/buttons/` — registration with the firmware.
- `src/handlers/` — entry-gesture pipelines (orchestration only; no SDK turbomodule imports, no React).
- `src/core/dict/` — the StarDict reader and lookup contract; pure functions, no React, no SDK.
- `src/core/dict/stardict/` — file-format parsers (`.ifo`, `.idx`, `.dict.dz`); each module owns one responsibility.
- `src/sdk/` — narrow utilities that bridge platform quirks (`utf8`, `base64`, `unwrap`, `closeView`, `types`).
- `src/i18n/` — locale resolution and string tables.
- `src/ui/` — React Native components and the event-bus that connects them to async handlers.

Flag changes that **leak across these boundaries** — e.g. a handler reaching into popup state, a parser importing React, a button-registration module pulling in dictionary types.

---

## Plugin-specific gotchas (do not regress)

These bit me on-device and are documented in PR notes / commit messages. Flag any change that breaks them.

### Reentrancy guard must clear synchronously before any await
`src/core/reentrancyGuard.ts` is module-level. Handlers must call `release()` **before** awaiting `closePluginView` in the `finally` block. Clearing it after the await leaves the flag stuck `true` if the host's `state:stop` transition suspends the JS context — every subsequent button press is then rejected as busy.

### Handler does NOT close the firmware overlay on the success path
When a popup is rendered, the firmware overlay must stay open until the user dismisses the popup. Closing it from the handler's `finally` while the popup is still on-screen orphans the overlay and the device hangs on subsequent pen taps. The popup's Close button calls `PluginManager.closePluginView()` directly, fire-and-forget. The handler's `finally` only closes the view on early-exit paths (busy / empty / failed) — track this with a `popupShown` flag.

Flag any PR that:
- Calls `closePluginView` unconditionally in a handler's `finally`.
- Removes the `popupShown` guard.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [j-raghavan/sn-dictionary](https://github.com/j-raghavan/sn-dictionary) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
