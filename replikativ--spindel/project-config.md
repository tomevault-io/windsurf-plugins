---
trigger: always_on
description: > **Audience**: contributors and AI assistants making changes to
---

# CLAUDE.md

> **Audience**: contributors and AI assistants making changes to
> Spindel. User docs live in [README.md](README.md) and
> [docs/](docs/) — `docs/concepts.md` for the mental model,
> `docs/engine.md` for the architectural deep-dive,
> `docs/scheduling.md` for the runtime dispatch model.
>
> This file is the contributor playbook: the rules that aren't worth
> putting in user docs but that will trip you (or an agent working
> on the engine) the first time. Read top-to-bottom before your
> first non-trivial change.

## Status

Beta. JVM: 772 tests / 2606 assertions; CLJS: 363 tests / 1374
assertions. Public API may evolve before 1.0.

For the architectural overview see
[`docs/engine.md`](docs/engine.md). For runtime dispatch / events /
executor / GC, see [`docs/scheduling.md`](docs/scheduling.md). For
the typed delta algebra, see [`docs/incremental.md`](docs/incremental.md).

---

## The Four CRITICAL Rules

These are the gotchas that have actually bitten people. Internalize
them; an agent that doesn't will produce subtly broken code that
passes type-check, sometimes passes a single test run, and fails
under load or in CLJS.

### Rule 1 — `(await x)` / `(track x)` inside spin bodies; never `@x`

Inside `(spin …)` bodies:

| Operation | Spin / Deferred | Signal |
|-----------|-----------------|--------|
| Correct   | `(await x)`     | `(track x)` |
| Wrong     | `@x`            | `@x` |

Outside spin bodies (REPL, tests), `@x` is fine.

**Why**: `@` (deref) blocks the calling thread. Inside a CPS-
transformed body that breaks the continuation chain — the spin
macro only inserts breakpoints at *registered effect* call sites,
which `await` / `track` are and `@` is not. On CLJS `@spin` simply
throws.

```clojure
;; ❌ blocks thread, breaks CPS, hangs on CLJS
(spin (let [result @some-spin] (process result)))

;; ✅ CPS-transformed
(spin (let [result (await some-spin)] (process result)))

;; ✅ track returns an Interval — read it with iv/get-new
(spin (let [iv (track some-signal)] (* 2 (iv/get-new iv))))
```

### Rule 2 — Effects don't survive into closures

The `spin` macro only CPS-transforms code it sees *lexically*.
Functions passed to `map` / `filter` / `reduce` and lazy-sequence
generators (`for`, `doseq`) hide their bodies from the macro, so
effects inside them stay as raw fn calls and blow up at runtime.

```clojure
;; ❌ closure body invisible to the macro
(spin (map #(await (fetch %)) items))

;; ❌ lazy-seq hides the awaits
(spin (for [x data] (await (fetch-spin x))))

;; ✅ explicit loop/recur — body is lexical
(spin
  (loop [remaining items, results []]
    (if (empty? remaining)
      results
      (recur (rest remaining)
             (conj results (await (fetch (first remaining))))))))

;; ✅ nest a (spin …) per item, splice with `parallel` for concurrency.
;;    Each (spin …) is its own CPS scope — the await inside the
;;    closure is lexical *within* that inner spin.
(require '[org.replikativ.spindel.spin.combinators :refer [parallel]])
(spin
  (let [child-spins (map (fn [x] (spin (await (fetch x)))) items)]
    (await (apply parallel child-spins))))
```

Same limitation as core.async's `go`. Three structural workarounds:
`loop`/`recur` for sequential effects, nested `(spin …)` per item
for concurrent, or async-sequence primitives in
`spindel.seq.{core,combinators}` for streaming. **No** automatic
`map-spins` / `filter-spins` exists — don't invent it.

### Rule 3 — Runtime access via protocols only; never the `:state` field

```clojure
;; ❌ direct field access — breaks across backend implementations
(get-in @(:state runtime) [:nodes spin-id :result])

;; ✅ protocol method via the runtime
(rtp/get-state runtime [:nodes spin-id :result])

;; ✅ facade that uses *execution-context* dynamically
(ec/get-state [:nodes spin-id :result])
```

The runtime protocols (`PState`, `PGraph`, `PSpinLifecycle`,
`PContinuation`, `PEngine`, `PScheduler`, `PDepsTracking`) are the
public engine API. Fields on the `ExecutionContext` record (the
`:backend`, `:state-atom`, etc.) are implementation details that
change shape between AtomBackend, OverlayBackend, and
ImmutableBackend.

Facade fns in `engine/core.cljc` (`ec/get-state`, `ec/swap-state!`,
`ec/cas-state!`) read `*execution-context*` dynamically — use these
when you have a current ctx; use the protocol forms when you
already have an explicit ctx in hand.

### Rule 4 — `pcps-async/*in-trampoline* false` when resuming from external threads

When invoking a continuation from outside the spin's own CPS
trampoline (a future / virtual-thread callback, a JS setTimeout, an
HTTP response, an event-listener), bind `*in-trampoline* false`
first:

```clojure
;; ✅ async callback from external thread
(future
  (try (Thread/sleep 10)
       (binding [pcps-async/*in-trampoline* false]
         (cont/resume resolve value))
       (catch Throwable t
         (binding [pcps-async/*in-trampoline* false]
           (cont/resume reject t)))))

;; ❌ inside CPS code — already in a trampoline; double-binding
;;    would establish a nested one and cap stack growth uselessly
(defn wrapped-cps-fn [r]
  (fn [_r _e]
    (cont/resume r nil)
    spin-core/incomplete))
```

| Re-entry context | Bind `*in-trampoline*`? | Why |
|------------------|------------------------|-----|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [replikativ/spindel](https://github.com/replikativ/spindel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
