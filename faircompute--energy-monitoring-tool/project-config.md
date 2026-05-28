---
trigger: always_on
description: This is the maintained project knowledge file for agents and contributors working on EMT. Keep this file concise and current. `CLAUDE.md` should only point here so guidance does not drift across files.
---

# AGENTS.md

This is the maintained project knowledge file for agents and contributors working on EMT. Keep this file concise and current. `CLAUDE.md` should only point here so guidance does not drift across files.

## Project Identity

EMT, the Energy Monitoring Tool, attributes hardware energy use to software workloads at process level. The core value is making energy visible enough for developers, researchers, and platform teams to compare workloads, identify hotspots, and report energy use in shared compute environments.

The shipping product is the Python package in `emt/`. The active strategic work is the Rust collector in `src/`, which should eventually provide the collection backend for the existing Python API through PyO3.

## Stable Product Contract

- Preserve the public Python API unless the task explicitly asks for a breaking change.
- The primary user workflow is still:

  ```python
  from emt import EnergyMonitor

  with EnergyMonitor() as monitor:
      run_workload()

  print(monitor.consumed_energy)
  ```

- `EnergyMonitor` must keep working as a context manager.
- `monitor.consumed_energy` and related total-energy properties must keep working.
- The Python API should be stable while performance-critical collection moves into Rust.

## Architecture Snapshot

### Python path, currently shipping

- `emt/energy_monitor.py` owns `EnergyMonitor`, `EnergyMonitorCore`, and the background monitoring thread.
- `EnergyMonitor` starts a dedicated OS thread so collection can continue while user code runs.
- `EnergyMonitorCore` runs an asyncio loop and gathers `PowerGroup.commence()` tasks.
- `PowerGroup` implementations read energy and utilization data:
  - RAPL CPU package and DRAM energy through `/sys/class/powercap`.
  - NVIDIA GPU energy and memory data through NVML.
- `TraceRecorder` writes traces, currently including CSV and related recorder paths.

### Rust path, actively developed

- `src/` contains the Rust collector and CLI.
- `EnergyGroup<T: EnergyCollector>` owns the generic monitor lifecycle.
- `EnergyCollector` implementations include `Rapl` and `NvidiaGpu`.
- Tokio runs the background monitoring task.
- Batched `EnergyRecord`s move over a bounded `mpsc` channel.
- `RotatingTrace` uses Polars DataFrames to bound trace memory over long sessions.
- The CLI is built from Rust and monitors a PID with arguments like `--pid`, `--duration`, and `--rate`.

### Intended integration

The target integration is PyO3:

- expose Rust collectors through an importable `emt._rust` module;
- let Python `EnergyMonitor` delegate to Rust internally;
- fall back to the Python `PowerGroup` path if the Rust extension is unavailable;
- avoid subprocess and socket overhead for the Python API path.

## Attribution Model

The formulas and semantics must remain equivalent between Python and Rust.

CPU energy attribution:

```text
process_energy = (socket_energy - dram_energy) * normalised_cpu_util
               + dram_energy * memory_util
```

Where:

- `socket_energy` is package RAPL energy delta.
- `dram_energy` is RAPL DRAM energy delta.
- `normalised_cpu_util` is the monitored process or process tree CPU share relative to active system CPU use.
- `memory_util` is the monitored process resident memory share relative to used system memory.

GPU energy attribution:

```text
gpu_process_energy = sum(gpu_zone_energy * process_gpu_memory / total_gpu_memory)
```

GPU memory is currently used as the per-process proxy because NVML does not expose reliable per-process compute energy.

## Verification Rule

Python and Rust must produce equivalent attribution for the same workload before Rust replaces Python collection.

The active parity runner is:

```bash
python scripts/verify.py
```

Useful variants:

```bash
python scripts/verify.py -n 5 -d 10
python scripts/verify.py --iterations 3 --duration 30
```

The active comparison is:

- Python EMT: `emt.EnergyMonitor`
- Rust CLI: `emt`
- Shared workload source: `scripts/verification_workload.py`
- Output: `.artifacts/verification_results.json`

Do not reintroduce the old bash baseline path. Historical notes say `verification/rapl_baseline.sh` and stress helper scripts were removed from the intended verification path.

## Commands

Python setup and checks:

```bash
pip install -e .[dev]
pytest
pytest tests/test_rapl_soc.py
pytest -k "test_name"
black .
coverage run -m pytest && coverage xml
```

Rust setup and checks:

```bash
cargo build
cargo build --release
cargo run -- --pid <PID> --duration 10 --rate 10
cargo test
```

Rust trace rotation tests:

```bash
cargo test --bin emt trace_rotation
```

## Active GitHub Backlog

Open issues as of 2026-05-21 are centered on Rust collector integration:

- #31 `feat: Integrate Rust collector with Python EnergyMonitor via PyO3`
- #32 `Verify Rust Rapl collector accuracy against Python RAPLSoC`
- #33 `Add PyO3 bindings: expose EnergyGroup<T> as emt._rust Python extension module`
- #34 `Verify Rust NvidiaGpu collector accuracy against Python NvidiaGPU`
- #35 `Update EnergyMonitor context manager to delegate to Rust EnergyGroup via PyO3`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FairCompute/energy-monitoring-tool](https://github.com/FairCompute/energy-monitoring-tool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
