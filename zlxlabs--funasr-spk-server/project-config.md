---
trigger: always_on
description: - 有完备的日志系统，方便后期调试确认问题。
---

Always respond in 中文

# 代码设计上的要求
- 各部分功能尽量低耦合，高内聚。
- 各个函数代码做好注释。
- 有完备的日志系统，方便后期调试确认问题。
- 避免写冗余代码，提高项目的可复用性。
- 尽力遵循工程上的最佳实践。适当使用文件夹来增加整个项目的可读性，但不要添加过多无关文件。
- 及时完善 .gitignore 文件。

# 测试约定（PR1 之后启用）

## pytest 套件
- 测试目录三层：
  - `tests/unit/` — 单元测试（mock 外部依赖，毫秒级）
  - `tests/integration/` — 端到端集成（含真实 FunASR 模型）
  - `tests/manual/` — 历史手工脚本（**不在 pytest 收集范围**），仅作复现参考
- 推荐命令：`venv/bin/python -m pytest`（venv 的 pytest binary shebang 漂移，用 `-m pytest` 走 venv python）
- integration 默认 skip，需 `FUNASR_RUN_INTEGRATION=1` 才跑

## TDD 流程
- 新功能 / 改 bug 都先写测试再改代码
- 红 → 绿 → commit 是最小单位，**不要积累多个改动一次性提交**
- 每个 commit message 写清楚改了什么 + 解决什么问题

## Parity 测试
- 改动 FunASR 路径（schemas / database / task_manager / websocket_handler / funasr_transcriber）后必跑：
  ```
  FUNASR_RUN_INTEGRATION=1 venv/bin/python -m pytest tests/integration/
  ```
- 通过 = 改动安全；失败 = 你改坏了 FunASR 路径，回去查
- golden baseline 在 `tests/fixtures/golden/`，由首次运行自动生成

# ASR 引擎架构

## 当前现状（2026-05-23）

两个引擎并行接入生产，dispatch 走轻量函数路由（**不是 ABC 抽象**）：

- **FunASR**（生产稳定）: `src/core/funasr_transcriber.py`，MPS GPU 加速。canonical 为**句级** segments（引擎层不再合并）；JSON 出口在 serve 投影层 `merge_segments_view` 做同说话人相邻句合并 + `segment_merge_max_span_sec`（默认 120s）上限（issue #1）
- **Qwen3**: `src/core/qwen3_pool_transcriber.py`（**runtime-aware 池 dispatch**, 见下文）+ `src/core/qwen3_transcriber.py`（单例）+ `src/core/qwen3/asr.py`（引擎构造）。Mac 上 frontend ONNX 走 CoreML ANE（`onnx_provider="COREML_ANE_FE"`），见 `spikes/qwen3_mac_hw_accel/SUMMARY.md`
- **Dispatch**: `src/core/transcriber_dispatch.py` 的 `resolve_transcriber()` 按 engine 名分支
- **引擎选择优先级**: `upload_request.engine` > `config.transcription.default_engine`（env `FUNASR_DEFAULT_ENGINE`）> `funasr`
- **缓存隔离**: 缓存 key 按 `(file_hash, engine)` 区分，跨引擎不命中；engine 维折 word_align / diarize 状态（`qwen3+wa:<lang>+nospk` 形态，见「diarize 开关」节），折维参数统一走 `database.cache_params_for(task)`（word_align 维读 `options.word_align` 非 config，且 `output_format=srt` 强制降 +wa；写入用 `cache_save_engine_for` 对齐失败时降 +wa 防毒化；跨引擎回退排除折维行，见 pipeline 5.5 词级时间戳节）

## Runtime + Diarize backend 抽象

`src/core/runtime.py` 的 `detect_runtime()` 返回 `MacRuntime / CudaRuntime / CpuRuntime`，按 `sys.platform` + `onnxruntime CUDAExecutionProvider` 探测自动选；`FUNASR_RUNTIME=cpu/mac_ane/cuda` env 强制 override。每个 runtime 暴露：
- `validate()` — `CudaRuntime` 显式 assert CUDA EP 在 ORT providers 列表，缺则 fail-fast（替代 ORT silent CPU fallback）
- `recommend_diarize_backend()` — Mac/Cpu → `sherpa`, Cuda → `ort_cuda`
- `recommend_num_threads()` — Mac 固定 4, Linux 按 `cpu_count()` 给 2/4

