---
trigger: always_on
description: `mail-time` NPM library. Email queue + sender for horizontally scaled Node.js & Bun apps. Synchronizes the queue across processes via Redis / MongoDB / PostgreSQL / custom adapter. Built on top of [`josk`](https://github.com/veliovgroup/josk) for scheduling and task management, and `nodemailer` for email transport management.
---

# AGENTS.md

`mail-time` NPM library. Email queue + sender for horizontally scaled Node.js & Bun apps. Synchronizes the queue across processes via Redis / MongoDB / PostgreSQL / custom adapter. Built on top of [`josk`](https://github.com/veliovgroup/josk) for scheduling and task management, and `nodemailer` for email transport management.

## Mission
Send and queue emails in horizontally scaled Node.js and Bun.js. Bulletproof. High perf. Storage agnostic. Two roles: `server` (drains + sends) and `client` (enqueues). Many clients + many servers behind one `prefix`.

## Topology / tuning (read README for use-cases)

### Multiple instances — encouraged
- **One `MailTime` per email class** when policies differ (`otp`, `transactional`, `marketing`, …): own options; **own `prefix` only when purpose/settings differ**.
- **`prefix` same** for all `client` + `server` on one logical queue (app enqueues `prefix: 'otp'`, mail worker drains `prefix: 'otp'`).
- **Never** same `prefix` on two instances with different mail policy (concat, retries, etc.).
- App pods: `type: 'client'`. Mail VM: `type: 'server'` (systemd: one unit per class, e.g. `mailtime@otp`).

### One mail host: 2–8 servers
- Run **2–8 `server` instances** on one machine (~**1 per CPU core**) for **parallel drains across prefixes**, not duplicate drains of same `prefix`.
- Same `prefix` cluster-wide = **one JoSk lease tick** at a time → extra pods ≠ N× throughput, but **do** buy failover/HA (warm standby with a different `lockOwnerId` takes the lease the next tick if the winner dies).
- High volume one queue → **shard prefixes** (`marketing-0`, …), not duplicate instances same `prefix`.

### Throughput levers
| Lever | Effect |
|---|---|
| More `prefix`es / instances | More parallel drain loops |
| `revolvingInterval` ↓, `josk.min/maxRevolvingDelay` ↓ | Faster pickup, more storage I/O |
| Dedicated mail workers | SMTP + tuning isolated from app |
| `concurrency: N` (MailTime) | N parallel SMTPs per server within a single instance |
| `mode: 'one'` (MailTime) | One row claimed per tick — fairness across nodes; storage CAS prevents dupes |
| `josk.concurrency: 1` | No overlapping `iterate` on one process |

### MailTime defaults (override in `opts` / `opts.josk`)
| Knob | Default | Tune |
|---|---|---|
| `mode` | `'batch'` | `'one'` to claim a single row per tick (fairness over throughput) |
| `concurrency` | `1` | N parallel SMTPs per instance. CAS guard on `isSending` prevents double-send. |
| `sendingTimeout` | 300000 (5 min) | Stale-lock recovery. Must exceed slowest legitimate SMTP roundtrip. |
| `revolvingInterval` | 1536 | Latency vs I/O |
| `josk.min/maxRevolvingDelay` | 512 / 2048 | Poll jitter (MailTime overrides JoSk 128/768) |
| `josk.zombieTime` | 60000 | **≥60s**. `___iterate` releases the JoSk lease as soon as scan completes, so zombies are rare unless storage itself stalls. |
| `josk.execute` | `'batch'` | Usually leave; one JoSk uid per instance |
| `josk.concurrency` | `Infinity` | `1` if ticks overlap |
| `josk.lockOwnerId` | random | **Set prod** (`hostname-pid`, pod name) |
| `retries` / `retryDelay` | `60` / `60s` (or `maxTries` 60 if unset) | Per class; OTP short, marketing long |
| `concatEmails` / `concatDelay` | false / 60s | Marketing on; OTP off |

### Per-row lifecycle (`isSending` lock)
1. `___iterate` (the JoSk handler) calls `queue.iterate({ limit, sendingTimeout })`.
2. Per due-and-unclaimed row, the adapter calls `await mailTimeInstance.___dispatch(row)`. `___dispatch` waits for a free pool slot, then starts the full send lifecycle in the background and returns — releasing iterate to move on.
3. The background lifecycle (`___send`) does an atomic CAS: set `isSending=true, sendingAt=now, tries=tries+1` only if `isSent=false AND isFailed=false AND isCancelled=false AND tries=task.tries AND (isSending=false OR sendingAt<=now-sendingTimeout)`. CAS losers drop silently.
4. SMTP runs.
5. On success the row is removed (or `isSent=true, isSending=false, sendingAt=0` with `keepHistory`). On retry, `isSending=false, sendingAt=0, sendAt=now+retryDelay`. On final failure, `isFailed=true, isSending=false`.
6. If a worker dies between step 3 and step 5, the row stays `isSending=true` until `sendingAt + sendingTimeout` elapses — then the iterate predicate makes it eligible again and the CAS allows a new worker to claim it.

### Presets — use `mailTimePreset(name, overrides)`
Recommend `mailTimePreset(name, overrides)` (exported from `mail-time`) before hand-tuning. Names: `transactional`, `otp`, `newsletter`, `marketing`, `notifications`, `alerts`. Each is a partial MailTime config; the function deep-clones and deep-merges overrides (scalars win, `josk` composes). User still supplies `queue` / `transports` / `josk.adapter` / `prefix`. Source: `presets.js`. See README §"Settings presets" for the table.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [veliovgroup/mail-time](https://github.com/veliovgroup/mail-time) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
