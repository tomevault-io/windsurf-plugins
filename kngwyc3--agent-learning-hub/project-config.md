---
trigger: always_on
description: > 本文档面向维护者与 AI Agent：快速理解仓库目标、目录结构、各 Stage 关系与修改原则。人类学习者请优先读 [README.md](README.md) 和 [index.html](index.html)。
---

# Agent Learning Hub 总览

> 本文档面向维护者与 AI Agent：快速理解仓库目标、目录结构、各 Stage 关系与修改原则。人类学习者请优先读 [README.md](README.md) 和 [index.html](index.html)。

## 仓库目标

这个仓库的核心目标，是把 AI Agent 学习路径整理成一份可执行、可打勾、可落地的 roadmap，而不是单纯堆链接。

它强调的不是“花哨的多智能体演示”，而是更接近真实生产的能力：

- 最小 agent loop
- 工具调用与严格 schema
- RAG 与引用
- 长期记忆与上下文压缩
- harness engineering
- Skills / MCP / A2A / ACP 等能力封装与协议
- Browser / computer-use agent
- 权限与安全边界
- 评测、trace 与可观测性

## 展示入口

| 入口 | 路径 | 用途 |
| --- | --- | --- |
| 主路线图 | [README.md](README.md) | Stage 0–9 学习清单、Project Ladder、精选资源 |
| 交互式学习页 | [index.html](index.html) | Stage 导航、资源卡片、进度勾选（本地 `python -m http.server` 访问） |
| 仓库总览 | [agent.md](agent.md) | 本文件：结构说明、Stage 映射、维护原则 |
| 分步教程 | [stage-1/](stage-1/) … [stage-9/](stage-9/) | 可运行代码与递增练习 |

## 仓库结构

```text
Agent-Learning-Hub/
  README.md                 # 主路线图
  index.html                # 交互式学习页
  agent.md                  # 本总览文档
  CONTRIBUTING.md           # 贡献指南
  stage-1/                  # 最小 agent loop（6 步 Python）
  stage-2/                  # RAG + 记忆（7 步 Python）
  stage-3/                  # Claude Code harness 12 章导读
  stage-4/                  # Multi-agent coordination（pipeline / supervisor）
  stage-5/                  # Skills 能力打包（4 步 + smoke test）
  stage-6/                  # Browser agent（Playwright + 安全边界）
  stage-7/                  # Eval / trace / 安全门禁
  stage-8/                  # 可部署 CLI Agent（trace / 安全 / 成本上限）
  stage-9/                  # 上下文压缩与记忆（compaction + memory 最小实现）
  skills/teach/             # AI 导师 skill（本仓库自带，学习时直接使用）
```

---

## 各 Stage 说明

### `stage-1/` — 最小 Agent Loop

**目标**：用 Python + OpenAI 兼容 API 搭出「能选工具、能执行、能循环」的最小 Agent。

| 步骤 | 文件 | 内容 |
| --- | --- | --- |
| 1 | `step01_chat.py` | LLM 普通对话 |
| 2 | `step02_json.py` | 结构化 JSON 输出 |
| 3 | `step03_tools_def.py` | 工具 schema 定义 |
| 4 | `step04_one_round_tool.py` | 单轮 tool call 解析与执行 |
| 5 | `step05_agent_loop.py` | 完整 agent loop |
| 产出 | `agent.py` | 50–150 行最小 agent，含步数/超时/错误处理 |

```bash
cd stage-1 && pip install -r requirements.txt && cp .env.example .env
python step01_chat.py
```

---

### `stage-2/` — 工具、RAG 与记忆

**目标**：在 Stage 1 之上加入检索增强、长期记忆和上下文压缩。

| 模块 | 文件 / 工具 | 内容 |
| --- | --- | --- |
| 记忆分层 | `step01_memory_layers.py` | 短期 / 会话 / 长期记忆区分 |
| RAGFlow | `step02`–`step04` | chunk / embed / retrieve / 带引用回答 |
| mem0 | `step05_mem0_memory.py` | 长期记忆写入与召回 |
| Letta | `step06_letta_compaction.py` | 上下文压缩与长对话管理 |
| RAG as Tool | `step07_rag_as_tool.py` | 把 RAG 封装成 agent 工具 |
| 产出 | `agent.py` | 带引用的资料研究助手 |

```bash
cd stage-2 && pip install -r requirements.txt && python step01_memory_layers.py
```

