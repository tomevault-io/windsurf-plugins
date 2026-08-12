---
trigger: always_on
description: 本仓库包含 Magnetar 模型部署工具。所有 Agent 回复默认使用中文。
---

# AGENTS.md

本仓库包含 Magnetar 模型部署工具。所有 Agent 回复默认使用中文。

## 项目目标

将远程或本地浮点模型转换为 AX 芯片客户交付包：

`模型 → ONNX → Pulsar2 编译 → AXMODEL → 仿真验证 → Python/C++ SDK → 交付包 → 发布`

## 工具库

Agent 负责编排和决策。`magnetar/stages/*.py` 提供确定性执行函数：

| 模块 | 函数 | 用途 |
|------|------|------|
| `magnetar.config` | `load_config()` | 读取 `.magnetarrc` + 环境变量 |
| `magnetar.docker_util` | `latest_pulsar2_image()`, `docker_pulsar2()` | Docker/Pulsar2 封装 |
| `magnetar.board_util` | `select_board()`, `ssh()`, `scp_to()`, `scp_from()`, `ensure_remote_infer()`, `port_open()` | AX 板端操作（上板前确保 ax-remote-infer 已装，18500 端口可发现板子） |
| `magnetar.stages.init` | `run(config)` → `task_dir` | 创建 TASK_DIR 结构 |
| `magnetar.stages.acquire` | `run(task_dir, source)`；`write_model_flow(task_dir, flow)` | 获取模型到 origin/ 并记录运行流程 |
| `magnetar.stages.export` | `run_mobilenet(task_dir)` → `sample`；`run_generic(task_dir, ...)` → `result` | MobileNet 专用 / 任意模型通用导出（先简后繁自动降级） |
| `magnetar.stages.toolchain` | `run()` → `pulsar_image` | 验证 Pulsar2 Docker 可用 |
| `magnetar.stages.compile` | `run(task_dir, target_hw, image)` | Pulsar2 编译 AXMODEL |
| `magnetar.stages.simulate` | `run(task_dir, sample, image, board=board, target_hw=...)` → `metrics` | 精度对分（有板优先上板 ax_run_model，无板才回退 pulsar2 run） |
| `magnetar.stages.sdk_gen` | `run_mobilenet_python()`, `run_mobilenet_cpp()`；`run_generic_python(task_dir)`, `run_generic_cpp(task_dir)` | 生成 Python/C++ SDK（通用版基于 model_meta + model_flow） |
| `magnetar.stages.runonboard` | `run(task_dir, sample, hw, pwd)` → `metrics` | 板端部署验证 |
| `magnetar.stages.package` | `assemble(task_dir, metrics, image)` → `pkg`, `self_test(pkg)` → `result` | 组装面向小白的交付包，含一键脚本 + README + 自测 |
| `magnetar.stages.publish` | `publish(pkg, target, name, token, org, model)` → `result` | 发布到 GitHub（源码）或 HuggingFace（预编译） |
| `magnetar.stages.llm` | `classify(origin, ...)` → 路由；`llm_build(task_dir, input, chip, image, ...)` → `model_dir`；`install_axllm(board)` / `serve_axllm(board, model_dir)` / `validate_chat(api_url, ...)` | LLM/自回归模型路由与 ax-llm 部署（llm_build2 编译 + axllm 板端 serve/验证） |

非 MobileNet 模型：优先使用 `magnetar.stages.export.run_generic` /
`scripts/export_onnx.py` 通用导出器（load 脚本约定 `build()` 返回 `(model, example_inputs)`），
导出失败时依据 `export/export_report.md` 的诊断报告决定人工处理方向；确需手写导出逻辑时
再自行实现并正确填写 `model_meta.json`。

LLM/自回归模型（route=llm）：不走通用 ONNX 路径，改用 ax-llm——
`pulsar2 llm_build2`（Pulsar2 ≥ 6.0）直接编译 HF 权重 → `compile/llm_model_dir/`
（逐层 axmodel + post axmodel + bf16 embedding + tokenizer + axllm config.json），
板端用 `axllm run/serve`（AXERA-TECH/ax-llm，axllm 分支）；SDK 为 OpenAI 兼容
HTTP 客户端（Python 依赖仅 requests）。判定函数：`magnetar.stages.llm.classify`
（config architectures/model_type、pipeline_tag、model_flow task、模型名）。
hybrid 组合模型（MOSS-TTS、NeuTTS-2E 等）需先确认 LLM/AR 子模型拆分方案。

## 执行流程

