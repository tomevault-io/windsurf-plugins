---
trigger: always_on
description: <!-- BEGIN KATA (managed by `kata init --with-agents`) -->
---

<!-- BEGIN KATA (managed by `kata init --with-agents`) -->
## kata issue tracker

This project uses [kata](https://github.com/kenn-io/kata) as its shared issue
ledger. Run `kata quickstart` at the start of each session for the full agent
contract. The short version:

- Search before creating: `kata search "<keywords>" --agent`.
- Prefer updating existing issues over duplicates (`kata comment`, `kata label add`, `kata edit`).
- Default to `--agent` for ordinary reads and mutations; use `--json` only when a script needs structured data.
- Close only verified work: `kata close <ref> --done --message "<scope + verification>" --commit <sha>`.
- If work is incomplete, label `needs-review` and comment what remains rather than closing.
- Never `kata delete` or `kata purge` without explicit user authorization.
<!-- END KATA -->

## Project rules

### Design constraints (do not relax without asking)

- Tailnet-first. `tailscale serve` (tailnet-only exposure) is the default
  path. `tailscale funnel` (public internet exposure) IS supported, but only
  as a deliberate, per-service opt-in via the `p` key (kata vzj4 swapped this
  to `P`, on the theory that capital guards the more-permanent exposure; kata
  58ws REVERSED that call at the owner's request — funnel is back on the
  bare `p`, and Publish moved to `d`, see below) behind a strong y/n
  confirm that names the port and shows the resulting public URL. `:22` (SSH)
  is hard-blocked from funnel. Never funnel implicitly, in bulk, or without
  that confirm. (Implemented under kata yt69: the `P` key, `entryConfirmFunnel`
  gate, and `tsserve.FunnelOn/FunnelOff/FunnelStatus`; re-lettered `P` -> `p`
  under kata 58ws.)
- Publish-via-edge is a SECOND public path (the `d` key, kata v1z5; swapped
  from `P` to `p` under vzj4, then moved from `p` to `d` under kata 58ws,
  which also reversed vzj4's swap for Funnel), **independent of Funnel —
  not layered or ranked above it, and no longer mutually exclusive with
  it: kata th05 relaxed that rule (an owner-approved reversal)**. A local
  port may now carry Funnel AND Publish at once, each its own navigable
  route sub-row: the `d` path no longer refuses a funnelled port, nor does
  the `p` path refuse a Caddy-published one — there's no more "remove the
  other exposure first."
  There is no implicit precedence between them, and multiple public paths on
  one port are now a legitimate, expected state, not drift — never silently
  collapsed to one marker. It carries the same funnel-grade guardrails:
  per-service opt-in, a strong y/n confirm naming the exact `https://<hostname>`
  URL, `:22` hard-blocked, ungated de-escalation (an immediate unpublish, no
  confirm), and unpublish never touches serve state. The `d` key is a TOGGLE
  (kata prp1): pressed again on a port published earlier THIS session
  (remembered hostname + auth, session-only, never persisted), it re-publishes
  with that remembered config, skipping the host/auth setup prompts — still
  behind the same y/n confirm naming the exact hostname, UNLESS the owner has
  opted into `caddy.silent_republish` (config, default OFF), an
  owner-approved, documented exception to the always-confirm rule that skips
  even that confirm. This exception is scoped strictly to RE-publishing an
  ALREADY-consented hostname within the SAME session — a port's FIRST publish
  always confirms regardless of this setting, and Funnel's confirm is
  completely unaffected. A dedicated `e` key opens the same setup flow (also
  always confirming) to edit a port's publish config: it changes the AUTH of an
  already-published port in place, but REFUSES to move a still-published port to
  a new hostname (that would be a non-atomic delete-and-create that could leave
  the old route dangling — the user unpublishes first, kata sw2y); it never
  de-escalates. That refuse is BEST-EFFORT: it keys off the poll cache, which can
  be stale/empty, so a rename can still slip through in that narrow window (a
  pre-existing property of the cache-based model, not a regression; roborev
  44n7) — the authoritative fix (a live-route scan / atomic replace at publish
  time) is tracked in srx1 for v0.2.1. In this path **Tailscale
  supplies private tailnet transport only; Caddy owns the entire public trust
  plane** (custom-domain DNS, public `:443` ingress, TLS termination and
  renewal, hostname routing). Basic auth at the edge is a single SHARED
  credential stored as a bcrypt hash, never plaintext. Published state is read
  live from the edge's `@id`-tagged routes on a separate poll, never persisted
  per-port — Caddy is the source of truth, the same philosophy as serve/funnel
  state being read live. (Implemented under kata v1z5: `internal/caddyedge`,
  the `caddy:` config block, and the `p` key / `entryConfirmPublish` gate /
  published-state poll in `internal/ui`. The `p` toggle, the `e` edit key, the
  session-only `lastPublish` memory, and `caddy.silent_republish` were added
  under kata prp1. Re-lettered `p` -> `d` under kata 58ws, which also swapped
  Funnel back to the bare `p` — see that bullet above for the reversal.)
- Cloudflare Tunnel is a THIRD public path (the `o` key, kata nc1j;
  re-lettered from `t` under kata 7nss, which moved serve onto `t`),

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gruen/tailport](https://github.com/gruen/tailport) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-20 -->
