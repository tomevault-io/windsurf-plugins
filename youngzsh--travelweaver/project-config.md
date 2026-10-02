---
trigger: always_on
description: TravelWeaver 是面向长程旅行规划 Agent 的确定性、可回放环境。环境、Function Calling
---

# TravelWeaver 仓库协作指南

## 项目目标与当前阶段

TravelWeaver 是面向长程旅行规划 Agent 的确定性、可回放环境。环境、Function Calling
协议、通用任务规格、确定性 Reward、可审计任务合成、Tool-Graph SFT、在线 GRPO 和固定
ChinaTravel 官方评测链路已经可用。
当前工作重心是：

1. 扩展 official-recombined 合成任务与可执行 Tool-Graph ReAct 数据，并继续提高公开证据质量；
2. 在 Qwen3.5-4B 上迭代 SFT、hard-first Reward 和严格 on-policy GRPO；
3. 用固定官方评测、内部 replay/failure-stage 诊断和同配置 ablation 验证改动，而不是混合指标。

ChinaTravel benchmark 用于约束分布和表达风格参考，以及最终效果评估；合成训练数据不要求
逐字或逐题复制 benchmark，也不得直接复用 benchmark 原句。训练数据应覆盖相近能力分布，
同时保留更丰富的组合和表述。

## 两套隔离环境

### 根目录：环境、合成与 API rollout

- Python 固定为 3.10，使用 `uv`，不要向系统 Python 安装依赖。
- 初始化子模块：`git submodule update --init --recursive`
- 安装普通开发环境：`uv sync --dev`
- 安装 API rollout 依赖：`uv sync --extra api --dev`
- 准备 ChinaTravel 数据：`uv run travelweaver bootstrap chinatravel`
- 导入 benchmark：`uv run travelweaver import-tasks --split benchmark`
- 环境检查：`uv run travelweaver check-env`
- 根目录测试：`uv run pytest`
- 根目录静态检查：`uv run ruff check .`

根目录环境必须保持轻量，不得引入 PyTorch、CUDA、veRL、vLLM 或训练框架。

### `training/`：SFT 与 GRPO

训练栈是独立的 uv project，不得从根项目导入 GPU 依赖：

```bash
uv sync --project training --dev
uv run --project training python training/scripts/check_environment.py
```

当前训练环境：

- Python `3.10.19`
- veRL `main` commit `4a2cba76f7f605d2b9f56e640faaeaa71c2c7f71`（`0.9.0.dev`）
- PyTorch `2.10.0`，CUDA runtime `12.8`
- Transformers `5.5.3`，vLLM `0.19.1`
- FlashAttention `2.8.3`
- Flash Linear Attention `0.5.1`，FlashInfer `0.6.6`
- FSDP/FSDP2 训练后端，vLLM rollout 后端
- 当前主机 8 张 NVIDIA A800 80GB SXM4，compute capability `8.0`；已审计的 SFT/GRPO
  launcher 使用 2 卡或 4 卡配置，当前正式实验使用 GPU 0-3 的 4 卡配置

FlashAttention 在当前 glibc 2.31 主机上使用 CUDA 12.6 toolkit 本地编译；相关构建变量已写入
`training/pyproject.toml`。不要单独升级 PyTorch、vLLM、FlashAttention 或 CUDA 组合。当前不使用
SGLang、Megatron-LM、Apex 或 TransformerEngine。

修改训练依赖后应同步更新 `training/uv.lock`，重新运行环境检查。新增训练代码后使用：

```bash
uv run --project training pytest training/tests
uv run --project training ruff check training
```

## 当前代码结构

- `src/travelweaver/env/`：episode 状态机、15 个公开工具、稳定 ID、Scenario 和
  ChinaTravel backend。
- `src/travelweaver/data/`：数据库准备、校验和 benchmark 任务快照导入。
- `src/travelweaver/tasks/`：`TaskBlueprint`、`TaskSurface`、通用 `TravelTaskSpec`、编译与解析。
- `src/travelweaver/synthesis/`：配额目录、可行 witness、canonical 渲染、LLM polisher、
  preference audit 和版本化产物。
- `src/travelweaver/llm/`：provider-neutral OpenAI-compatible client 和 DeepSeek 配置适配。
- `src/travelweaver/rollout/`：Agent 循环、单任务 API rollout、可恢复的生成任务批量 rollout
  和完整轨迹。
