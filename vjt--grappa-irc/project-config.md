---
trigger: always_on
description: An always-on IRC bouncer with a REST API + Phoenix Channels real-time event
---

# Grappa — Project Memory

## What This Is

An always-on IRC bouncer with a REST API + Phoenix Channels real-time event
push, plus a browser PWA (`cicchetto`, separate codebase) that looks like
irssi. One supervised OTP process per `(user, network)`; sqlite-backed
scrollback; Phase 6 adds a downstream IRCv3 listener facade.

See `README.md` for the spec and `docs/DESIGN_NOTES.md` for the
chronological decision log. Backlog and roadmap live in GitHub issues
(`gh issue list`); implementation plans are ephemeral scratch, never
committed (see Docs map). Operator + developer runbook (verbs, scripts,
deploy machinery, per-host overrides, runtime data, monitoring) lives in
**`docs/OPERATIONS.md`**.

## Architecture

Top-level supervision tree:

```
Grappa.Application
├── Grappa.Vault                       (Cloak — encrypts at-rest creds; before Repo)
├── Grappa.Repo.LockWatch              (#1420 write-lock holder/waiter observer; before Repo — owns the ETS table the BEGIN IMMEDIATE seam writes to)
├── Grappa.Repo                        (Ecto + sqlite)
├── Phoenix.PubSub                     (name: Grappa.PubSub)
├── Registry                           (name: Grappa.SessionRegistry)
├── Registry                           (name: Grappa.SourceAliasHolders — #543 derived-alias holder index; before SessionSupervisor: sessions register on acquire)
├── Grappa.Session.Backoff             (ETS — per-(subject, network) failure counter)
├── Grappa.WSPresence                  (per-user WS pid tracking → auto-away signal)
├── Grappa.Admission.NetworkCircuit    (T31 ETS-backed per-network circuit breaker)
├── Grappa.AdminEvents                 (M-11 admin-event ring buffer + telemetry sink)
├── Grappa.SessionLog                  (#215 IRC session-lifecycle log sink)
├── Grappa.DbLatency                   (#357 SQLite write/query-latency telemetry sink)
├── Grappa.Push.VendorLog              (#1321 push rejection-reason telemetry sink)
├── Grappa.ShareTokens                 (ETS one-shot share-link tokens, both subject kinds)
├── Grappa.RateLimit.DailyQuota        (#75 per-(bucket, subject, day) creation quota)
├── Grappa.RateLimit.FailureWindow     (S6 per-(bucket, key) login-throttle window)
├── Grappa.Accounts.WebAuthnChallengeStore (short-lived WebAuthn ceremony challenges)
├── Grappa.RateLimit.TokenBucket       (#340 per-(subject, network) send token bucket)
├── Grappa.Net.PtrCache                (#252 vhost reverse-DNS (PTR) name cache)
├── Task.Supervisor                    (name: Grappa.TaskSupervisor — detached tasks)
├── Grappa.WindowCounts.Pusher.Coalescer (#1768 one window_counts snapshot per window per window_ms; after TaskSupervisor — flushes into it)
├── DynamicSupervisor                  (name: Grappa.SessionSupervisor)
│   └── Grappa.Session.Server          (one per (user, network), :transient)
├── GrappaWeb.Endpoint                 (Phoenix HTTP + WS)
├── GrappaWeb.SessionRevocationListener (turns a bearer-death event into the WS teardown; with/after Endpoint)
├── Grappa.Net.SourceAliasManager      (#543 alias ref-counts; after Endpoint — boot reconcile; before Bootstrap — sessions acquire)
├── Grappa.Visitors.Reaper             (60s sweep of expired visitors; after Endpoint)
├── Grappa.Uploads.Reaper              (UX-6-B1 upload GC sweep; after Endpoint)
├── Grappa.Avatars.Reaper              (M3b peer-avatar cache GC sweep; after Endpoint)
├── Grappa.Dcc.Reaper                  (2089 DCC spool GC sweep; after Endpoint — the serving route must be up first)
├── Grappa.Accounts.Reaper             (#223 idle auth-session GC; after Endpoint)
└── Grappa.Bootstrap                   (reads DB credentials, spawns sessions; LAST)
```

Child order is load-bearing — see `lib/grappa/application.ex` for the
why-comment per child. Vault before Repo (Cloak schema callbacks);
LockWatch before Repo (#1420: it owns the ETS table
`Repo.immediate_transaction/1` writes on every write transaction, and the
seam self-disables while the table is absent);
Backoff/WSPresence/NetworkCircuit before SessionSupervisor (ETS
tables read directly from `Session.Server`'s start path); Bootstrap
LAST (depends on Registry + SessionSupervisor existing). `Grappa.SpawnOrchestrator`
is a top-level boundary module (admission → Backoff.reset → spawn
verb), NOT a supervised child — both Bootstrap and
`NetworksController.connect/2` call into it.

Key invariants — break only with deliberate cause + DESIGN_NOTES entry:
- **One IRC parser, on the server.** `Grappa.IRC.Parser` is the single
  source of truth for IRC framing. `cicchetto` (the web PWA) NEVER parses
  IRC; it consumes typed JSON events.
- **Scrollback is bouncer-owned.** sqlite via Ecto. Schema is
  `(network_id, channel, server_time DESC)`-indexed; a future
  `CHATHISTORY` listener facade (Phase 6) is a mechanical query
  translation, not a redesign.
- **Identifiers (nicks AND channels) are case-folded by ONE
  `canonical_target` fold at every KEY boundary (GH #537, unifying #121
  nicks + #364/#525 channels); wire + display stay RAW.** bahamut
  (Azzurra, all of prod) advertises AND implements `CASEMAPPING=ascii`: it
  folds `A-Z` ONLY, leaving `[ ] \ ~` UNTOUCHED, and folds channels the
  SAME way it folds nicks. The single source of truth is

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vjt/grappa-irc](https://github.com/vjt/grappa-irc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-14 -->
