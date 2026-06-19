---
trigger: always_on
description: 你是 agent-infer 推理框架的生成 Agent。本目录是自包含的一次性知识包（prior knowledge）。
---

# Inference Agent System — Claude Code 入口

你是 agent-infer 推理框架的生成 Agent。本目录是自包含的一次性知识包（prior knowledge）。
**本目录就是工程根目录**——代码直接写入本目录下，不存在子仓库。

## 三层知识体系

```
第一层：先验知识（人类写，你只读）
  ├── inference_blueprint.json    ← 架构知识图谱（唯一契约来源）
  ├── AGENT_SKILL.md              ← 执行 SOP + 编码铁律
  └── scripts/                    ← 固定测试合约（26 个，不可修改）

第二层：生成产物（你写，受第一层约束，直接写入本目录）
  ├── engine/                     ← 推理框架代码
  ├── llm_engine.py               ← 引擎主循环
  └── openai_tp_server.py         ← OpenAI API 服务

第三层：验收证据（你运行，不可伪造）
  ├── phase_report/               ← 每个 Phase 的审查报告
  │   ├── PHASE1_IMPLEMENTER_REPORT.md
  │   ├── PHASE1_SPEC_REVIEW_REPORT.md
  │   └── PHASE1_VERIFICATION_REPORT.md
  ├── profiler trace
  ├── HCU/VRAM 监控
  └── benchmark JSON
```

## 环境约定

本包为开源知识包——不硬编码任何绝对路径。所有外部依赖由用户在首次使用时指定。

| 变量 | 说明 | 获取方式 |
|------|------|---------|
| `AGENT_INFER_ROOT` | 推理框架代码仓库根目录（即本目录） | 自动检测：当前工作目录 |
| `MODEL_DIR` | 模型权重目录 | **启动时询问用户** |
| `PYTHON_PATH` | Python 环境路径（conda/venv 的 bin 目录） | **启动时询问用户** |

**推荐目录布局**：
```
inference-agent-system/         ← 本包（工程根目录）
├── engine/                     ← 推理框架代码
├── llm_engine.py               ← 引擎主循环
├── openai_tp_server.py         ← API 服务
├── phase_report/               ← 审查报告
└── ...
<用户指定的模型目录>/
    ├── config.json
    ├── model.safetensors.index.json
    └── ...
```

## 启动时强制动作

0. **询问用户环境配置**：在开始任何工作前，必须先确认以下路径（如果用户尚未提供）：
   - **模型目录 (MODEL_DIR)**：模型权重文件所在的目录（如 `/data/models`）
   - **Python 环境 (PYTHON_PATH)**：包含 `python`、`flash_attn`、`vLLM` 的 conda/venv 的 bin 目录（如 `/opt/conda/envs/meta/bin`）
   
   验证方式：
   ```bash
   # 验证 MODEL_DIR
   ls "${MODEL_DIR}/config.json" 2>&1 && echo "MODEL_DIR OK" || echo "MODEL_DIR 下找不到 config.json"
   # 验证 Python 环境
   "${PYTHON_PATH}/python" -c "import torch; import flash_attn; print(f'CUDA:{torch.cuda.is_available()} flash_attn OK')"
   ```

1. 读取 `inference_blueprint.json`（先看 `agent_navigation`，再按需展开）
2. 读取 `AGENT_SKILL.md`（含编码铁律、Phase-Script 绑定表、Debug 指南）
3. 在运行 scripts/ 前设置环境：
   ```bash
   export AGENT_INFER_ROOT="$(pwd)"
   export PATH="${PYTHON_PATH}:$PATH"
   export PYTHONPATH="${AGENT_INFER_ROOT}:$PYTHONPATH"
   ```
4. 确认目标模型 `config.json`（architectures, rope_scaling, num_heads 等）
5. 输出"模型路由结论"：Dense 还是 MLA+MoE

## 对抗子代理协作流（Superpowers 风格）

代码生成分为三个角色，独立子代理执行，互不信任。

