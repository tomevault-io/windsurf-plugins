---
trigger: always_on
description: This repository uses LabVIEW, g-cli, and PowerShell tooling. Follow the steps below so the agent can run tests and local CI parity safely.
---

# Local Agent Instructions

This repository uses LabVIEW, g-cli, and PowerShell tooling. Follow the steps below so the agent can run tests and local CI parity safely.

## Prerequisites
- Windows with PowerShell 7+ available as `pwsh`.
- `g-cli` available on PATH.
- LabVIEW 2021 (21.0) 32-bit and 64-bit installed.
- VIPM/VIPC installed (required for dependency application).

## Repo Setup
- Open a PowerShell terminal at the repo root.
- Confirm `g-cli` is available:
  - `g-cli --version`

## Worktree root (short paths)
Use a short path for worktrees to avoid Windows path-length issues. Default to `C:\dev` for local dev; for self-hosted runners, standardize under the runner directory (example: `C:\actions-runner\_work\lvie\w`).

Override:
- Set `LVIE_WORKTREE_ROOT` to change the default worktree root.
  - Runner contract helper: `pwsh -NoProfile -File .\Tooling\Setup-Runner.ps1 -RunnerRoot C:\actions-runner -Scope Machine` (creates `<runner-root>\_work\lvie\w`, writes `<runner-root>\_work\lvie\runner-contract.json`, and sets env vars).

Preflight requirement:
- If the chosen worktree root does not exist, ask the user to create it before proceeding.
- For CI/self-hosted runners, ensure the directory is pre-created; fail fast with a clear message if missing.
 - Local parity scripts hard-fail if `RepoRoot` is not under the worktree root; set `LVIE_WORKTREE_ROOT` or run from a worktree path.

Example preflight (PowerShell):
```
$worktreeRoot = $env:LVIE_WORKTREE_ROOT
if ([string]::IsNullOrWhiteSpace($worktreeRoot)) { $worktreeRoot = 'C:\dev' }
if (-not (Test-Path $worktreeRoot)) {
  throw "Worktree root '$worktreeRoot' does not exist. Create it or set LVIE_WORKTREE_ROOT."
}
```

Worktree creation helper (recommended):
```
pwsh -NoProfile -File .\Tooling\New-CIWorktree.ps1 `
  -Ref HEAD
```

Notes:
- The helper enforces the worktree root and fails fast if it is missing.
- Use `-Name` to label the worktree directory.
- Use `-WorktreeRoot` (or `LVIE_WORKTREE_ROOT`) to override the default.

## CI worktree naming (ci-composite.yml)
CI jobs create short-path worktrees under `LVIE_WORKTREE_ROOT` with a deterministic name:
- `ci-<jobhash>-<bitness>-<runid>-<attempt>`
- Some workflows insert an extra variant token (e.g. LabVIEW version) between `<jobhash>` and `<bitness>`.
- `jobhash` is the first 8 chars of the SHA1 of `GITHUB_JOB` (prevents collisions across jobs).
- `bitness` is `32` or `64`.
Example: `C:\dev\ci-D170BDEE-64-21534416929-1`

The workflow exports:
- `REPO_ROOT` → worktree path (authoritative for all scripts)
- `PROJECT_PATH` → `$REPO_ROOT\lv_icon_editor.lvproj`
- `LABVIEW_VERSION_YEAR` / `LABVIEW_MINOR_REVISION` → derived from `.lvversion` (e.g., `21.0` → `2021` and minor `0`)

Note: CI reads `.lvversion` from `REPO_ROOT` as the canonical LabVIEW version for runs.

Helper used by CI:
```
pwsh -NoProfile -File .\Tooling\New-CIWorktreeForJob.ps1 -Bitness 64
```

## CI concurrency (self-hosted LabVIEW runners)
LabVIEW workflows are serialized on the shared self-hosted runner label to avoid concurrent g-cli/LabVIEW conflicts.

Notes:
- Workflows share a concurrency group keyed by repository + runner label (e.g., `labview-<repo>-self-hosted-windows-lv-ie`).
- Missing-in-project is inlined in `ci-composite.yml` to avoid reusable workflow skips; the standalone workflow is manual only.
- Self-hosted LabVIEW jobs acquire a runner lock at `<lock_root>\labview-runner.lock` via `Tooling\RunnerLock.ps1`. The lock auto-expires stale entries (lease + optional GitHub run status check) and logs owner metadata. Env overrides: `LVIE_LOCK_ROOT`, `LVIE_RUNNER_LOCK_TIMEOUT_SECONDS`, `LVIE_RUNNER_LOCK_LEASE_SECONDS`, `LVIE_RUNNER_LOCK_STALE_SECONDS`, `LVIE_RUNNER_LOCK_GITHUB_CHECK`, `LVIE_RUNNER_LOCK_GITHUB_MIN_AGE_SECONDS`, `LVIE_RUNNER_LOCK_GITHUB_CHECK_INTERVAL_SECONDS`.

## Local CI Parity (recommended)
Run the local parity script that mirrors `ci-composite.yml`:
```
pwsh -NoProfile -File .\Tooling\Run-CICompositeLocal.ps1 `
  -LabVIEWVersion 2021 `
  -EnsureCleanState
```

Notes:
- Outputs go to `$WORKTREE_ROOT\artifacts\<runid>\ci-local` when guardrails are active (default for local runs).
- GitHub Actions disables artifact roots by default unless `LVIE_ENABLE_ARTIFACT_ROOT=1` or an explicit `-RunId`/`-ArtifactRoot` is passed.
- The script always runs both 64-bit and 32-bit steps for LabVIEW 2021 (21.0).
- The script handles Verify IE Paths, VIPC, missing-in-project, unit tests, PPL builds, and VIP build.
- If LabVIEW or g-cli is already running, the script waits for them to exit before starting.
- You can skip steps with switches like `-SkipBuildVip` or `-SkipUnitTests`.
- VIP builds flow through `Tooling\Invoke-VipBuild.ps1`, which emits `builds\status\vip-build.json` and respects `LVIE_VIPM_TIMEOUT_SECONDS`, `LVIE_VIPM_MAX_ATTEMPTS`, and `LVIE_VIPM_RETRY_DELAY_SECONDS`.

## Adaptive timeouts and continuous troubleshooting
Use fixed timeouts for deterministic CI runs. Use adaptive timeouts only for local/manual runs while tuning.

**CI guidance (deterministic):**
- Set conservative fixed values and keep them stable across runs.
- Prefer failing on the status-file contract over wall-clock timing when possible.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ni/labview-icon-editor](https://github.com/ni/labview-icon-editor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
