---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Package Management
- `uv sync` - Install dependencies
- `uv add <package>` - Add new dependency
- `uv run <command>` - Run commands in virtual environment

### CLI Usage

#### Job Management (SLURM-aligned commands)
Command names mirror SLURM's CLI where it makes sense (`sbatch` / `squeue` /
`scancel` / `sinfo`) so a SLURM user can map their muscle memory directly.
srunx-specific commands that don't map to a SLURM binary:

- `history` — srunx's own submission history (SQLite-backed). Named
  `history` rather than `sacct` because it shares no backend with real
  `sacct`; only jobs submitted via srunx are listed.
- `gpus` — GPU aggregate summary across partitions.
- `tail` / `watch` — log tailing and cluster watching.

- `uv run srunx sbatch <script>` - Submit a sbatch script (positional, like real sbatch)
- `uv run srunx sbatch --wrap "cmd ..."` - Wrap a command into a SLURM job (mutually exclusive with the positional script)
- `uv run srunx sbatch --profile <name> ...` - Submit over SSH to a configured profile
- `uv run srunx squeue` - List active jobs (all users by default — matches native `squeue`). Default columns: Job ID, User, Name, Status, GPUs, Elapsed, NodeList
- `uv run srunx squeue -j <job_id>` - Filter to a specific job (replaces the old `srunx status` for active jobs)
- `uv run srunx squeue -u <user>` - Filter to a single user
- `uv run srunx squeue --show-partition --show-cpus --show-limit --show-nodes` - Opt-in columns (each flag adds one)
- `uv run srunx squeue -a` - Shortcut for all opt-in columns at once
- `uv run srunx squeue -i <seconds>` - Live refresh: re-query and redraw in place every N seconds (like native `squeue -i`). Ctrl+C exits
- `uv run srunx squeue --format json` - JSON always includes every field regardless of show flags
- `uv run srunx scancel <job_id>` - Cancel a job
- `uv run srunx sinfo` - Partition / state / nodelist listing (same columns as native SLURM `sinfo`)
- `uv run srunx sinfo --profile <name>` - Query a remote cluster via SSH adapter (#139)
- `uv run srunx sinfo --partition gpu --format json` - Partition rows as JSON
- `uv run srunx gpus` - GPU aggregate summary (what the old `srunx sinfo` showed)
- `uv run srunx gpus --profile <name> --partition gpu` - GPU summary scoped to one partition on a remote cluster
- `uv run srunx history` - DB-backed submission history (srunx's own SQLite). Only jobs submitted via srunx are listed
- `uv run srunx history -j <job_id>` - Filter history to a specific job (replaces `srunx status` for finished jobs)
- `uv run srunx history --profile <name>` - Scope history to a single cluster's jobs (`scheduler_key` filter)
- `uv run srunx sacct` - Real SLURM `sacct` wrapper (cluster accounting DB — includes manual sbatch jobs, needs slurmdbd). Default columns: Job ID, User, Name, Partition, State, ExitCode, Elapsed
- `uv run srunx sacct -a -S now-1day` - All users over the last day (like native `sacct -a -S now-1day`)
- `uv run srunx sacct -j <job_id>` / `-u <user>` / `-s FAILED,TIMEOUT` / `-p <partition>` - Standard sacct filters
- `uv run srunx sacct --show-steps` - Include `.batch` / `.extern` sub-step rows
- `uv run srunx tail <job_id>` - Show the last 10 lines of the job log (matches native `tail`; use `-n N` for a different cap, `--all` to dump the whole file)
- `uv run srunx tail <job_id> --follow` - Stream job logs. Works over SSH via periodic `tail_log_incremental` polls; tune with `--interval <seconds>` (default 2s). Ctrl+C exits

`sbatch` accepts the standard SLURM short flags (`-J` / `-N` / `-n` / `-c` /
`-t` / `-p` / `-w` / `-D`) and `--gres=gpu:N`. srunx-specific extensions
(`--profile` / `--conda` / `--venv` / `--container` / `--template` / etc.)
layer on top. `status` was intentionally dropped — use `squeue -j <id>` for
active jobs or `history -j <id>` for finished jobs.

##### Auto-sync + in-place execution

When the positional script lives under one of the SSH profile's mounts
(`mount.local`), srunx:

1. **Auto-rsyncs** that mount to the remote (`mount.remote`) under a
   per-mount file lock (default ON; opt out with `--no-sync` or
   `[sync] auto = false`).
2. Translates the script path to its remote equivalent and invokes
   `sbatch` **directly on the remote file** — no tmp copy, no
   ``-o $SLURM_LOG_DIR/%x_%j.log`` auto-injection, your script's own
   `#SBATCH` directives win.
3. ``cd``s into the script's mount-translated parent directory before
   sbatch, so relative paths inside the script (e.g.
   ``#SBATCH --output=./logs/%j.out``) resolve where you'd expect.

Generated artifacts (``--wrap``, ``--template``, workflow ShellJobs
with Jinja substitution) always go through the historical
``$SRUNX_TEMP_DIR`` upload path because the rendered bytes have no
canonical home in the mount.

Sync defaults are configured under `[sync]` in
`~/.config/srunx/config.json`:

```json
{
  "sync": {
    "auto": true,
    "lock_timeout_seconds": 120,
    "warn_dirty": true,
    "require_clean": false
  }
}
```

Per-invocation overrides:

- `srunx sbatch --sync` / `--no-sync`
- `SRUNX_SYNC_AUTO=0` / `SRUNX_SYNC_REQUIRE_CLEAN=1` etc.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ksterx/srunx](https://github.com/ksterx/srunx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
