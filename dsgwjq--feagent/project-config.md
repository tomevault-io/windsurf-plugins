---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## 项目概述

**Feagent** 是企业级AI Agent编排与执行平台，基于 FastAPI + LangChain + DDD-lite 架构。

**当前阶段**: 多Agent协作系统（Phase 8+ - Unified Definition System）
- 三Agent架构：CoordinatorAgent、ConversationAgent、WorkflowAgent
- EventBus事件驱动通信
- 八段压缩器（PowerCompressor）
- WebSocket实时通道
- 可配置规则引擎与干预系统
- 自描述节点验证与依赖图

---

## Claude ↔ Codex 协作工作流（精简版）

1) 需求理解 → Claude 快速识别疑问 → Codex 深度推理
2) 上下文收集 → Codex 全面检索 → 输出分析报告
3) 任务规划 → Claude 基于分析制定计划
4) 代码执行 → Claude 直接编码（遇复杂逻辑调用 Codex）
5) 质量审查 → Codex 深度审查 → Claude 最终决策

**角色分工 / 产出**
- Claude：提炼问题、制定计划、落地代码与决策
- Codex：深度推理/检索、给出代码原型（统一 diff 参考）、质量审查
- 产出物：分析报告 → 计划 → 代码原型参考 → 落地实现 → Codex Review 意见 → Claude 采纳/决策

**与现有规则/架构的对齐**
- 对应“架构顺序”：需求分析→Domain→Ports→Infrastructure→Application→Interface；在 Domain/Ports 阶段优先让 Codex 做深推与检索。
- 代码执行阶段继续遵守“每次最多改 2 个文件 + TDD”与命名约定。
- 前端/后端改动时保持原有项目结构；Codex 仅给出参考 patch，真实修改由 Claude 完成。
- 关于 Codex 详细调用规范与合作要求，沿用文末《Core Instruction for CodeX MCP》与《Codex Tool Invocation Specification》。

---

## 关键规则（必读）

### 开发约束

1. **开发节奏**：每次最多修改2个文件，等待用户确认后继续

2. **TDD强制**：Red → Green → Refactor
   - Domain层覆盖率 ≥ 80%
   - Application层覆盖率 ≥ 70%

3. **架构顺序（严格）**：
   ```
   需求分析 → Domain → Ports → Infrastructure → Application → Interface
   ```

4. **依赖方向（单向）**：
   ```
   Interface → Application → Domain ← Infrastructure
   ```
   **Domain层禁止导入**: SQLAlchemy、FastAPI、LangChain 或任何框架

### 命名约定

| 模式 | 含义 | 示例 |
|------|------|------|
| `get_xxx` | 必须存在，否则抛异常 | `get_agent(id)` |
| `find_xxx` | 允许返回None | `find_agent(id)` |
| `XxxUseCase` | 应用层用例 | `CreateAgentUseCase` |
| `XxxInput/Request/Response` | DTO | `CreateAgentInput` |

---

## 开发命令

### 后端

```bash
# 安装依赖
pip install -e ".[dev]"

# 数据库迁移
alembic upgrade head
alembic revision --autogenerate -m "description"

# 启动服务器（Windows 必须使用 python -m）
python -m uvicorn src.interfaces.api.main:app --reload --port 8000

# 测试
pytest                                              # 全部测试
pytest tests/unit                                   # 单元测试
pytest tests/integration                            # 集成测试
pytest tests/unit/domain/entities/test_agent.py -v # 单个文件
pytest -k "test_create_agent"                       # 按名称

# 代码质量
ruff check .                             # Lint
ruff format .                            # Format
pyright src/                             # 类型检查
```

> **Windows 注意**：必须使用 `python -m uvicorn` 而非直接 `uvicorn`，以确保 watchfiles shim 正确加载，避免 Ctrl+C 信号问题。

### 前端

```bash
cd web
pnpm install
pnpm dev      # 开发服务器
pnpm build    # 构建
pnpm lint     # Lint
```

---

## 架构概览

### 四层架构

```
┌─────────────────────────────────────────────────┐
│              Interface Layer (接口层)            │
│  FastAPI Routes + DTO (Pydantic)                │
└────────────────────────┬────────────────────────┘
                         │
┌────────────────────────▼────────────────────────┐
│          Application Layer (应用层)              │
│  UseCases: 用例编排、事务边界                     │
└────────────────────────┬────────────────────────┘
                         │ Ports (Protocol)
┌────────────────────────▼────────────────────────┐
│            Domain Layer (领域层)                 │
│  Entities + Agents + Services                   │
│  ❌ 禁止导入任何框架                              │
└────────────────────────┬────────────────────────┘
                         │ Adapters
┌────────────────────────▼────────────────────────┐
│       Infrastructure Layer (基础设施层)          │
│  ORM + Repository + External Services           │
└─────────────────────────────────────────────────┘
```

### 多Agent协作架构

```
┌─────────────────────────────────────────────────────────────┐
│                        用户交互层                            │
│     对话面板 (Chat)  ◄──────►  工作流画布 (Canvas)           │
└───────────────────────────┬─────────────────────────────────┘
                            │ WebSocket
┌───────────────────────────▼─────────────────────────────────┐
│                      Agent 协作层                            │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              CoordinatorAgent (协调者)               │   │
│  │  规则引擎 │ 上下文服务 │ 压缩器服务 │ 子Agent调度    │   │
│  └─────────────────────────┬───────────────────────────┘   │
│                            │                                │
│     ┌──────────────────────┼──────────────────────┐        │
│     ▼                      ▼                      ▼        │
│  ConversationAgent     EventBus            WorkflowAgent   │
│  (意图分类/ReAct推理)  (事件总线)          (节点执行)       │
└─────────────────────────────────────────────────────────────┘
```

**核心组件位置**：
| 组件 | 路径 | 职责 |
|------|------|------|
| CoordinatorAgent | `src/domain/agents/coordinator_agent.py` | 规则验证、上下文管理、子Agent调度 |
| ConversationAgent | `src/domain/agents/conversation_agent.py` | 意图分类、ReAct推理、决策生成 |
| WorkflowAgent | `src/domain/agents/workflow_agent.py` | 节点执行、DAG拓扑排序、状态同步 |
| EventBus | `src/domain/services/event_bus.py` | 事件发布/订阅、中间件链 |
| PowerCompressor | `src/domain/services/power_compressor.py` | 八段压缩（多Agent协作场景） |
| RuleEngineFacade | `src/domain/services/rule_engine_facade.py` | 规则引擎统一入口（Facade模式） |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DSGWJQ/Feagent](https://github.com/DSGWJQ/Feagent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