严格按以下顺序推进 10 阶段，不可跳过。每阶段完成后更新 `task.md` 和 `analysis.md`。
INIT 后先过 `model_route` gate：route=llm 时 EXPORT/COMPILE/SIMULATE/SDK-GEN/
RUNONBOARD/PACKAGE 按 ax-llm 分支执行（详见 `.codex/skills/magnetar/SKILL.md` 路由节）。

状态机（回退/重试/循环）由 `workflows/magnetar.yaml` 控制。日常执行按
`workflows/magnetar-summary.md`（全局读一次）+ `workflows/steps/<阶段>.md`（每阶段读对应片段），
仅状态机诊断/排障时才读 yaml 全文。

SIMULATE 有板必上板（ax_run_model 秒级），pulsar2 run 仅无板/板端失败时回退；BOARD 未配置时先 `select_board()` 找空闲板，找不到才用仿真。

## STOP 点

必须暂停等待用户确认：
- `SOURCE`、`TARGET_HARDWARE` 未提供
- ONNX 与原模型对分失败（cosine < 0.99）
- 模型含动态 shape 且静态化失败
- Pulsar2 不可用
- 编译失败需改 ONNX → 退回 EXPORT
- SIMULATE 精度不达标（先查 `issues/`；INT8/U16/混合精度全试过仍不过时，STOP 前先向用户提议上 QAT）
- 需要私有凭据
- PUBLISH 需用户确认发布目标、仓库名、凭据

BOARD 缺失不是 STOP：SIMULATE 先用 `select_board()` 找空闲板上板，找不到才回退 pulsar2 run；RUNONBOARD 无板自动跳过。

## 配置

优先读取 `.magnetarrc`（shell 风格 key=value），环境变量可覆盖。详见 `.magnetarrc.example`。

多任务并发隔离约定：
- `.magnetarrc` 只放公共默认（凭据/工具链/镜像/行为选项），任务参数（SOURCE/TARGET_HARDWARE/MODEL_NAME/BOARD/TASK_DIR）不要在里面反复改写
- 每个任务 INIT 时把任务参数固化到 `TASK_DIR/config.json`；之后各阶段一律用 `magnetar.config.load_task_config(task_dir)` 读取，禁止并发任务互相改写 `.magnetarrc`

## 目录约定

```
TASK_DIR/
  origin/       export/       compile/       simulate/
  sdk/python/   sdk/cpp/      runonboard/    package/    cache/
  task.md       analysis.md
```

产物不得污染原始模型工程。

## 模型获取

- 模型下载/获取优先 ModelScope（国内 CDN 快，公开模型无需额外凭据），HuggingFace 仅作回退
- 涉及 HF 的任何东西（模型权重、Pulsar2、AX650-Community-Hub、ModelZoo 等）先查
  ModelScope 有没有：`magnetar.net_util.modelscope_available("<org>/<name>")` 探测
  （HF repo id 与 ModelScope 约定一致，如 AXERA-TECH/Pulsar2 两侧都有），有则优先
  ModelScope 下载，没有才回退 HF/hf-mirror
- HF 大文件（权重等）下载默认用 hf-mirror 的 hfd 工具：
  `scripts/download_hf.sh <org>/<name> --local-dir origin/<name> -x 8`
  （自动缓存 `~/.cache/magnetar/hfd.sh`，端点默认 hf-mirror，aria2c 多线程；
  小文件如 tokenizer.json 仍可直接单线走 HF_ENDPOINT）
- 默认国内镜像：HuggingFace `HF_ENDPOINT=https://hf-mirror.com`；GitHub 克隆/下载经
  `GH_PROXY=https://gh-proxy.com`；uv/pip 默认 `PIP_INDEX_URL=https://mirrors.aliyun.com/pypi/simple/`
  （海外用户可在 `.magnetarrc` 置空字符串禁用，恢复直连）
- 大权重可用 ModelScope CDN 分片并行下载（参考 `issues/013_moss-tts-realtime_ax650_pipeline_pitfalls.md`）
- SOURCE 支持 ModelScope / HuggingFace / Git URL / HTTP URL / 本地文件或目录

## 关键技术点

### 校准归一化对齐

Pulsar2 用 `(img - mean) / std`，libdet 用 `(input - mean) * std`。必须反向对齐：

| 组件 | 配置 | 输入范围 |
|------|------|----------|
| Pulsar2 校准 | `calibration_std = 255` | uint8/255 = [0,1] |
| libdet 推理 | `std = 1/255` | uint8 × (1/255) = [0,1] |

**常见错误**：`calibration_std = 0.004`（即 1/255）→ 校准输入 [0,65025] → 板端全零。

### 量化

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AXERA-TECH/Magnetar](https://github.com/AXERA-TECH/Magnetar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
