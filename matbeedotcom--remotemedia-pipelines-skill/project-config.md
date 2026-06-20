---
trigger: always_on
description: Use when running, prototyping, composing, or benchmarking remotemedia SDK media pipelines from Python — audio/video/text processing, Whisper STT, Kokoro/VibeVoice/Voxtral/CosyVoice3 TTS, LFM2/LFM25 audio, Silero VAD, resampling, real-time bidirectional streaming sessions, mic→speaker S2S, or any manifest with `node_type` like `WhisperSTTNode`, `KokoroTTSNode`, `FastResampleNode`, `SileroVADNode`, `AudioChunkerNode`, `VogentTurnOnnxNode`, `LFM25AudioOnnxNode`. Triggers on `from remotemedia.core.p
---


# remotemedia Pipelines (Python)

## What this skill is for

Build and execute media pipelines from Python using the `remotemedia` package. Five entry points exist — pick by what you have in hand:

| You have | Entry point | Notes |
|---|---|---|
| `Node` instances | `Pipeline.run(input)` | Easiest. Auto-falls back to Python executor when a node isn't in the Rust registry. |
| A list of `Node` instances or dict-manifests | `execute_pipeline(items)` / `execute_pipeline_with_input(items, inputs)` | Goes through `runtime_wrapper`. Pure Node lists use `execute_pipeline_with_instances` (registry-bypass). **Strict unary** — one input → one output, session closes. |
| A JSON/dict manifest with Rust `node_type`s | `remotemedia.runtime.execute_pipeline(json_str, enable_metrics)` | Direct FFI. Only works for node types the linked `.so` actually registered. Strict unary. |
| A long-running bidirectional pipeline (mic→speaker, live VAD, streaming TTS, multimodal interleaved S2S) | `remotemedia.runtime.create_streaming_session(manifest_json)` | **Continuous** input/output. Multiple `send_input` + multiple `recv_audio` / `recv_data` / `recv_video`. Required for any manifest whose nodes emit multiple outputs per input (LFM2-Audio interleaved, VAD+`AudioBufferAccumulatorNode` chains, anything streaming TTS). |
| A manifest you want to **profile** end-to-end (TTFA / TTFT / per-node percentiles / capture) | `remotemedia.runtime.benchmark_pipeline(manifest_json, input_paths, options_json=None)` | Replays one or more `.wav`/`.txt` inputs as utterances through the manifest, measures eos-to-first-emit + per-node merged HDR percentiles, optionally captures per-utterance outputs to disk. Returns the bench report as a JSON string. |

`pipeline.run()` is the safest default — it tries Rust, then transparently falls back. Use `create_streaming_session` whenever the pipeline is genuinely streaming — the unary entry points only emit the *first* output and close the session, which silently truncates everything else. Use `benchmark_pipeline` when the question is "how fast?" or "where's the bottleneck?" — same harness the CLI's `remotemedia bench` uses, including merged-HDR per-node percentiles.

## Required setup before any code runs

```bash
pip install remotemedia                       # installs `remotemedia` package + the Rust FFI extension
```

Verify the install:

```python
import remotemedia
print(remotemedia.__file__)                    # path inside your site-packages
print(remotemedia.is_rust_runtime_available()) # True if FFI .so is loadable
```

**Gotcha** — if a `remotemedia/__init__.py` stub exists in your project's current working directory, Python will resolve `import remotemedia` to that stub instead of the installed package. From that cwd, `is_rust_runtime_available` will be missing. Fix by running from a different cwd (`cd ..`) or renaming the stub directory.

## CUDA / cuDNN runtime setup (only if you want GPU acceleration)

The pinned ONNX Runtime version inside the SDK needs **cuDNN 9** (`libcudnn.so.9`) at runtime to enable its CUDA execution provider. If you only see `Successfully registered CPUExecutionProvider` in the logs and a `cannot open libcudnn.so.9` error above it, that's why.

You don't need CUDA at all for the unary or streaming entry points **unless** your manifest specifies `device: "cuda:N"` on a node. Without it, ORT silently falls back to CPU EP. Audio-DSP nodes (`FastResampleNode`, `SileroVADNode`, `AudioChunkerNode`, `AudioBufferAccumulatorNode`) are CPU-only and don't care.

To enable CUDA, install cuDNN 9 + cuBLAS + CUDA runtime via whichever Python env you already use, then point `LD_LIBRARY_PATH` at them before launching Python:

**uv / pip venv:**
```bash
uv pip install nvidia-cudnn-cu12 nvidia-cublas-cu12 nvidia-cuda-runtime-cu12
LD_LIBRARY_PATH="$VIRTUAL_ENV/lib/python$(python -c 'import sys;print(f"{sys.version_info.major}.{sys.version_info.minor}")')/site-packages/nvidia/cudnn/lib:$VIRTUAL_ENV/lib/python.../site-packages/nvidia/cublas/lib:${LD_LIBRARY_PATH:-}" \
  python your_script.py
```

**conda:**
```bash
conda install -c conda-forge cudnn=9
LD_LIBRARY_PATH="$CONDA_PREFIX/lib:${LD_LIBRARY_PATH:-}" python your_script.py
```

**System CUDA install** (`/usr/local/cuda-*`):
```bash
LD_LIBRARY_PATH="/usr/local/cuda/lib64:${LD_LIBRARY_PATH:-}" python your_script.py
```

After setting the right `LD_LIBRARY_PATH`, the SDK's installed ORT provider `.so` files (`libonnxruntime_providers_cuda.so` etc.) will load cuDNN/cuBLAS correctly. These provider files ship inside the `remotemedia` package itself — you don't manage them.

### Probe-once helper


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [matbeedotcom/remotemedia-pipelines-skill](https://github.com/matbeedotcom/remotemedia-pipelines-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