---

### `stage-3/claude-code-docs/` — Claude Code Harness 导读

**目标**：研究现代 coding agent harness 的工程实现，不是教学脚本，而是「生产级样本」的拆解文档。

> 路径已从旧名 `claude-code-source-code/` 更正为 `claude-code-docs/`。

| 章节 | 文件 | 核心内容 |
| --- | --- | --- |
| 00 | [00-概览与项目结构.md](stage-3/claude-code-docs/00-概览与项目结构.md) | 架构全貌、技术栈、最小 Agent 循环 |
| 01 | [01-Tool系统.md](stage-3/claude-code-docs/01-Tool系统.md) | Tool 接口、buildTool、40+ 工具、BashTool |
| 02 | [02-Query引擎.md](stage-3/claude-code-docs/02-Query引擎.md) | Agent 主循环、流式、Auto Compact |
| 03 | [03-Agent系统.md](stage-3/claude-code-docs/03-Agent系统.md) | 子 Agent、Fork Worktree、蜂群协作 |
| 04 | [04-Task系统.md](stage-3/claude-code-docs/04-Task系统.md) | TaskType、状态机、磁盘输出流 |
| 05 | [05-状态管理.md](stage-3/claude-code-docs/05-状态管理.md) | AppState、Store 模式、Speculation |
| 06 | [06-权限系统.md](stage-3/claude-code-docs/06-权限系统.md) | PermissionMode、规则引擎、DenialTracking |
| 07 | [07-MCP集成.md](stage-3/claude-code-docs/07-MCP集成.md) | MCP 协议、ToolSearch 延迟加载 |
| 08–11 | 服务层 / UI 层 / CLI / 设计精髓 | 压缩、Analytics、TUI、设计模式 |

**与 Stage 7 的衔接**：[stage-7/docs/claude-code-permissions.md](stage-7/docs/claude-code-permissions.md) 把 CC 权限链路与 `safety_gate.py` 做了对照说明。

---

### `stage-4/` — Multi-Agent 协调

**目标**：理解多 agent 是协调问题，不是魔法。

| 步骤 | 文件 | 内容 |
| --- | --- | --- |
| 1 | `step01_roles_contracts.py` | 角色职责、输入输出契约、停止条件 |
| 2 | `step02_fixed_pipeline.py` | research → write → review → revise 固定流水线 |
| 3 | `step03_supervisor_router.py` | supervisor 路由下一步 |
| 4 | `step04_stop_conditions.py` | 防止循环、争论和任务漂移 |
| 5 | `step05_single_vs_multi.py` | 判断什么时候单 agent 更好 |
| 专题 | [docs/learn/a2a-vs-shared-state.md](docs/learn/a2a-vs-shared-state.md) | A2A 与共享状态的工程边界 |
| 产出 | `agent.py` | 可调试的多 agent 写作系统 |

```bash
cd stage-4 && pip install -r requirements.txt
python agent.py "写一段解释 supervisor 模式的短文"
```

---

### `stage-5/` — Skills 与能力打包

**目标**：把一类 agent 能力从「临时 prompt」升级成可复用、可测试、可分发的 skill。

| 步骤 | 文件 | 内容 |
| --- | --- | --- |
| 1 | `step01_boundaries.py` | Skill vs Tool / Prompt / MCP 边界 |
| 2 | `step02_load_skill.py` | 加载并校验 `SKILL.md` frontmatter |
| 3 | `step03_validate_report.py` | 验收报告格式检查 |
| 4 | `step04_run_smoke_cases.py` | smoke test 跑通 |
| 公共模块 | `skill_common.py`, `report_check.py` | skill 加载与报告校验 |
| 产出 | `my-skill/` | 完整 skill 示例（SKILL.md + 模板 + 脚本 + 测试） |

```bash
cd stage-5 && python step01_boundaries.py && python step04_run_smoke_cases.py
```

---

### `stage-6/` — Browser Agent

**目标**：让 agent 操作公开网页，并记录截图、DOM、动作日志，同时严守安全边界。

| 步骤 | 文件 | 内容 |
| --- | --- | --- |
| 1 | `step01_validate_url.py` | URL 白名单 / 黑名单校验（无 Playwright 依赖） |
| 2 | `step02_observe_page.py` | 页面观察与 DOM 摘要 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kngwyc3/Agent-Learning-Hub](https://github.com/kngwyc3/Agent-Learning-Hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
