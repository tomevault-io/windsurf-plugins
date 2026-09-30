---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Running tests

```bash
# Python suite (dev task — run as a module, not a published console script)
poetry run python -m scripts.test_local

# Single file / single test
poetry run python -m scripts.test_local test/recorders/test_process_recorder.py
poetry run python -m scripts.test_local test/signals/test_metrics.py::MetricStoreTest::test_add_gauge

# Tests use --forked (each test in a fresh process) and -vv with DEBUG logging by default
```

**CUDA tests:** Any test that touches real CUDA must be marked with `@pytest.mark.cuda`. The default run invokes `pytest-forked`, and a `fork()` after CUDA init corrupts the child's CUDA context; `test/conftest.py` intercepts `cuda`-marked tests and reruns them unforked in a fresh subprocess.

```bash
# Native pure-C++ tests (no GPU; run on macOS or Linux)
make -f Makefile.cupti test-probe test-common test-discovery   # test-discovery: Linux only

# Native GPU tests (Linux + GPU)
make -f Makefile.cupti test-cupti-activity
make -f Makefile.rocm test-rocm-activity ROCM_MAJOR=7
```

```bash
# Everything on the GPU box in one shot: rsync the repo there, run the Python
# suite and the native tests (CUPTI headers auto-downloaded from the matching
# nvidia-cuda-cupti wheel when the toolkit doesn't ship them)
scripts/test-gpu.sh                 # host: $GRAPHSIGNAL_GPU_HOST or dev-sglang-spark-01
scripts/test-gpu.sh --setup         # first run: also installs deps via scripts/init-novenv.sh
scripts/test-gpu.sh --python-only test/recorders/test_shm_recorder.py
scripts/test-gpu.sh --native-only
```

GPU box: `ssh dev-sglang-spark-01` (DGX Spark, arm64, CUDA 13.x). `scripts/test-gpu-native.sh` is the remote half (runs on the box; also usable directly on any GPU machine).

### Build

```bash
poetry install            # deps (use `poetry config virtualenvs.in-project true` for ./.venv)
poetry build              # wheel/sdist
bash scripts/build-protobuf.sh               # regenerate graphsignal/proto/signals_pb2.py

# Native libraries (multi-arch, via docker buildx; copies .so's into graphsignal/_native/)
make -f Makefile.cupti cupti-buildx cupti-buildx-install
make -f Makefile.rocm rocm-buildx rocm-buildx-install
```

## Architecture

The product is a **universal, local profiler for AI inference workloads**. It observes a workload from a sidecar process and exposes everything through a local HTTP endpoint (`GET /signals`, default port 18259) designed for AI-agent consumers. Uploading is an optional addon (see Collector below).

### Two-process model

`graphsignal-run <cmd>` (commands/graphsignal_run.py) consumes its leading flags (`--metrics-port`, `--listen-host`, `--listen-port`, `--cuda-graph-trace`), picks a launcher (vllm/sglang/trtllm/fallback — first `match()` wins), sets up the native injection env vars (`CUDA_INJECTION64_PATH` via profilers/cupti_profiler.py, `ROCP_TOOL_LIBRARIES` via profilers/rocm_profiler.py; `--cuda-graph-trace` becomes `GRAPHSIGNAL_CUDA_GRAPH_TRACE` there too, and the flag beats an inherited value), and hands off to `launchers/supervisor.py::launch_supervised`: the workload runs as a supervised child (tini-style signal forwarding, exact exit-status propagation, console tee to `/dev/shm/graphsignal_log_<pid>/` for LogRecorder), and a **watcher** subprocess (`python -m graphsignal.commands.graphsignal_watch --pid <workload_pid>`) observes it externally. The watcher never shares a process with CUDA.

Launcher argv policy: the workload command passes through byte-for-byte, with exactly two metric-related adjustments — the SGLang launcher appends `--enable-metrics` (its Prometheus endpoint is off by default) and the vLLM launcher strips `--disable-log-stats`.

### Watcher (graphsignal/watcher/)

`graphsignal.watcher.configure()` creates the `Watcher` singleton (access via `graphsignal.watcher.watcher()`; `is_configured()` is the non-raising check). `Watcher.setup()`:

1. Creates the stores: `MetricStore`, `LogStore`, `ResourceStore` (graphsignal/signals/).
2. Sets the `instance.id` tag; global tags are served once as the `/signals` `context` object — stores keep only caller-supplied tags (`process.pid`, `kernel`, `device.uuid`, …) and never touch the singleton.
3. Creates a `Collector` **only if an API key is configured** (api key is optional; env `GRAPHSIGNAL_API_KEY`).
4. Starts the `SignalsEndpoint` (signals/routes.py) on `<listen_host>:<listen_port>` (default `127.0.0.1:18259`) — port conflicts log an error and disable the endpoint, never crash the watcher.
5. Starts a `PidMonitor` (watcher/pid_monitor.py) polling the target and its descendants every 2s, and a 1s tick loop.

On each tick: every recorder's `on_tick()` runs, then the collector's `on_tick()` (when present). On target termination: recorders `finalize()` first (drains a crash's last console lines into the stores), then a final blocking tick, then the watcher process exits.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [graphsignal/graphsignal](https://github.com/graphsignal/graphsignal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
