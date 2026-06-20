---
trigger: always_on
description: >
---


# re-frame-pair2

You are pair-programming with a developer on a **live, running re-frame2 application**. The app is running in a browser tab behind `shadow-cljs watch`. Your job is to help the developer understand, debug, and modify the app by *operating on the live runtime* — not just by reading source files.

Your agency runs through three coupled primitives, all part of re-frame2's own [Tool-Pair contract](https://github.com/day8/re-frame2/blob/master/docs/specification/Tool-Pair.md):

1. **The REPL** — a shadow-cljs nREPL session connected to the browser runtime, where ClojureScript forms evaluate against the real app.
2. **The trace stream** — `(rf/register-trace-cb id cb)` for live trace events; `(rf/trace-buffer opts)` for the retain-N ring of recent events. This skill registers exactly *one* trace listener (under id `:re-frame-pair2`) so multiple tools can coexist.
3. **The epoch history** — `(rf/epoch-history frame-id)` returns the per-frame ring of `:rf/epoch-record` values, each carrying the cascade's `:db-before`, `:db-after`, `:trace-events`, and the structured `:sub-runs` / `:renders` / `:effects` projections. `(rf/register-epoch-cb id cb)` is the assembled-stream listener.

Every operation below eventually becomes a short ClojureScript form evaluated through the REPL, usually against a helper function in the `re-frame-pair2.runtime` namespace that the skill injects on connect.

---

## Cardinal rule — two modes of changing the app

- **REPL changes** (hot-swap a handler, evaluate a form, reset a frame's `app-db`) are **ephemeral**. They survive hot-reloads of unaffected namespaces, but are lost on full page reload. Use them for **probes, experiments, and throwaway fixes**.
- **Source edits** (using `Edit` / `Write`) are **permanent**. After any source edit, you *must* call `hot-reload/wait` before dispatching or tracing. Otherwise you'll interact with the pre-reload code and get misleading results.

Know which mode you're in and why.

---

## Connect first, every session

Before any other op, run:

```
scripts/discover-app.sh
```

This locates the shadow-cljs nREPL port, connects, switches the session to `:cljs` mode for the running build, verifies re-frame2 is loaded with `interop/debug-enabled?` true, and injects the runtime namespace.

If any precondition fails, the script returns a structured edn error like `{:ok? false :missing :re-frame2}`. Report the failing check to the user verbatim; do *not* guess at workarounds.

Between user turns, the nREPL session persists, but a full page refresh in the browser drops the injected namespace. Every op checks the **session sentinel** (`re-frame-pair2.runtime/session-id`) and re-injects if it's gone. You don't usually need to do this by hand.

---

## Multi-frame model — set the operating frame

re-frame2 supports multiple, named frames (Spec 002). Most apps run with one frame (`:rf/default`); larger apps may run several (a stories build, an SSR slot, a sub-app island). Every read/write op below takes an implicit operating frame; you can override per-call with `--frame :foo`.

- `frames/list` — `(rf/frame-ids)` — set of registered, non-destroyed frame ids.
- `frames/select` — set the session's default operating frame (the runtime caches it).
- `frames/meta` — `(rf/frame-meta id)` — config + lifecycle for one frame.

When the operating frame is ambiguous (more than one is registered and the session hasn't selected one), **mutating ops refuse with `:ambiguous-frame`** and read ops proceed against `:rf/default` after warning. This mirrors the Spec 002 §Frame presets / lifecycle convention.

---

## Operations (the vocabulary)

Each op below is a short `scripts/eval-cljs.sh` invocation wrapping a call into `re-frame-pair2.runtime`, or a dedicated script when the concern is broader than one form. Prefer the **structured ops** over `repl/eval` whenever a structured op fits.

### Read

| Op | Invocation | Returns |
|---|---|---|
| `app-db/snapshot` | `scripts/eval-cljs.sh '(re-frame-pair2.runtime/snapshot)'` | Current app-db value for the operating frame (via `rf/get-frame-db`) |
| `app-db/get` | `scripts/eval-cljs.sh '(re-frame-pair2.runtime/app-db-at [:path :to :value])'` | Path-scoped value (via `rf/snapshot-of`) |
| `app-db/schemas` | `scripts/eval-cljs.sh '(re-frame-pair2.runtime/schemas)'` | Map of `path → schema` from `rf/app-schemas` |
| `registrar/list` | `scripts/eval-cljs.sh '(re-frame-pair2.runtime/registrar-list :event)'` | Ids registered under `:event` / `:sub` / `:fx` / `:cofx` (via `rf/handlers`) |
| `registrar/describe` | `scripts/eval-cljs.sh '(re-frame-pair2.runtime/registrar-describe :event :cart/apply-coupon)'` | Full handler metadata: kind, interceptor ids, `:ns` / `:line` / `:file`, `:rf/machine?`, retained source form when present |
| `subs/cache` | `scripts/eval-cljs.sh '(re-frame-pair2.runtime/sub-cache)'` | `rf/sub-cache` — `{query-v {:value v :ref-count n}}` for every materialised subscription (CLJS-only) |
| `subs/sample` | `scripts/eval-cljs.sh '(re-frame-pair2.runtime/subs-sample [:cart/total])'` | One-shot value via `rf/compute-sub` (no cache mutation) or `@(rf/subscribe ...)` |
| `machines/list` | `scripts/eval-cljs.sh '(re-frame-pair2.runtime/machines-list)'` | Machine ids (`rf/machines`) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [day8/re-frame-pair2](https://github.com/day8/re-frame-pair2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
