---
trigger: always_on
description: A desktop app that renders a heap dump's dominator tree as a navigable treemap, as rings around a
---

# Shark Explorer — agent guide

A desktop app that renders a heap dump's dominator tree as a navigable treemap, as rings around a
centre, or as a stack of rows the way a profiler draws a call tree. The long term goal is a YourKit-style
heap explorer; these are the first surfaces.

This file is scoped to `shark/shark-explorer/`. It only records things an agent would get wrong by
reading the source alone — everything else is in the code. Keep it that way.

## Modules

| Module | What it is | Constraints |
| --- | --- | --- |
| `shark-explorer-core` | Heap dump → dominator tree → layout model. Layout, hit testing, navigation state. | **No Compose dependency, Java 8 target.** Must stay reusable from the Android `leakcanary-app`. |
| `shark-explorer-jdwp` | Attaches to a live app as a debugger to read the pixels of its bitmaps. | **Imports `com.sun.jdi`, so it needs a JDK and can't be loaded on Android.** That's the whole reason it isn't in `core`. |
| `shark-explorer-app` | Compose Desktop UI: window, the canvas each shape draws into, details panel. | **Java 17 target** — see below. |

`shark/shark-explorer/` itself holds no code, matching how `shark/` and `leakcanary/` are grouping
directories in this repo.

Put logic in `shark-explorer-core` by default. Anything in `shark-explorer-app` is hard to unit test
and can't be shared with Android, so it should be limited to composables and wiring.

## Use `HeapDominatorTree`, not `ApproximateDominatorTree`

`shark.ApproximateDominatorTree` is the on device BFS approximation, and it is **known to be wrong**
— a cross edge can be processed while the parent's dominator is still stale, so retained sizes get
under-attributed. Don't build on it.

`shark.HeapDominatorTree` is the exact one, which is what `HeapExplorer` uses. See
`notes/dominator-tree.md` for its memory profile and for the reference reader behaviour that makes a
treemap read strangely until you know about it.

## The heap dump is read off the UI thread

A `HeapGraph` is read only and safe to read from several threads at once, so the reason everything that
touches a `HeapExplorer` goes through `HeapDumpSession.read` is **latency, not safety**: labelling a
rectangle is IO, and summarising a selection or walking up to the GC roots is seconds of it on a large
dump. Doing any of that in a composable freezes the window.

What follows for the UI: a composable never holds a tree, only what was already computed from one. A
laid out, labelled view is a `TreemapPresentation` or a `RadialPresentation`, and a selection is a
`HeapObjectSummary`; both arrive a little after whatever asked for them changed.

The one thing that isn't thread safe is a `Sequence` a `HeapGraph` hands out — iterating one reads
through it — so a thread reading `graph.objects` needs its own rather than a shared one.

**Cancelling the coroutine that asked for a read stops the read**, which is what a `LaunchedEffect`
being relaunched does. Shark does the stopping, not us: the heap dump is opened with a `CancelSignal`
asking whether the read in flight is still wanted, and it's asked on every record read, so the work
gives up shortly after the question is withdrawn and comes back as a `CancellationException`. A read
given up on while it was still queued never starts at all. So dragging a window edge costs the size it
lands on and a little of each size it passed through, rather than all of them in full.

Two things follow. **A read is only cancellable at the granularity of what it reads** — a stretch that
computes without reading, like a layout over an already-labelled tree, stops when it next reads — so the
`HOVER_SETTLE_MILLIS` half of this, not starting work that isn't wanted yet, still earns its keep.
And **anything a read mutates has to survive being abandoned half way**: today's reads are safe because
the built-on-first-use indexes are `by lazy` initializers that build a whole object before assigning it
(a cancelled build is simply retried, since `lazy` doesn't cache a failure), and the walks reuse arrays
stamped with a generation per walk rather than cleared at the end.

**The pointer asks questions on that thread too**, because moving over a rectangle describes it. Which is
why nothing is read until the pointer has been still for `HOVER_SETTLE_MILLIS`, and why what a hover asks
for is capped and index-backed: a chain from a GC root is one walk over `ReferrerIndex` with at most 20
steps read out, and the search for every way an object is held runs for the object clicked and no other.
A new question the panels ask has to be measured before it goes in the hover path — `notes/decisions.md`
has the numbers on the biggest dump in the repo.

## A heap dump can have `android.os.Build` and not the fields Shark reads off it

`AndroidBuildMirror.fromHeapGraph` reads `MANUFACTURER`, `ID` and `VERSION.SDK_INT` with `!!`, and nearly
every one of Shark's library leak patterns decides whether it applies by asking it. Those patterns are
filtered against the graph when a `ReferenceReader` is *created*, which in the explorer is while the
dominator tree is being built — so a dump that has the class and not the fields throws a bare NPE from

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [square/leakcanary](https://github.com/square/leakcanary) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
