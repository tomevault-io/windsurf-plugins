---
trigger: always_on
description: How to build ClawCodex Desktop well. This is a judgment guide, not an inventory —
---

# Desktop Engineering Guide

How to build ClawCodex Desktop well. This is a judgment guide, not an inventory —
it teaches the invariants and the reasoning behind them so a change fits the app
even as files move. Read it with the repository `AGENTS.md` (root rules still
apply) and [`DESIGN.md`](./DESIGN.md) for the visual and interaction contract.

When a rule here and the code disagree, trust the code and fix whichever is
wrong — but never break an invariant to make a change easier.

## What this app is

Desktop is its own native chat surface. It is not the browser dashboard and it
does not embed the TUI. Three parties, each authoritative for one thing:

- **Electron** owns the machine: process lifecycle, native filesystem/git/
  windows, install/update, and a narrow, typed capability bridge.
- **The renderer** owns the experience: navigation, presentation, and ephemeral
  interaction state.
- **The agent backend** owns the work: sessions, tools, model calls, streaming.

Keep the seams clean. The renderer never reaches for Node or Electron directly;
native power arrives through a deliberate capability, not a general escape hatch.
Agent behavior lives behind the gateway, never reimplemented in React. When a
change blurs a seam, that is the smell — fix the seam, don't widen it.

## Decide state by authority

The first question for any piece of state is *who is allowed to be right about
it*, not where it is convenient to store it. Put state with its authority:

- The **backend** is authoritative for anything another ClawCodex surface can also
  change. Treat the renderer's copy as a cache of that truth.
- **Electron** is authoritative for machine and runtime facts.
- The **renderer** owns only what is purely about this window's presentation.

From that, everything else follows: shared renderer state lives in small stores
owned by the feature that owns the concern; request-shaped server data that wants
invalidation lives in the query layer; short-lived interaction detail stays in
the component; hot coordination that must not paint stays in a ref. Reach for the
narrowest home that still lets the state be correct. A new global store is a
claim that many distant surfaces need it — earn that claim.

Persisted state must declare its scope in its own key: is this global, or does it
belong to a connection, a profile, a stored session, a project, or a window?
Getting the scope wrong is how one profile's setting bleeds into another.

## Identity is not incidental

Sessions have more than one identity, and conflating them is a recurring source
of "session not found" and vanishing history. Reason about which identity a
surface needs: durable navigation and anything the user pins or persists key off
the stable/durable identity; live streaming keys off the runtime identity; state
that must outlive compression keys off the lineage root. Keep the mapping between
them explicit and translate at the boundary rather than passing the wrong id
inward.

## Server truth is cached, not owned

The renderer paints from a cache of backend truth, so it must reconcile, not
assume:

- **Merge, don't clobber.** A refresh is new information layered over what you
  already know, not a replacement that can drop live or pinned rows.
- **Be optimistic, then honest.** Direct manipulation should paint immediately
  from a snapshot; a failed write rolls back visibly and an authoritative
  refresh gets the last word.
- **Guard against the past.** Async results can arrive out of order; a stale
  response must never overwrite newer intent. Generation counters and request
  tokens exist for this.
- **Isolate the foreground.** Only the surface the user is looking at may publish
  into the shared view; background work updates its own cache quietly.
- **Coalesce noise, flush signal.** Batch high-frequency cosmetic updates, but
  let terminal transitions (a turn finishing, needing input, failing) reach the
  user immediately.
- **Preserve reference identity on no-ops.** Handing React a fresh array that
  contains the same data re-renders expensive trees for nothing.

## Switching context is a re-home, not a reboot

Changing profile, connection, or mode is a workspace switch, not a cold start.
The shell and whatever the user was doing stay put; only the gateway-bound view
is cleared and repopulated, and the previous context must not leak into the next
one. Reserve the full-screen boot/connecting experience for a genuinely unusable
backend.

There are three distinct switch shapes, and conflating them is the classic bug:

- A **connection/mode apply** (local ↔ remote ↔ cloud) is the soft re-home:
  shell mounted, gateway-bound stores explicitly wiped, then reconnect. Query
  invalidation alone cannot evict live session stores — wipe them.
- A **runtime home change** (switching the underlying `CLAWCODEX_CONFIG_DIR` profile) is
  a hard re-home: the window legitimately reloads and state resets by remount.
- A **live profile swap** in the same window activates another profile's socket
  while background profiles keep streaming; lists merge rather than wipe, and
  only an explicit user selection starts a fresh foreground draft.

Treating a soft switch as hard flickers the app; treating a hard one as soft

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agentforce314/clawcodex](https://github.com/agentforce314/clawcodex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-30 -->
