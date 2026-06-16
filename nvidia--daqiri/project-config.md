---
trigger: always_on
description: This file provides guidance to coding agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to coding agents when working with code in this repository.

## Build & run

```bash
# Configure and build
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=ON -DDAQIRI_BUILD_PYTHON=OFF -DDAQIRI_ENGINE="dpdk ibverbs"
cmake --build build -j
cmake --install build --prefix /opt/daqiri

# Container build (compiles patched DPDK from source)
BASE_TARGET=dpdk DAQIRI_ENGINE="dpdk ibverbs" scripts/build-container.sh
```

CMake options (full table in `docs/getting-started.md`):
- `DAQIRI_ENGINE` — space-separated list of optional engines to compile. Valid values: `dpdk` (raw Ethernet) and `ibverbs` (RDMA/RoCE). Linux sockets (UDP/TCP) are always built in, so there is no `socket` value. Default is `"dpdk ibverbs"`.
- `DAQIRI_BUILD_PYTHON` — builds `pybind11` bindings from `python/`.
- `DAQIRI_BUILD_EXAMPLES` — builds the benchmark executables (default `ON`).
- `DAQIRI_ENABLE_OTEL_METRICS` — enables OpenTelemetry metrics instrumentation (default `OFF`).
- `DAQIRI_REORDER_GPU_PROFILE` — enable CUDA event timing in the DPDK reorder kernels (off by default).
- `DAQIRI_ENABLE_S3` — enable AWS SDK-backed asynchronous raw packet writes to S3 (off by default).
- `DAQIRI_PREFER_SYSTEM_YAML_CPP` — prefer system `yaml-cpp` over the vendored `third_party/yaml-cpp` submodule (off by default; keep off when a conda/miniforge env is on `PATH`).

CUDA architectures default to `80;90` (A100, H100), with `121` (GB10) added when configuring with CUDA Toolkit 13.0 or newer. Override `CMAKE_CUDA_ARCHITECTURES` when targeting other GPUs.

**Socket / ibverbs relationship**: the socket engine is always built and provides UDP/TCP directly; its RoCE path delegates to the `ibverbs` engine (internally the `rdma` engine — `src/engines/rdma/`, target `daqiri_rdma`, define `DAQIRI_ENGINE_RDMA`; `ibverbs` is only the user-facing name). `src/CMakeLists.txt` builds the ibverbs/rdma engine only when `ibverbs` is in `DAQIRI_ENGINE`, and the socket engine links it conditionally — so `socket` RoCE is available only when `ibverbs` was built, while plain UDP/TCP always works.

## Benchmarks (the "tests")

There is no unit test suite. Verification is done via the benchmark executables in `examples/`, driven by YAML configs. Build outputs (`examples/CMakeLists.txt:59-71`):

| Executable | Source | Typical config |
|---|---|---|
| `daqiri_bench_raw_gpudirect` | `raw_gpudirect_bench.cpp` | `daqiri_bench_raw_tx_rx.yaml`, `daqiri_bench_raw_tx_rx_4q.yaml`, `daqiri_bench_raw_tx_rx_spark.yaml`, `daqiri_bench_raw_{tx,rx}_spark_xhost.yaml`, `daqiri_bench_raw_sw_loopback.yaml`, `daqiri_bench_raw_rx_multi_q.yaml`, `daqiri_bench_raw_tx_rx_spark_mq.yaml` (mq base; `run_spark_mq_bench.sh` derives the 4 cells via `scripts/gen_spark_mq_config.py`), `daqiri_bench_raw_tx_rx_pacing.yaml` (per-queue `pacing_mbps`; DPDK engine only) |
| `daqiri_example_dynamic_rx_flow` | `dynamic_rx_flow_example.cpp` | `daqiri_example_dynamic_rx_flow.yaml` — queues-only `flow_isolation: true` startup followed by runtime RX flow add/delete |
| `daqiri_bench_raw_hds` | `raw_hds_bench.cpp` | `daqiri_bench_raw_tx_rx_hds.yaml` |
| `daqiri_bench_raw_reorder_seq` | `raw_reorder_seq_bench.cpp` | `daqiri_bench_raw_tx_rx_reorder_seq_1024*.yaml`, `daqiri_bench_raw_rx_reorder_seq_*.yaml` |
| `daqiri_bench_raw_reorder_quantize` | `raw_reorder_quantize_bench.cpp` | `daqiri_bench_raw_tx_rx_reorder_quantize_seq_batch.yaml` |
| `daqiri_bench_rdma` | `rdma_bench.cpp` | `daqiri_bench_rdma_tx_rx.yaml`, `daqiri_bench_rdma_tx_rx_spark.yaml`, `daqiri_bench_rdma_tx_rx_spark_xhost.yaml`, `daqiri_bench_rdma_tx_rx_spark_netns.yaml` (combined-role netns base; `run_spark_bench.sh` splits per role via `scripts/gen_spark_netns_config.py`) |
| `daqiri_bench_socket` | `socket_bench.cpp` | `daqiri_bench_socket_{udp,tcp}_tx_rx.yaml`, `daqiri_bench_socket_{udp,tcp}_tx_rx_spark_netns.yaml` (combined-role netns bases) |
| `daqiri_pool_ring_bench` | `pool_ring_bench.cpp` | none — microbenchmark comparing `daqiri::Ring`/`daqiri::ObjectPool` vs DPDK `rte_ring`/`rte_mempool` (SPSC/MPMC, single/bulk, thread sweep). The `rte_*` comparison arm compiles only in a DPDK-enabled build; takes no YAML/CLI args |

The four `raw_*` benches share `raw_bench_common.{cpp,h}` and accept `--seconds N`. `daqiri_bench_rdma` and `daqiri_bench_socket` also take `--mode {tx,rx,both}`.

```bash
./build/examples/daqiri_bench_raw_gpudirect ./build/examples/daqiri_bench_raw_tx_rx.yaml --seconds 10
./build/examples/daqiri_bench_socket        ./build/examples/daqiri_bench_socket_udp_tx_rx.yaml --seconds 10 --mode both
```

YAML files contain `<angle-bracket>` placeholders (PCIe addresses, CPU cores, MACs, IPs) that **must** be replaced for your system. `daqiri_bench_raw_sw_loopback.yaml` requires no physical link and is the fastest way to smoke-test a build.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NVIDIA/daqiri](https://github.com/NVIDIA/daqiri) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