Qwen3 diarize 有 **两个 backend 实现**，通过 `src/core/qwen3/diarize.py:run_diarization_dispatched` 路由：

- **`sherpa`**（默认，Mac/Cpu）: `src/core/qwen3/diarize.py:run_diarization`，sherpa-onnx `OfflineSpeakerDiarization` + sherpa C++ FastClustering
- **`ort_cuda`**（CUDA 平台默认）: `src/core/qwen3/diarize_ort.py:run_diarization_ort_cuda`，Python `onnxruntime` 直 wrap pyannote-segmentation-3.0 + TitaNet + scipy 复刻 FastClustering（cosine + complete linkage）。**pipeline 结构 1:1 移植 sherpa C++**（per-chunk 独立 argmax + per-(chunk,speaker) embedding，2026-06-10 修短音频 under-detect 时重写，见 `docs/开发/2026-06-10-ort_cuda短音频under-detect-修复结案.md`；⚠️ pyannote slot 是 chunk 局部的，禁止跨 chunk 平均 logits；TitaNet ONNX 双输出必须取 `embs` 不是 `logits`）。**8 vCPU + RTX 3060 上 30min wall RTF 0.047 vs sherpa CPU 0.080**，详见 `docs/开发/gpu加速/2026-05-22-ORT-CUDA-diarize-backend.md`
- **优先级**: 显式 `backend` 参数 > `FUNASR_QWEN3_DIARIZE_BACKEND` env > `runtime.recommend_diarize_backend()`
- **为什么自建 ort_cuda 而不是用 sherpa CUDA build**: sherpa-onnx CUDA build 的 C++ wrapper 跟 llama.cpp CUDA 撞 segfault，ORT Python API 不撞（`scripts/_remote_ort_cuda_clash_check.py` 验证过）

## Qwen3 池 dispatch (runtime-aware)

`src/core/qwen3_pool_transcriber.py:get_qwen3_pool_transcriber()` 按 `detect_runtime()` 分发到两套池：

- **`cuda` runtime** → **`Qwen3InProcPool`** (`src/core/qwen3_inproc_pool.py`)
  - 单进程内 N 个 `Qwen3DiarizeTranscriber` 实例，`asyncio.Queue` 调度 acquire/release
  - 共享同一 cuda context，**race-free** — 避开 multi-process 跨进程 CUDNN/cuda buffer race
  - RTX 3060 + 8 vCPU 实测 `pool_size=2` 跑 1800s × 2 并发: TOTAL_WALL 142s, 每 task RTF 0.079
  - 详见 `docs/开发/gpu加速/2026-05-23-CUDA并发突破.md`
- **其他 runtime (Mac/CPU)** → **`Qwen3PoolTranscriber`** (file-based multi-process pool, 历史路径)
  - `FileBasedProcessPool` 派发到 `qwen3_worker_process.py` subprocess
  - Mac MPS 上行为 100% 不变

**为什么不统一一套池**: CUDA 多进程下 cuDNN handle 跨进程 race 撞死 worker (实测 MPS 任何 thread% 设置都不解); Mac 多进程下 sherpa CPU 没此问题, multi-process 隔离反而更稳. **rule-of-two backends, runtime 自动选**.

`pool_size` 两套共用 `config.transcription.qwen3_pool_size` (env `FUNASR_QWEN3_POOL_SIZE` 覆盖). 单例缓存在 `_qwen3_pool_singleton`, 测试用 `reset_qwen3_pool_singleton()` 清.

## Qwen3 后处理 pipeline

`Qwen3DiarizeTranscriber.transcribe` 在 ASR + diarize 后串联多层后处理（顺序固定，第 1–5 + 5.5 + 5.7 层各有 config flag + env override 可关，第 6 层是无条件的输出层规范化）。**per-request `options.diarize=False` 时走精简管线**（见下文「diarize 开关」节）：

1. **`filter_spurious_speakers`** — 丢掉总时长太小的"假说话人"，把碎片归到时间最近的有效 speaker

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zlxlabs/funasr_spk_server](https://github.com/zlxlabs/funasr_spk_server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