- `src/travelweaver/reward/`：确定性约束验证、Reward 和严格 RFT 接纳过滤。
- `src/travelweaver/sft/`：程序化 Tool-Graph teacher、可见 rationale 润色、批次审计、轨迹重放
  和 trainer-neutral SFT。
- `src/travelweaver/evaluation/`：ChinaTravel 官方计划导出与审计，以及与训练 Reward 分离的
  盲测 LLM Judge。
- `src/travelweaver/cli/`：数据、合成、重写、rollout 和环境检查命令入口。
- `training/`：隔离的 SFT/GRPO 依赖、Qwen adapter、在线 AgentLoop、训练侧 Reward/sampler、
  checkpoint 评测、配置和 launcher。
- `tests/`：按根项目模块镜像组织的离线测试。
- `docs/`：架构、协议、Reward、合成计划和人工审计结论。
- `vendor/ChinaTravel/`：固定版本上游子模块；除非任务明确要求，不要直接修改。

## 协议与产物版本

修改序列化结构或行为时，检查并按需升级对应版本：

- Environment：`travelweaver-environment-v0.8`
- Observation：`travelweaver-observation-v5`
- Tools：`travelweaver-tools-v5-agent`
- Plan snapshot / evidence：`travelweaver-plan-snapshot-v2` / `travelweaver-evidence-v3`
- TaskSpec：`travelweaver-task-spec-v3`
- Blueprint / Surface：`travelweaver-task-blueprint-v2` / `travelweaver-task-surface-v3`
- Reward：`travelweaver-reward-v9`
- Outcome contract：`travelweaver-outcome-contract-v2`
- Trajectory：`travelweaver-trajectory-v12`
- Model tool response：`travelweaver-model-tool-response-v4`（默认 `delta`，兼容 `snapshot`）
- Model context policy：`travelweaver-model-context-policy-v4-commit-first`
- Scenario：`travelweaver-scenario-v1`
- Synthesis / artifacts：`travelweaver-synthesis-v50` /
  `travelweaver-synthesis-artifacts-v39`
- Programmatic policy / artifacts：`travelweaver-programmatic-policy-v75` /
  `travelweaver-programmatic-artifacts-v43`
- Hierarchical Tool Graph：`travelweaver-hierarchical-tool-graph-v4`
- Tool-call / generation graph：`travelweaver-tool-call-graph-v17` /
  `travelweaver-generation-dependency-graph-v9`
- Rationale polisher / prompt：`travelweaver-trajectory-rationale-polisher-v16` /
  `travelweaver-trajectory-rationale-prompt-v11`
- SFT：`travelweaver-sft-v7`
- GRPO prompts / split report / split artifacts：`travelweaver-grpo-prompts-v3` /
  `travelweaver-grpo-prompt-split-v4` / `travelweaver-grpo-prompts-split-v4`
- GRPO training Reward：`travelweaver-grpo-training-reward-v3`
- ChinaTravel official export / audit：`travelweaver-chinatravel-export-v1` /
  `travelweaver-chinatravel-official-audit-v1`
- Polisher prompt：`travelweaver-zh-polisher-v7`

不要在不升级版本和补兼容测试的情况下静默改变字段含义。读取旧快照时保持显式兼容或明确
拒绝，不要猜测缺失字段。

## 核心设计约束

1. 环境必须保持确定性和可回放。排序、分页、ID、路线、Scenario 和 Reward 不得依赖未固定
   的随机状态。
2. `tool_schemas.py` 是面向模型的统一 Function Calling 协议，不是 ChinaTravel 原始 API 的
   逐字段复制；上游字段转换集中在 backend。
3. Agent 只能引用本 episode 已展示的实体；计划只能引用已保存候选和已查询路线。不得绕过
   evidence contract 直接读取 oracle 数据。
4. ChinaTravel 只是首个任务来源。新约束进入通用 `TravelTaskSpec`，不得把 benchmark 特有
   逻辑硬编码进 Reward。
5. 训练 Reward 必须确定、可审计。LLM Judge 只做离线主观评估，不参与训练 Reward，也不与
   Reward 合并成单分数。
6. 默认使用进程内 Function Calling，不引入 MCP，除非项目范围明确改变。
7. Preference-like 的 `unscored_preferences` 当前不进入训练 Reward；只能通过独立 preference
   audit 或离线 Judge 分析，不能宣称 Reward 证明了偏好最优。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [YoungZSh/Travelweaver](https://github.com/YoungZSh/Travelweaver) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
