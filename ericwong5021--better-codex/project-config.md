---
trigger: always_on
description: - The local Runtime is the only owner of the Better Codex business database. Its runtime lock and published identity must agree on PID, process start time, instance ID, port, version, and profile.
---

# Better Codex runtime invariants

- The local Runtime is the only owner of the Better Codex business database. Its runtime lock and published identity must agree on PID, process start time, instance ID, port, version, and profile.
- Codex token activity is collected by one Runtime-owned resident watcher with incremental rollout reads and periodic reconciliation. Output speed is the weighted `last_token_usage.output_tokens` rate across completed model-output streaming intervals, while request count includes every completed output-bearing model request in the rolling window. Browser and Relay consumers read the cached rolling snapshot and must never create per-client filesystem scanners.
- A Session Host is single-instance within one profile. Runtime connections are authenticated before replacement and fenced by connection epoch. The Host owns one catalog-only App Server and one disposable App Server worker for each Better Codex-owned thread. Host status must continuously identify the Host PID, process start time, Host instance, Runtime instance, catalog App Server, every thread worker, and command state. A thread worker must exit after terminal turn delivery is durably queued or before native Codex resumes that thread; active threads reject handoff instead of sharing a writer.
- Native-thread handoff releases Session Host writer ownership while retaining the Issue-to-thread binding. Card commands for a handed-off thread remain durable Runtime commands and are consumed only by the authenticated native-session proxy in the Codex host; the Session Host must not claim them. Native turns are reconciled from the Runtime-owned rollout watcher so start, completion, interruption, restart recovery, archive protection, and local/Relay projections stay synchronized without a second writer resuming the handed-off thread.
- Runtime rollout reconciliation covers every unarchived Issue-to-thread binding, including idle owned sessions. Persisted active-turn and handoff flags must never exclude a binding from watcher-driven or periodic reconciliation. Reconciliation observes execution without changing writer ownership and records discovered turns with their previous session state and terminal turns with their completion state.
- Stable and development Session Hosts may use separate profile homes, but every detected Host must be represented by a fresh status record. `untracked` Hosts are operational drift and must never be killed without verifying their exact PID, start time, profile lock, and active work.
- Relay accepts one active Runtime connection. Connection-wide closure is reserved for authentication, identity, epoch, or protocol failures. Request sequence and channel state failures terminate only the affected channel and emit structured diagnostics.
- `/livez` proves only that the process can answer. `/readyz` proves that required dependencies can serve traffic. Docker uses `/livez`; external monitoring and remote-access acceptance use `/readyz`.
- Desktop integration is reported separately as ready, waiting for a main window, disabled, or failed. Its evidence is scoped to Runtime instance and generation, compatibility package, profile, and renderer document. Auxiliary windows and loading renderers never establish incompatibility. Compatibility probes do not modify version pointers, and a closed Codex window does not block a service update.
- Standalone VPS mode activates the Compose `standalone` profile and bundled Caddy. Existing-proxy mode starts only `hub` and requires bundled Caddy to be stopped.
- Storage below the critical reserve makes Runtime or Relay not ready and blocks updates before staging. Storage below the warning reserve is reported as degraded and requires cleanup before routine upgrades.
- Errors on Runtime, Session Host, Relay, update, database, and deployment paths must include structured identity and state fields. Do not convert a failed dependency into a successful health response.
- New Issues persist their input before allocating a Codex thread. The first durable start command creates the thread and submits the original input in the same worker; creation alone never proves persistence. Legacy bind commands retain their worker until the returned rollout is materialized. Once a turn has started, the Issue keeps that canonical thread. Agent Issues remain queued until turn start; generated titles use a durable, idempotent owner-routed command.
- Provider, process, and transport failures remain visible on the Run, Reply, and Session records and return the Issue to `blocked`. Failed execution takes precedence over scheduler decisions; `in_review` is reserved for successful execution awaiting review or an unavailable semantic decision. A terminal execution interruption returns the Issue to `blocked`, while a superseded turn remains `in_progress` under its replacement turn.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ericwong5021/better-codex](https://github.com/Ericwong5021/better-codex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
