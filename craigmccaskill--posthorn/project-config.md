---
trigger: always_on
description: Auto-loaded by Claude Code at session start. Captures the durable project context, current status, and the guardrails that need to be in front of every code change. Build/test commands, commit conventions, and contributor scope policy live in [CONTRIBUTING.md](./CONTRIBUTING.md) — read both at the start of any code session.
---

# CLAUDE.md — Posthorn

Auto-loaded by Claude Code at session start. Captures the durable project context, current status, and the guardrails that need to be in front of every code change. Build/test commands, commit conventions, and contributor scope policy live in [CONTRIBUTING.md](./CONTRIBUTING.md) — read both at the start of any code session.

## Project context

Posthorn is the **unified outbound mail layer for self-hosted projects** — the gateway between an operator's apps and a transactional mail provider they already chose. v1.0 ships three ingress shapes (HTTP form, HTTP API mode with Bearer auth and idempotency, SMTP listener with AUTH PLAIN + STARTTLS), five transports (Postmark, Resend, Mailgun, AWS SES with bespoke SigV4, outbound-SMTP relay), and an operational surface (`/healthz`, `/metrics` Prometheus exposition, dry-run, CSRF tokens, IP-stripping, named `trusted_proxies` presets). v2 adds platform features (durable storage, suppression, lifecycle webhooks, attachments).

Originally sequenced as v1.0 → v1.1 (API mode) → v1.2 (multi-transport + ops) → v1.3 (SMTP ingress) themed releases. Folded into a single v1.0 release on 2026-05-16 after evaluation concluded the surface area was small enough that splitting them into four releases produced more carve-outs than coherent product moments. v2 remains the next planned milestone.

The original wedge — cloud-blocks-SMTP — is preserved as the canonical discovery entry point. The broader value is the unified-layer pattern (the same outbound concern duplicated across N self-hosted apps now centralizes through one Posthorn gateway), which applies even where SMTP is unblocked.

The full project history (initial scope as a Caddy form handler called `caddy-formward`, the 2026-04-27 scope expansion, the 2026-05-15 positioning sharpening, the 2026-05-16 v1.x-fold-into-v1.0) is in [`spec/01-project-brief.md`](./spec/01-project-brief.md) §"Status log". Don't re-derive — read the spec.

## Design principles (short)

These pin the shape of Posthorn across versions and override new feature requests that conflict with them. Full reasoning in [`spec/01-project-brief.md`](./spec/01-project-brief.md) §"Design principles".

1. **Gateway, not infrastructure.** Posthorn sits between apps and a provider — it doesn't replace the provider, doesn't run its own outbound SMTP, doesn't manage IP reputation, doesn't host mailboxes.
2. **Integration layer, not mail-receiving layer.** Posthorn unifies outbound (many ingress shapes → one transport surface). It does not unify inbound (MX hosting / receive-side filtering / mailbox storage); those are mail-server concerns.
3. **No feature-count competition with category leaders.** Stalwart owns mail-server territory; Postal owns outbound-platform territory; Listmonk owns marketing. We don't try to match them on feature count in their lanes.
4. **Config files over admin UIs.** A single TOML file is the source of truth. No runtime mutation surface that could drift from the config file. v3+ may add read-only UIs for browsing logs; not for configuration.
5. **Bespoke before SDK, when the surface is small.** Postmark / Resend / Mailgun / SES integrations are written directly (stdlib + minimal deps), not via vendor SDKs. The rule of thumb: bespoke when ~200 lines suffices; SDK when bespoke would be 1000+. [ADR-1](./spec/03-architecture.md#architectural-decisions-log) elevated project-wide.

When a feature request or implementation proposal conflicts with one of these, the principle wins. Take it back to spec discussion before changing code.

## Status (as of 2026-07-03)

**Phase: v1.2.0 released 2026-07-04** (spam-protection minor; validated via `v1.2.0-rc.1` deployed on craigmccaskill.com against live spam before GA). `:latest`/`:1`/`:1.2`/`:1.2.0` → the release; `:1.1.0`/`:1.0.0` preserved. **v1.2.0 = the spam checks** the whole audit was in service of: after real captured spam showed the pattern is content-shapeless (distributed reply-bait, burner Gmails, four languages, 0–1 links), the milestone was re-cut to content-agnostic checks — #44 reputation (StopForumSpam email/IP, fail-open, cached), #45 proof-of-browser (ADR-18 challenge endpoint + `min_age` time-trap), #33 Turnstile (fail-closed backstop), all on the #60 two-phase pipeline. Plus #57/#58 SMTP metrics/validate fixes and Authentik/Mastodon recipes. Content-shape #34 and domain-blocklist #35 deprioritized against the data.

**Earlier: v1.1.0 (2026-07-04) = the full-audit hardening release** (PRs #49/#64/#65/#66/#67/#77/#78/#79 + 6 Dependabot): #37 listener-only-config bug, #41 auth-none public-bind refusal (BREAKING — needs `trusted_network = true`), #42 server timeouts, #50 SMTP brute-force/conn caps, #59 shared retry, #38/#39/#56 CI/supply-chain, #76 provider-testing harness. v1.0.0 was 2026-05-26. Two external humans engaged (amit-tewari #30 multi-tenant SMTP; monperrus python-posthorn).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [craigmccaskill/posthorn](https://github.com/craigmccaskill/posthorn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
