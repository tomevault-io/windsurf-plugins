---
trigger: always_on
description: Standalone Rust TensorRT inference + real-time vision **algorithm libraries**
---

# vision-rt

Standalone Rust TensorRT inference + real-time vision **algorithm libraries**
for Jetson Orin (aarch64, SM87, TensorRT 10.3.x, CUDA 12.6). Pure libraries — no
orchestration framework, no bubbaloop dependency. Sensor drivers live in the
separate `sensor-rt` repo; GPU image/tensor types come from `kornia-rs`
(pinned git dep, `cudarc` feature).

## Workspace layout

Package `vrt` (core) + `vrt-*` / `trt-sys` satellites. Short crate names — code
uses `use vrt::`, `use vrt_xfeat::`, `use trt_sys::`. Errors: per-crate
`thiserror` enums; `vrt::BoxError` for algorithm constructors that aggregate kinds.

This repo is being open-sourced under the kornia org incrementally, one model
crate per PR.

| Crate | Role |
|-------|------|
| `crates/trt-sys` | Raw FFI: pure-C shim over TensorRT C++ (bindgen never sees C++ headers) |
| `crates/vrt` | Safe core: Logger→Runtime→Engine→Session Arc chain, `ModelSession`, `cuda` launch helpers |
| `crates/vrt-hub` | Model weights (HF Hub, sha256-pinned) + on-device engine cache |
| `crates/vrt-types` | Model-free leaf (no TRT/GPU-model): `CameraIntrinsics`/`Extrinsics`, GPU `Undistorter`, depth-at-mask sampling |
| `crates/vrt-xfeat` | XFeat keypoints: backbone + GPU NMS/top-K/descriptor sampling/mutual-NN. Crate-local `examples/` (`xfeat_match`, `xfeat_bench`) + `scripts/export_xfeat_backbone.py` |
| `crates/vrt-rfdetr` | RF-DETR object detector (NMS-free) + on-device GPU decode |
| `crates/vrt-rfdetr-seg` | RF-DETR instance segmentation: boxes + per-instance masks + on-device GPU decode |
| `crates/vrt-rfdetr-kpts` | RF-DETR human pose: box + 17 COCO keypoints + confidence (CPU decode) |
| `crates/vrt-depth-anything` | Depth Anything V2 metric depth + depth-at-mask/box fusion kernels |
| `crates/vrt-track` | Pure-CPU **3D multi-object tracker** (ByteTrack assoc + depth-gated 3D Kalman); no GPU/TRT — depends only on `vrt-types` |
| `crates/vrt-viz` | Pure-CPU render (masks/boxes/BEV) + **H.264/WebSocket live view** (browser WebCodecs); no GPU/TRT |

## Architecture in one paragraph

Each model is a plain type that owns a kornia `Preprocessor` and shares **one
CUDA stream** with the rest of the app: `run()` = enqueue all GPU work async →
ONE `cudaStreamSynchronize` → CPU post-process. `ModelSession` wraps the
Session and takes a kornia `Tensor<f32,4>` device input. `XFeat` offers
convenience constructors (`from_hub`/`from_onnx`/`from_engine_file`) over the
`vrt-hub` weight-fetch + engine-cache. No `Pipeline`/`Operator` framework —
composition is just calling methods in a loop.

## Composing multiple models (one image, one stream)

The single-model idiom extends to running **N models on the same frame** with no
framework. Build every model on **one shared `Arc<CudaStream>`**; pass the **same**
device `Image<u8,3>` **by reference** to each `submit` (each preprocessor only reads
it and writes its own reused `input` tensor — no aliasing, no divergence); enqueue
any **fusion kernel last**; then **one** `stream.synchronize()` drains everything.

The stream is an ordered FIFO, so enqueue order *is* the dependency edge: a fusion
kernel enqueued after two models' `submit`s is guaranteed to see both models'
finished outputs — no CUDA events, no second stream (single serial stream by design;
event tracking is deliberately disabled). Caller responsibilities: `submit` all
models from the **same** frame before advancing the source, and keep each frame /
`input` / result buffer alive until the sync (the GPU reads their device pointers
during it). Coordinates line up because every model decodes back to **source pixel
space** and the full-frame `Stretch` preprocess makes cross-grid scaling a plain
`grid/src` ratio. Worked example: `vrt-depth-anything`'s `detect_depth` (RF-DETR-Seg
+ Depth Anything V2 on one stream → per-instance mask-sampled metric depth, one sync).
The full **flagship** — `examples/rtsp_track` — feeds that seg+depth+fusion into the
CPU `vrt-track` 3D tracker (metric depth → `Detection::with_depth` → depth-gated
association) and renders/streams it via `vrt-viz` (annotated view + world-frame BEV,
live over H.264/WebSocket → browser WebCodecs).

## Multiple cameras

One physical GPU → CUDA compute is **serial** whatever you do; `Session` is `Send`
but **`!Sync`** (drive each model from one thread). But **NVDEC decode + VIC resize
are separate fixed-function blocks**, so N cameras *decode concurrently* — only model
inference serializes. Three patterns:

- **A — round-robin, one stream, shared models (default on Orin Nano).** One stream +
  one set of model instances; loop cameras, each with its own reused result buffers;
  one sync per camera-frame. Memory-light (one copy of each engine), works today, no
  code changes. Throughput ≈ `1/(N × per-frame GPU ms)` — honest, since you're
  GPU-bound anyway. Right default for a handful of cameras.
- **B — stream + thread per camera.** Each camera gets its own thread, stream, and
  **own** model instances (`!Sync`) → **N× engine memory**; on 7.4 GB that's ~2–3
  cameras with seg+depth. The single GPU still serializes compute, so little
  throughput gain — only worth it for independent per-camera latency. Usually avoid.
- **C — batched engine.** Re-export at `batch=N` (or dynamic batch), stack N frames

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kornia/vision-rt](https://github.com/kornia/vision-rt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-14 -->
