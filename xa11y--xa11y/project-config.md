---
trigger: always_on
description: The goal is **full coverage** of the public API surface through real integration tests.
---

# Agent Guidelines

## Integration Test Coverage

The goal is **full coverage** of the public API surface through real integration tests.

When adding new tests:

1. If the AccessKit test app lacks a widget needed for a test, add it to the test app first. The test app uses AccessKit + winit and is defined in `test-apps/accesskit/src/main.rs`.
2. All integration tests must be `#[ignore]` and run via `cargo xtask test-integ`.
3. Run `cargo xtask test-integ` to verify tests pass before committing.

### Test helpers

Integration tests use shared helpers from `xa11y/tests/integ/mod.rs`:
- `h::app_tree()` — get test app root Element with retries
- `h::one(root, "selector")` — find exactly one element by selector
- `h::named(root, "Submit")` — find element by name substring
- `h::act(element, action)` — perform action, wait, re-read tree, return new root

### Key coverage gaps to address

- _(none currently)_ — Qt-on-macOS integration tests are now enabled in `.github/workflows/ci.yml` (`integ` matrix, `macos-latest × qt`), alongside the AccessKit, Cocoa, and Tauri macOS coverage.

## Design Tenets

1. **No silent fallbacks.** If an operation fails, return the error — don't silently try a different mechanism. Fallbacks hide bugs and make behavior unpredictable for consumers. Surface failures clearly so callers can handle them.

   **Anti-patterns that violate this tenet:**
   - `let _ = some_call();` — if the call's result matters, propagate it; if it genuinely doesn't, leave a one-line comment explaining why.
   - `some_call().ok()` used to coerce `Result → Option` and discard the error reason.
   - `if let Ok(x) = some_call() { ... } // else fall through` — this treats a real error as "no match". Match on the specific expected variant (e.g. `Err(Error::SelectorNotMatched)`) and propagate the rest.
   - Fallback chains: try A, on failure try B, on failure try C. Each step hides the original failure and changes effective behavior. If multiple mechanisms genuinely need to be tried, do it explicitly with logged reasoning, not silent fall-through.

2. **Only expose what accessibility APIs support.** If a platform has no accessibility interface for an operation, don't implement it with input simulation — leave it out.

3. **Action fidelity.** If an element reports an action name in its `actions` list, calling that action must invoke the original platform action — not a substitute or alias.

   `press`, `toggle`, `focus`, `select`, `expand`, `collapse` are *semantic verbs* — cross-platform concepts. Tenet 3 applies to the semantic verb, not a specific platform API name. For example: `press` on Windows legitimately dispatches to Invoke, Toggle, SelectionItem.Select, or ExpandCollapse based on the element's primary-activation pattern — this is the Windows canonical implementation of "activate this element," matching AXPress on macOS and AT-SPI `DoAction("click")` on Linux. A violation would be advertising `press` in actions but calling a platform API that doesn't implement the semantic (e.g. input simulation, or an unrelated pattern).

4. **Fail surfaceably, not fatally.** Prefer `Result` over `.unwrap()` / `.expect()` in provider and binding code.
   - **Locks**: `.lock().unwrap()` on caches or memoized state should be `.lock().unwrap_or_else(|e| e.into_inner())` — poisoning in a cache is recoverable. Only panic on locks that guard a genuine invariant.
   - **Platform FFI returns**: never `.unwrap()` a CF / AX / UIA / AT-SPI2 return. Propagate as `Error::Platform`.
   - **Tests** may use `.expect("...")` with a descriptive message when failure would indicate a broken test fixture.
   - If you add a new `.unwrap()`, a reviewer should be able to point at an invariant one line above that proves it can't panic.

5. **Blocking calls release the host runtime's lock.** In language bindings, any call that can block, sleep, or poll — waits, auto-waiting actions, attach/discovery loops, event receives — must release the host runtime's global lock (Python's GIL, or the platform equivalent) for the duration of the block. A wait that holds the GIL freezes every other thread in the consumer's process for up to the full timeout, and forces consumers into architectural workarounds (e.g. moving an in-process mock server into a separate process).

   **Anti-patterns that violate this tenet:**
   - A binding method that calls a core wait/poll loop directly instead of inside `py.allow_threads` (or the platform equivalent).
   - Holding the lock across a whole poll loop because one step needs it. The only legitimate reason to hold the lock is calling back into the host language (e.g. a Python predicate in `wait_until` / `App.find`) — reacquire it per callback, never for the loop.
   - Treating this as an optimization. A missing `allow_threads` on a blocking path is a correctness bug, not a style choice.

   Enforced for Python by `xa11y-python/tests/test_gil_release.py`, which asserts that a background thread keeps making progress while a native wait blocks.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xa11y/xa11y](https://github.com/xa11y/xa11y) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
