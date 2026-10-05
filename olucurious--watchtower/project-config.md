---
trigger: always_on
description: This file is for anyone changing Watchtower, people and coding agents alike.
---

# Working on Watchtower

This file is for anyone changing Watchtower, people and coding agents alike.
It describes how the system fits together and the rules every change must
keep. [CONTRIBUTING.md](CONTRIBUTING.md) covers setup and the pull request
process; [README.md](README.md) covers using it.

## What Watchtower is

A self-hosted error tracker. Official vendor SDKs (Sentry, AppSignal) report
to it unchanged: users change only an endpoint or DSN. Each vendor protocol is
an **adapter**; everything behind the adapters is shared. It ships as one Go
binary with the web UI embedded, and Postgres is its only dependency.

```
SDK ─▶ adapter (auth, decode, convert) ─▶ scrub ─▶ ingest_queue ─▶ worker ─▶ issues + events
                                                      (symbolicate, group, regressions,
                                                       alerts and emails in one transaction)
```

## Where things live

| Path | What it is |
|---|---|
| `cmd/watchtower` | The binary: `serve`, `migrate` and admin commands; wires everything together |
| `internal/adapter/<vendor>` | Ingestion protocols (`sentry`, `appsignal`); `adaptertest` has in-memory fakes |
| `internal/event` | The canonical event model every adapter produces |
| `internal/scrub` | Redaction applied before anything is stored |
| `internal/grouping` | Which issue an event belongs to (versioned) |
| `internal/symbolicate`, `internal/sourcemaps` | Source map uploads and frame mapping |
| `internal/store` | Everything in Postgres; migrations in `store/migrations` |
| `internal/worker` | Drains the ingest queue; hourly maintenance |
| `internal/alert` | Slack and Linear delivery, Linear sync, credential sealing |
| `internal/email` | SMTP and Cloudflare senders, rendering, the mailer loop |
| `internal/mcpserver` | The MCP endpoint for coding agents |
| `internal/api` | The JSON API behind the web UI (`/api/v1/`) |
| `internal/server` | Assembles the HTTP surface: adapters, health, metrics |
| `internal/config` | `WATCHTOWER_*` settings, validated at startup |
| `web/` | React UI, built by Vite into `internal/ui/dist` and embedded |
| `install.sh`, `compose.yaml` | The one-line installer and the Compose file it downloads |

## Invariants

These hold everywhere. A change that needs to break one needs a discussion
first.

- **Adapters only translate.** They authenticate, decode, convert to
  `event.Event` and call `Sink.Accept`. Grouping, storage and alerting never
  live in an adapter.
- **Acknowledge only after a durable write.** A 2xx means the event is in
  `ingest_queue`.
- **Scrub before anything durable.** `server.scrubbingSink` runs before the
  queue. A new free-text field in `event.Event` must also be handled in
  `scrub.Event` and `Event.StripNUL`.
- **The event model is an allowlist.** Request bodies, headers, cookies,
  breadcrumbs, process state and free-form SDK "extra" data are dropped at the
  adapter. `Event.Context` keeps only named diagnostic values (logger metadata
  an app chose to send, a failed job's identity); extend its allowlist
  deliberately, never by passing extras through.
- **Side effects share the event's transaction.** Issues, regressions, alert
  notifications and emails are written together, so an event is never stored
  without its alert, or alerted twice. Delivery happens later from those
  outboxes.
- **Grouping is versioned.** Any change to `grouping.Components` output bumps
  `grouping.Version`.
- **Shed load explicitly.** Overload returns the vendor's own back-off
  response (for example 429 with `X-Sentry-Rate-Limits`); never block or drop
  silently. The queue limit is an estimate (`depthGauge`); never reintroduce
  a shared counter row, which serializes every ingest commit.
- **Lock issues in (project, fingerprint) order.** The worker locks every
  issue a batch touches in that order before writing events, so concurrent
  workers wait for each other but never deadlock. Any new code that locks
  several issues in one transaction must do the same.
- **Count from the rollup.** Charts, totals and digests read `issue_hourly`,
  which the worker keeps with the events. Don't count `events` rows on a
  request path; it grows with traffic.
- **Migrations are append-only.** Never edit a migration that has been
  released; add a new one. They run automatically at startup, so they must be
  safe on a live database.

## Security rules

- **Never log request bodies or query strings.** SDKs put credentials in both;
  AppSignal embeds its key in the body.
- **Secrets are never stored in plaintext.** Keys, tokens and sessions are
  stored as SHA-256 digests and shown once. Credentials Watchtower must use
  later (Slack webhooks and bot tokens, Linear keys) are sealed with
  `alert.Sealer` (AES-256-GCM under `WATCHTOWER_SECRET_KEY`). Error messages
  and logs never include them.
- **Event content is untrusted.** Anyone who can send an app errors, including
  through a browser SDK's public key, controls titles, messages, frames, tags
  and context. Render it escaped (React, `html/template`), keep it out of email
  headers (`email.oneLine`), and label it as data wherever an agent reads it
  (`mcpserver` briefs).
- **State-changing API endpoints** go through `API.mutation` (JSON only, same

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [olucurious/watchtower](https://github.com/olucurious/watchtower) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-05 -->
