---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working
with code in this repository. Refreshed 2026-07-03 — the previous
version documented the retired UIx/React architecture.

## What simmis is

A business-oriented, value-building, self-programming environment on the
datahike.io stack: chat + wiki + knowledge bases + AI staff, one shared
memory substrate. The UI is **spindel** (FRP signals/spins rendering
DOM incrementally — NOT React); the agent substrate is **dvergr**
(discourse rooms, SCI sandbox, per-room scheduler, telegram channel);
persistence is **datahike** over **konserve** with **yggdrasil** CoW
branching, synced client↔server via **konserve-sync** over **kabel**
websockets with **kabel-auth** JWT.

simmis is the prime showcase for spindel — hold UI code to idiomatic
reactive decomposition (see "spindel sharp edges" below).

Documentation: `doc/architecture.md` first, then `doc/data-model.md`,
`doc/authority.md`, `doc/proposals-and-time-travel.md`, `doc/agents.md`.
Each describes code that exists; there is no roadmap or backlog in the
repository, and open work is not tracked in documentation.

## Instructions

- Use `clj-nrepl-eval` to iterate and validate before large changes.
- *Avoid fallback solutions* except for experimentation; ask when you
  can't solve something without one.
- Git commit after significant milestones; not while stuck or debugging.
- Circle back to `doc/` after an implementation and before committing. Those
  five documents describe code that EXISTS — if a change makes one of them
  wrong, fix it in the same commit rather than adding a note about it.
- When isolating a problem is hard, add log statements / REPL probes.
  For reactive-correctness bugs, trace ENGINE state via REPL probes —
  not app-level workarounds.

### NEVER Use Delays as Patches

Never use `setTimeout`/`Thread/sleep`/timeouts to "fix" timing or
synchronization issues. Use real coordination: await events, channels,
promises, signals, or pass data explicitly. (One legitimate exception:
a `requestAnimationFrame` to defer DOM measurements until after tree
linkage — a documented DOM lifecycle idiom, not a race patch.)

## Development environment

```bash
npm i               # once
clj -M:dev          # START HERE: Maven only, needs nothing but this repo
clj -M:stack:dev    # + a local ../dvergr, for co-developing the agent substrate
clj -M:local:dev    # + every replikativ sibling as a git checkout (see below)
```

One command starts: nREPL **47888** (CLJ server), shadow-cljs watcher
with nREPL **9631** (CLJS), websocket server **ws://localhost:47295**,
static files **http://localhost:8080**.

Dependencies come in three sets (deps.edn): Maven releases by default;
`:stack` overlays ONLY `../dvergr` (the sibling simmis co-develops with); `:local`
overlays every replikativ sibling (`../spindel`, `../datahike`, `../konserve`,
…).

**Default to `:stack`.** `:local` inherits whatever branch each sibling
checkout happens to sit on, so parallel work in another repo breaks simmis's
boot with an error that points at simmis. Two real instances: a `../datahike`
checkout parked on a query-planner branch fails the konserve version check
(`:now nil` — a source checkout reports no version; datahike#902 makes the
check tolerate that, still open), and a sibling mid-edit fails compilation.
Reach for `:local` deliberately, when the change you are testing IS in a
sibling — and expect to check what branch it is on.

### REPL usage

```bash
# CLJ (server) — port 47888
clj-nrepl-eval -p 47888 "(status)"
clj-nrepl-eval -p 47888 "(help)"                  # the read surface, listed
clj-nrepl-eval -p 47888 "(user/restart-web!)"     # web server only, no JVM restart

# CLJS (browser) — port 9631; jack into the build FIRST (session persists):
clj-nrepl-eval -p 9631 "(shadow/repl :app)"
clj-nrepl-eval -p 9631 "@is.simm.uis.web.desktop.signals/layout-columns"
```

**Start with `(status)`.** It probes the ports rather than reporting the
dev namespace's own bookkeeping (which stays `true` after a component
has died), reports whether the shadow watcher is up, and — the one that
saves the most time — whether the bundle is STALE:

```clojure
{:ports {:nrepl-clj true :nrepl-cljs true :http true :websocket true}
 :shadow {:app-watching? true}
 :bundle {:built? true :size-mb 34 :age-min 0 :stale? false}
 :system-db {:rooms 10 :parties 12 :kbs 8}}
```

`:stale?` compares `public/js/app.js` against the newest `.cljs`/`.cljc`
source. A FAILED build leaves the previous bundle in place, so "Build
completed" in the log and a served page can both predate your edit —
which is indistinguishable from success until you check this.

**Read the running system without writing queries or requires.** `dev/repl.clj`
is referred into `user`, so these work as bare one-liners:

```clojure
(rooms)                  ;; every room, newest first
(parties) (knowledge-bases)
(room "verification")    ;; by slug, uuid, or a substring of the name
(room-conn "verification");; that room's OWN store conn — nil = not hydrated here
(kb-conn db-scope)
(conn) (db)              ;; the system DB
(q '[:find ?n :where [_ :room/name ?n]])
```

`repl` only reads; lifecycle stays in `user`, where it is harder to call

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [replikativ/simmis](https://github.com/replikativ/simmis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