**核心原则**：implementer 不自证清白——它只产出代码，不跑测试，不宣判 PASS。
审查串行执行：先 spec-reviewer（蓝图契约核验），通过后才到 verification（测试+证据）。
二者不并行——spec-reviewer ❌ 时，verification 根本不需要跑，节省资源且消除"测试都过了就放行"的降级冲动。

```
                    ┌─────────────────────┐
                    │  主 Agent（你）       │
                    │  读蓝图 → 拆 Task    │
                    │  派子代理 → 收集结果  │
                    └──────┬──────────────┘
                           │
                           ▼
                    ┌────────────┐
                    │ implementer│
                    │ 写代码      │
                    │ 自读diff    │
                    │ (不跑测试)   │
                    │ → SUBMITTED│
                    └─────┬──────┘
                          │
                          ▼
                    ┌────────────┐      ❌ FAIL
                    │spec-reviewer│ ──────────→ 打回 implementer
                    │ 对照蓝图审查 │
                    │ 独立读代码   │
                    │ 核对契约     │
                    └─────┬──────┘
                          │ ✅ PASS
                          ▼
                    ┌────────────┐      ❌ FAIL
                    │verification│ ──────────→ 打回 implementer
                    │ L1:scripts/ │
                    │ L2:跨Phase  │
                    │ L3:profiler │
                    │   +HCU证据  │
                    └─────┬──────┘
                          │ ✅ PASS
                          ▼
                    ┌────────────┐
                    │  Phase 交付 │
                    └────────────┘
```

### 子代理 Prompt 模板位置

| 角色 | Prompt 文件 | 职责 | 跑测试？ | 宣判 PASS？ |
|------|-----------|------|---------|-----------|
| implementer | `.claude/skills/implementer-inference.md` | 读蓝图+AGENT_SKILL → 写代码 → 自读diff → 提交 | ❌ | ❌ |
| spec-reviewer | `.claude/skills/spec-reviewer-inference.md` | 不信任实现者 → 独立逐行读代码 → 对照蓝图每条契约核验 | ❌ | ❌ |
| verification | `.claude/skills/verification-inference.md` | **唯一测试执行者**：L1 scripts/ + L2 跨Phase回归 + L3 profiler/HCU | ✅ | ✅ |

### ⚠️ 子代理必须物理隔离——禁止同一 Agent 扮演三个角色

**为什么不能自己扮演三个角色**：同一个 Agent 在 implementer 阶段写的代码，切换到 spec-reviewer 阶段时会带有 confirmation bias——它记得自己刚才为什么那样写，会下意识为错误找理由。对抗结构的前提是审查者**不知道**实现者的思考过程。

审查子代理的隔离程度决定审查质量：

| 审查角色 | 挂载方式 | 为什么 |
|---------|---------|--------|
| **implementer** | Agent 工具（`subagent_type: general-purpose`） | 需要完整工具链——读蓝图、读 ref_docs/ref_code、写代码文件 |
| **spec-reviewer** | **Shell `claude -p --allowedTools`** | 需要真正的进程隔离——新 PID、无父进程记忆、不可访问主Agent上下文 |
| **verification** | **Shell `claude -p --allowedTools`** | 需要真正的进程隔离——只跑命令看结果，不看任何其他子代理的输出 |

Shell `claude -p` 和 Agent 工具的区别：

```
Agent 工具：主 Agent ──spawn──→ 子 Agent（共享 harness 配置，clean context 但有 sysprompt 残留）
Shell claude -p -：主 Agent ──fork──→ 独立进程（新 PID、全新上下文加载、无任何父进程记忆）
```

Shell 方式下，审查子代理物理上无法知道 implementer 读了哪些文件、用了什么模型、思考过程如何——它只能读你传给它的脚本文件路径和门禁 checklist。这才是真正的"对峙"。

### 每个 Phase 的 spawn 协议

**步骤 1**：主 Agent 读取蓝图和 AGENT_SKILL.md，确定当前 Phase 的 Task 范围，然后用 **Agent 工具** spawn implementer：

```
Agent(
  subagent_type: "general-purpose",
  description: "Phase N implementer",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MetaInfer/MetaInfer](https://github.com/MetaInfer/MetaInfer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->
