---
trigger: always_on
description: Orientation for AI agents working in the GEOPM (Global Extensible Open Power
---

# GEOPM — Agent Navigation Guide

Orientation for AI agents working in the GEOPM (Global Extensible Open Power
Manager) repository. Read this before searching the tree; it tells you where
things live, which components matter most, and how to build, test, and verify
changes.

## What GEOPM is

GEOPM provides **fine-grained, low-latency, access-controlled reads and writes
of platform power/energy/frequency metrics and control knobs on Linux**. The
distinguishing feature is *safe, session-scoped hardware control for
unprivileged users*: a user may change a hardware setting, and the **GEOPM
Access Service** (`geopmd`) automatically restores the prior value when the
user's session ends.

Two major components:

| Component | Purpose | C/C++ | Python |
|---|---|---|---|
| **GEOPM Access Service** | Privileged daemon exposing signals/controls with per-user access control | [libgeopmd](libgeopmd) | [geopmdpy](geopmdpy) |
| **GEOPM Runtime Service** | Unprivileged HPC control framework (agents, MPI profiling) | [libgeopm](libgeopm) | [geopmpy](geopmpy) |

**Focus your attention on the Access Service side** (`libgeopmd` + `geopmdpy`).
That is where the primary supported use cases live. See
[Runtime Service transition](#runtime-service-transition) for how to handle the
HPC side.

## Primary use cases

Most user requests map to one of these `geopmdpy` command line tools. Prefer
these over library-level work unless the task explicitly requires C/C++ changes.

| Tool | Implementation | Man page | Use it for |
|---|---|---|---|
| `geopmread` | [geopmdpy/geopmdpy/read.py](geopmdpy/geopmdpy/read.py) | [geopmread.1.rst](docs/source/geopmread.1.rst) | One-shot read of a signal at a topology domain; discover available signals |
| `geopmwrite` | [geopmdpy/geopmdpy/write.py](geopmdpy/geopmdpy/write.py) | [geopmwrite.1.rst](docs/source/geopmwrite.1.rst) | Write a control (power cap, frequency limit); discover available controls |
| `geopmsession` | [geopmdpy/geopmdpy/session.py](geopmdpy/geopmdpy/session.py) | [geopmsession.1.rst](docs/source/geopmsession.1.rst) | Time-series traces, YAML/CSV summary reports, control application for the duration of a run, daemonized collection, Python "agents" |
| `geopmopt` | [geopmdpy/geopmdpy/optimizer.py](geopmdpy/geopmdpy/optimizer.py) | [geopmopt.1.rst](docs/source/geopmopt.1.rst) | Bayesian optimization of control settings against an application objective |
| `geopmgrid` | [geopmdpy/geopmdpy/grid.py](geopmdpy/geopmdpy/grid.py) | [geopmgrid.1.rst](docs/source/geopmgrid.1.rst) | Exhaustive parameter sweeps; defines the control grid `geopmopt` searches |
| `geopmaccess` | [geopmdpy/geopmdpy/access.py](geopmdpy/geopmdpy/access.py) | [geopmaccess.1.rst](docs/source/geopmaccess.1.rst) | Administrator management of per-user/per-group signal and control allow lists |
| `geopmexporter` | [geopmdpy/geopmdpy/exporter.py](geopmdpy/geopmdpy/exporter.py) | [geopmexporter.1.rst](docs/source/geopmexporter.1.rst) | Prometheus metric export |
| `geopmd` | [geopmdpy/geopmdpy/\_\_main\_\_.py](geopmdpy/geopmdpy/__main__.py) | — | The Access Service daemon itself |

Console-script entry points are declared in
[geopmdpy/setup.cfg](geopmdpy/setup.cfg) — check there first when asked "where
does command X live?".

### Representative commands

```bash
geopmread CPU_POWER board 0                       # total board CPU power, Watts
geopmread --info-all                              # every signal, with description
geopmwrite CPU_FREQUENCY_MAX_CONTROL board 0 3.0e9

# 10 s trace at 1 Hz of time and per-package core frequency
printf 'TIME board 0\nCPU_FREQUENCY_STATUS package 0\n' | geopmsession -p 1.0 -t 10.0

# Trace + YAML summary report scoped to an application run
printf 'TIME board 0\nCPU_ENERGY package *\n' | \
    geopmsession -o trace.csv -r report.yaml -p 0.01 -- ./workload.sh

# Search CPU and uncore frequency for the best figure of merit
geopmopt --sweep cpu-freq@board --sweep uncore-freq@board \
         --metric-regex 'GFLOPS: ([0-9.]+)' --trials 30 -- ./bench.sh
```

## Core concepts

Understanding these four ideas is enough to reason about most of the codebase.

- **Signal** — a readable value (`CPU_POWER`, `CPU_ENERGY`,
  `CPU_FREQUENCY_STATUS`, `TIME`, `GPU_POWER`, ...).
- **Control** — a writable setting (`CPU_POWER_LIMIT_CONTROL`,
  `CPU_FREQUENCY_MAX_CONTROL`, ...).
- **Domain** — the topology scope of a signal/control: `board`, `package`,
  `core`, `cpu`, `memory`, `gpu`, `nic`, etc. Every read/write names a domain
  type *and* a domain index. `*` means "native domain" (2nd field) or "all
  indices" (3rd field). See [geopm_topo.3.rst](docs/source/geopm_topo.3.rst).
- **IOGroup** — the plugin abstraction that supplies signals/controls to
  `PlatformIO`. Every hardware backend (MSR, sysfs, SST, NVML, DCGM,
  LevelZero, ...) is an `IOGroup`. See [geopm_pio.7.rst](docs/source/geopm_pio.7.rst)
  and the per-backend `docs/source/geopm_pio_*.7.rst` pages.

Signal and control *names* are the stable public API; treat renames as breaking
changes.

## Safety and the access model

Hardware writes are mediated — do not reason about them as raw MSR pokes.

- **`geopmd` is the safety mechanism.** Unprivileged users reach controls only
  through the Access Service, never directly.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [geopm/geopm](https://github.com/geopm/geopm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
