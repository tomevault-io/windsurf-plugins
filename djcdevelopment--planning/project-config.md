---
trigger: always_on
description: If you're an AI session (Claude Code, another agent, future-me) picking up this project, read this first. It's the fast path to understanding where we are without reading the entire conversation history that produced the current state.
---

# CLAUDE.md — repo-root handoff for Claude sessions

If you're an AI session (Claude Code, another agent, future-me) picking up this project, read this first. It's the fast path to understanding where we are without reading the entire conversation history that produced the current state.

## What this project is (one paragraph)

**Farmer** is a .NET 9 control plane that orchestrates Claude CLI workers on Hyper-V Ubuntu VMs, with a Microsoft Agent Framework (MAF) retrospective agent on the host that reviews every run. NATS JetStream + ObjectStore for coordination, Jaeger for traces, HTTP `/trigger` for ingress, OTel-instrumented throughout. Target audience: Azure/.NET developers learning agent orchestration. The competitive differentiator is using MAF "as much as possible" while keeping workers autonomous on VMs. See [README.md](./README.md) for the full pitch and architecture diagram.

## Current phase state (as of 2026-04-15 session)

Everything below is merged to `main`.

- **Phase 5** shipped: externalized runtime, OTel, real SSH end-to-end verified.
- **Phase 6** shipped: real `worker.sh` + Claude CLI on VM, `RetrospectiveStage` + MAF OpenAI `gpt-4o-mini`.
- **NATS cutover (PR #5)**: file-based `InboxWatcher` retired. Every stage transition publishes a `RunEvent` to the `FARMER_RUNS` JetStream stream; run artifacts upload to the `farmer-runs-out` ObjectStore bucket. See [ADR-010](./docs/adr/adr-010-nats-messaging-cutover.md).
- **Phase 7 retry driver (PR #8)**: opt-in retry via `RetryPolicy` on the `/trigger` body. Driver loops up to `max_attempts` on configured verdicts; each retry gets a synthetic `0-feedback.md` prompt with the prior attempt's `ReviewVerdict.Findings` + `Suggestions`. Chain linked via `parent_run_id`. See [ADR-011](./docs/adr/adr-011-retry-driver.md).
- **VM release fix (PR #10)**: `RunWorkflow.ExecuteAsync` now releases the reserved VM in a `finally` block. Before this, `IVmManager.ReleaseAsync` was never called by anything; in-process retry chains failed at attempt 2's ReserveVm. See [docs/session-retro-2026-04-15.md](./docs/session-retro-2026-04-15.md).
- **IWorkflowRunner seam + real-Retry demo (PR #12)**: `RetryDriver` now depends on an `IWorkflowRunner` interface so its loop is testable (2 integration tests with a `FakeWorkflowRunner`). Also restored cost-report persistence that PR #8 accidentally dropped. New `WORKER_MODE=fake-bad` produces adversarial canned output on the first attempt and clean output on the retry -- the loop fires on real `Retry`/`Reject` verdicts instead of a contrived `retry_on_verdicts: ["Accept"]`. See [docs/retry-demo-2026-04-16.md](./docs/retry-demo-2026-04-16.md).
- **Tests**: 133 green (128 unit + 5 integration with NatsServerFixture).

## The plan file for the active session

Plans live at `C:\Users\Derek\.claude\plans\robust-forging-iverson.md` (outside the repo -- session artifact, not tracked in git). The commit log is authoritative when the plan file disagrees.

## Runtime directory (NOT in git)

Runtime state lives at `C:\work\iso\planning-runtime\`, deliberately outside the repo. Path is configurable via `Farmer:Paths` in `appsettings.Development.json`. Structure:

```
C:\work\iso\planning-runtime\
├── data\sample-plans\     ← worker inputs (copied from repo data/sample-plans/)
├── runs\{run_id}\         ← immutable after completion, one dir per run
├── nats\                  ← JetStream store_dir (FARMER_RUNS stream + farmer-runs-out bucket)
├── outbox\
├── qa\
```

## Registered ports

| Service | Port | Status |
|---|---|---|
| `farmer/api` | 5100 | dotnet, HTTP (`scripts/dev-run.ps1`) |
| `nats-server` | 4222 | NATS core (`infra/start-nats.ps1`) |
| `nats-monitoring` | 8222 | NATS HTTP monitoring |
| `jaeger/otlp-grpc` | 4317 | Jaeger OTLP ingest (`infra/start-jaeger.ps1`) |
| `jaeger/ui` | 16686 | Jaeger UI (http://localhost:16686) |

## Common gotchas

- **SSH key path.** `FarmerSettings.SshKeyPath` defaults to `id_ed25519`. The legacy `id_rsa` is encrypted with a passphrase on this machine, and `Renci.SshNet.PrivateKeyFile` can't talk to `ssh-agent`. Don't "fix" this by changing to `id_rsa` — it will fail at `Deliver` stage with `SshPassPhraseNullOrEmptyException`.
- **SSH uses absolute paths for SCP destinations.** `Renci.SshNet`'s `ScpClient` does NOT expand `~`. The VM config uses `/home/claude/projects` as `RemoteProjectPath`, not `~/projects`. See [commit `a3c3c5b`](../git) on the verification branch for history.
- **Mapped drive is read-only from Windows.** Writes to VM always go through SSH/SCP. Reads come through the mapped drive (`O:\projects\` for `claudefarm2`). There's a ~500ms SSHFS cache lag that `MappedDriveReader` handles with a retry loop.
- **`CollectStage` rejects empty `files_changed`**. Phase 5 fake workers used sentinel strings (`FAKE_WORKER_NO_REAL_CHANGES`) to satisfy this. Phase 6 workers will populate `Manifest.Outputs[]` AND leave `files_changed` non-empty for back-compat. See [ADR-003](./docs/adr/adr-003-anti-drift-contract.md) and `CollectStage.cs`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [djcdevelopment/planning](https://github.com/djcdevelopment/planning) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
