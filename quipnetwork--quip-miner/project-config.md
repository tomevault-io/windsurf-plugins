---
trigger: always_on
description: Cross-tool instructions for AI coding assistants (Claude Code, Codex, Cursor, Gemini CLI).
---

# AGENTS.md — QuIP Protocol Project Instructions

Cross-tool instructions for AI coding assistants (Claude Code, Codex, Cursor, Gemini CLI).

For the **runtime architecture**, read [`ARCHITECTURE.md`](./ARCHITECTURE.md).
This file is the developer-facing how-to: commands, dependencies, code
style. Anything about how the system *runs* belongs in
`ARCHITECTURE.md`.

## Environment

The `.quip` virtualenv is already active in development shells —
don't prefix shell commands with `source .quip/bin/activate`.

```bash
# Fresh install (only if .quip doesn't exist yet)
python3 -m venv .quip
source .quip/bin/activate
pip install -U pip setuptools wheel
pip install -e .            # core + CPU
pip install -e .[cuda]      # CUDA backend
pip install -e .[metal]     # Apple Silicon backend
pip install -e .[dev]       # pytest + pytest-asyncio
```

`.env` holds `DWAVE_API_KEY` and other credentials. **Never read or
display its contents.**

## Running the miner

The CLI is `quip-miner` (defined in `quip_cli.py`, entry point in
`pyproject.toml`). It attaches to a substrate validator over WS or
HTTP; there is no longer any in-process P2P node to run.

```bash
# Generate a hybrid sr25519 + ML-DSA-44 keystore
quip-miner keygen --out ~/.quip-miner/signing.json

# Bootstrap (one-shot reachability + funding check against a validator)
quip-miner bootstrap --validator ws://127.0.0.1:9944 \
  --signer-key ~/.quip-miner/signing.json

# CPU PoW miner
quip-miner cpu --validator ws://127.0.0.1:9944 \
  --signer-key ~/.quip-miner/signing.json

# CUDA / Metal / D-Wave
quip-miner gpu --validator ws://127.0.0.1:9944 --gpu-backend local --signer-key ...
quip-miner gpu --validator ws://127.0.0.1:9944 --gpu-backend metal --signer-key ...
quip-miner qpu --validator ws://127.0.0.1:9944 --daily-budget 30s --signer-key ...

# Multiple CPU workers (PoW + mempool jobs share the same workers)
quip-miner cpu --validator ws://... --num-cpus 4 --signer-key ...

# TOML config (see docker/quip-miner.cpu.toml, docker/quip-miner.cuda.toml)
quip-miner cpu --config ./docker/quip-miner.cpu.toml

# Production: run everything the config declares, supervised
quip-miner --config ./docker/quip-miner.cpu.toml

# Narrow a multi-backend config to one miner type (CLI-only; the
# supervisor echoes which configured types were dropped)
quip-miner --config config.toml --mode gpu
```

Miner-type selection is CLI-only: the supervisor's `--mode cpu|gpu|qpu`
keeps one configured type (warning about the dropped ones), and a
direct `quip-miner cpu|gpu|qpu` run does the same narrowing with the
same warning. There is no config key for it — a legacy `[miner] mode`
key still loads but is ignored.

**Mempool participation is config-only and per-miner** — `mempool` is
an unquoted TOML bool set INSIDE each backend section
(`[cpu] mempool = false`, `[gpu]`/`[metal]`/`[modal]`, or a qpu vendor
section like `[dwave] mempool = true`); defaults: cpu/gpu on, qpu off —
paid QPU samples are opt-in. A `mempool` key in `[miner]` is rejected
at load time; `[miner] mempool_min_reward` (0 = accept all) stays
global. There is no CLI flag for it and no mempool-only mode (`--mode`
selects miner types, not the work source): every worker mines PoW
continuously, mempool jobs preempt PoW on the same workers, and PoW
resumes afterward. Solver
registration is automatic at miner startup (query-first, never
auto-deregisters — switching solver type requires an explicit
`quip-miner deregister-solver` and restart). A mempool-fatal submit
receipt parks the mempool side for the run while PoW mining continues.
On a multi-backend config the mempool owner is derived from the
per-section keys: an explicit `mempool = true` outranks default-on
groups, then the first default-on group in canonical cpu,gpu,qpu order
owns; every other child resolves mempool off from the same TOML (one
substrate account can only register one solver type on chain). Set
`mempool = false` in a section to move ownership to the next group.
Nothing is transported out-of-band, so supervised, direct-subcommand,
and `--mode`-narrowed runs all agree; the supervisor echoes the
election so operators see why a child is pow-only.

Live integration uses the docker-compose validator under `docker/`
(`docker compose up quip-validator`); the validator listens on
`ws://127.0.0.1:9944` by default.

**Metal interactive cap (Apple Silicon):** the `[metal]` section runs an
adaptive governor when `yielding` is on (default). It senses HID-idle /
thermal / battery / displays and caps **GPU occupancy** — the jank lever is
concurrent threads per command buffer (`problems × reads`), not core count or
duty cycle. While you're present it splits reads so each command buffer stays
under `active_util` % of the GPU's max thread capacity
(`maxTotalThreadsPerThreadgroup × cores`; default 85); idle/headless runs
uncapped (full speed); thermal-serious halves it; battery / critical thermal
pause. Total reads and sweeps are always preserved. (On an M4 Max steady-state
mining is smooth even at full saturation, so the cap is mainly insurance for
weaker GPUs / sustained thermal load.) This path
is **independent of the CUDA util monitor** — it lives in
`GPU/metal_scheduler.py` + `GPU/macos_sensors.py` and shares no utilization

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [QuipNetwork/quip-miner](https://github.com/QuipNetwork/quip-miner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-10 -->
