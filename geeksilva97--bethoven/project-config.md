---
trigger: always_on
description: World Cup prediction pool played **over SSH**. Players connect with `ssh`, pick
---

# CLAUDE.md — BEThoven

World Cup prediction pool played **over SSH**. Players connect with `ssh`, pick
scores in a Bubble Tea TUI, and compete on a leaderboard. A player's **SSH public
key is their identity** — no passwords, no signup forms.

## Stack

- **Go** (module `bethoven`, single static binary).
- **charmbracelet/wish** — SSH server. **charmbracelet/bubbletea** + **bubbles** +
  **lipgloss** — TUI. **charmbracelet/wish/bubbletea** — bridges a session to a Tea program.
- **modernc.org/sqlite** — pure-Go SQLite (no cgo), via `database/sql`.

## Architecture — the layering is the point

```
cmd/bethoven        entrypoint: config -> db.Open -> seed -> service.New -> server.New
internal/config     env config (BETHOVEN_*)
internal/clock      Clock interface: Real (prod) + Fake (tests). INJECTED everywhere time matters.
internal/models     pure domain types (no behaviour, no deps) — shared by store/scoring/service
internal/db         SQLite: Open, embedded schema.sql, Store (typed queries), seed.go
internal/auth       key fingerprint, admin allowlist, invite-code check
internal/scoring    PURE Points(bet, match) — the heavily unit-tested core
internal/live       OPTIONAL live feed: ESPN adapter + in-memory Cache + Poller. Behind the service.LiveStore port.
internal/analytics  OPTIONAL usage tracking: own SQLite DB + async Recorder. Behind the service.AnalyticsSink port.
internal/ai         OPTIONAL AI player "BETanIA": Claude-API predictor + live Bettor + seeder + Monitor. Behind the service.AIMonitor port + a trigger seam. NEVER imports service (function seams).
internal/service    ALL business rules. Depends only on Store + Clock (+ optional LiveStore/AnalyticsSink/AIMonitor). The integration-test seam.
internal/server     thin wish SSH server -> resolves key to user -> launches TUI
internal/tui        presentation only; every action calls a service method
```

**Golden rule:** business logic lives in `internal/service`. `server` and `tui`
are thin. If you're tempted to put a rule in the TUI, put it in the service and
call it from the TUI — that's how it stays testable (tests drive the service
directly, with a fake clock, no terminal).

## Core rules / invariants

- **Kickoff lock (most important):** you can only bet while `clock.Now().UTC() <
  match.StartsAt`. Enforced in `service.PlaceBet` using the **server clock only** —
  never trust client time. Also rejects `match.Finished` (belt-and-suspenders for
  clock skew / early result entry). Invariant: *cannot bet an ongoing or ended match.*
- **Own-bets-only (default):** player queries are scoped to their `user_id`;
  players never see another player's individual picks. The admin `AllBets` grid
  exposes raw picks and is gated by `requireAdmin` in the service (not just hidden
  in the UI). **Exception — `public_bets`:** an admin can flip the DB-backed
  `public_bets` setting (admin TUI → Settings) to open the grid to all players via
  `service.PublicBetsGrid`. Even then, picks are revealed **only for matches that
  have kicked off or finished** (`m.Finished || now >= m.StartsAt`) — upcoming
  matches are omitted entirely, mirroring the `MatchLeaderboard` reveal rule, so
  blind betting is preserved. Enforced server-side, not just in the UI.
- **Scoring — admin-selectable mode (`scoring_mode` setting; default Classic).**
  Three pure functions in `internal/scoring`, dispatched by `scoring.Score(mode, …)`:
    - **Classic** (default, `scoring.Points`): exact = 3; correct result (W/D/L)
      only = 1; wrong = 0. Tiers are **mutually exclusive** — exact is 3, not 3+1.
    - **Proximity** (`ProximityPoints`): 0 for a wrong result, else
      `max(1, 5 − distance)` where `distance = |predA−scoreA| + |predB−scoreB|`.
      Exact → 5, one goal off → 4, …, floor of 1 for calling the winner. (Kicktipp's
      "deduction per goal" model.)
    - **Scarcity** (`ScarcityPoints`): Proximity **plus** a contrarian bonus —
      +2 for a correct result <25% of the match's bets picked, +2 for a correct
      exact score <10% picked. **Quorum gate (`scarcityQuorum`, 8):** the bonus
      only applies once a match has ≥8 bets; below that "rare" is just noise (a
      lone picker in a 4-person field isn't a contrarian), so Scarcity scores
      identically to plain Proximity. **Pool-relative**, so it takes a
      `scoring.Pool` (counts of same-result / same-exact bets + total) the
      **service** computes; the pure function never reads the DB.
  Knockouts store the **regulation 90' score** in every mode, so ET/penalties are
  ignored and a 1-1 a.e.t. scores as a 1-1 draw.
  **Plumbing:** the service builds a `scorer` (`service_scoring.go`) once per
  read; it carries the mode and, **only for Scarcity**, the per-match pick
  distribution (built from `AllBets`). All point-bearing reads — `Leaderboard`
  (incl. the synthetic-match live fold), `MyResults`, `MatchLeaderboard`,
  `buildBetsGrid` — call `sc.points(b, m)`, never `scoring.Points` directly.
  Admin picks the mode in the TUI **Settings** screen; players see the active
  mode and its rules in **"How scoring works"** (`scoring_rules.go`). Mode is
  stored/read like `public_bets` (KV `settings` table; absent ⇒ Classic), so
  existing pools are unaffected until an admin opts in.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [geeksilva97/bethoven](https://github.com/geeksilva97/bethoven) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-21 -->
