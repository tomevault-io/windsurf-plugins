---
trigger: always_on
description: Agent 模块是 RAG Agent Platform 的核心智能体系统，负责智能对话、工具调用、任务编排与执行追踪。基于 LangChain4j 框架，实现了可配置、可扩展、可追踪的智能体生命周期管理。
---

# Agent 模块技术文档

## 1. 模块概述

Agent 模块是 RAG Agent Platform 的核心智能体系统，负责智能对话、工具调用、任务编排与执行追踪。基于 LangChain4j 框架，实现了可配置、可扩展、可追踪的智能体生命周期管理。

### 1.1 核心能力

- **智能体生命周期管理**: Agent 创建、版本发布、启用/禁用、删除
- **版本化机制**: 支持多版本管理，草稿编辑、审核发布、版本回滚
- **工具集成**: 集成 MCP 工具，支持动态工具加载与调用
- **RAG 增强**: 集成知识库检索，支持检索增强生成
- **执行链路追踪**: 完整记录每次 Agent 执行的详细过程
- **多模态支持**: 支持文本、图像等多模态输入与处理
- **会话管理**: 管理用户与 Agent 的多轮对话会话
- **任务编排**: 支持复杂任务的分解与并行执行

### 1.2 技术栈

- **核心框架**: LangChain4j (Agent 编排框架)
- **模型调用**: 统一 LLM 调用接口 (支持多模型提供商)
- **工具协议**: MCP (Model Context Protocol)
- **存储**: PostgreSQL (Agent 配置与执行记录)
- **消息队列**: RabbitMQ (异步任务处理)

---

## 2. 核心功能

### 2.1 Agent 生命周期

#### 2.1.1 Agent 创建与配置

Agent 由以下核心配置组成:

```
Agent 配置结构
├── 基础信息
│   ├── name: Agent 名称
│   ├── avatar: Agent 头像
│   └── description: Agent 描述
├── 提示词配置
│   ├── system_prompt: 系统提示词 (定义 Agent 角色与行为)
│   └── welcome_message: 欢迎消息
├── 能力配置
│   ├── tool_ids: 可使用的工具列表 (MCP 工具)
│   ├── knowledge_base_ids: 关联的知识库 (RAG 功能)
│   ├── tool_preset_params: 工具预设参数
│   └── multi_modal: 是否支持多模态
└── 版本控制
    ├── published_version: 当前发布的版本ID
    └── enabled: Agent 启用状态
```

**设计要点**:
- **分离草稿与发布**: `agents` 表存储当前工作草稿，`agent_versions` 表存储已发布的不可变版本
- **多租户隔离**: 每个 Agent 关联 `user_id`，实现租户级隔离
- **工具与知识库解耦**: 通过 JSON 数组引用外部资源ID，支持动态配置

#### 2.1.2 版本发布流程

```
版本发布状态流转
┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐
│ 草稿编辑 │────→│ 提交审核 │────→│ 审核通过 │────→│ 已发布   │
└──────────┘     └──────────┘     └──────────┘     └──────────┘
                       │                                  │
                       ├─────────┐                        │
                       │ 审核拒绝 │                        │
                       └─────────┘                        │
                                                          │
                       ┌──────────┐                       │
                       │ 已下架   │←──────────────────────┘
                       └──────────┘

状态说明:
- 1-审核中 (Pending Review)
- 2-已发布 (Published)
- 3-拒绝 (Rejected)
- 4-已下架 (Unpublished)
```

**版本化优势**:
- **不可变性**: 已发布版本不可修改，确保线上稳定性
- **快速回滚**: 切换 `published_version` 即可回滚到历史版本
- **变更追踪**: `change_log` 记录每个版本的更新内容
- **A/B 测试**: 不同用户可使用不同版本进行灰度发布

### 2.2 Agent 执行引擎

#### 2.2.1 执行链路架构

```
用户输入
   │
   ▼
┌─────────────────────────────────────────────────────────┐
│                   Agent 执行引擎                          │
├─────────────────────────────────────────────────────────┤
│  1. 会话上下文加载                                         │
│     - 加载历史消息 (messages 表)                           │
│     - 构建上下文窗口 (滑动窗口 + Token 管控)                │
│  ─────────────────────────────────────────────────────  │
│  2. RAG 检索 (如果配置了知识库)                            │
│     - 向量检索相关文档片段                                  │
│     - 注入到 System Prompt 或 User Message                │
│  ─────────────────────────────────────────────────────  │
│  3. LLM 推理                                              │
│     - 使用配置的模型进行推理                                │
│     - 支持多模态输入 (文本 + 图片)                          │
│     - 生成响应或工具调用指令                                │
│  ─────────────────────────────────────────────────────  │
│  4. 工具调用 (如果 LLM 返回 Tool Call)                     │
│     - 解析工具名称与参数                                    │
│     - 通过 MCP 协议调用工具                                 │
│     - 记录调用耗时、成功率                                  │
│  ─────────────────────────────────────────────────────  │
│  5. 多轮迭代 (ReAct 模式)                                  │
│     - 将工具结果返回给 LLM                                  │
│     - LLM 决定继续调用工具或生成最终答案                     │
│     - 最多迭代 N 轮 (防止死循环)                            │
│  ─────────────────────────────────────────────────────  │
│  6. 响应返回                                              │
│     - 保存消息到 messages 表                               │
│     - 更新会话元数据                                        │
│     - 记录 Token 消耗与成本                                 │
└─────────────────────────────────────────────────────────┘
   │
   ▼
用户收到响应
```

#### 2.2.2 执行追踪设计

**双表追踪模型**:

1. **汇总表 (agent_execution_summary)**:
   - 记录每次完整执行的汇总信息
   - `trace_id` 唯一标识一次执行
   - 统计总耗时、Token 消耗、工具调用次数、总成本

2. **详情表 (agent_execution_details)**:
   - 记录每个执行步骤的详细信息
   - 与汇总表通过 `trace_id` 关联
   - `sequence_no` 保证步骤顺序

**追踪数据结构**:

```sql
-- 汇总表示例数据
trace_id: "trace-abc123"
user_id: "user-001"
session_id: "session-xyz"
agent_id: "agent-456"
execution_start_time: "2025-12-08 10:30:00"
execution_end_time: "2025-12-08 10:30:15"
total_execution_time: 15000  -- 15秒
total_input_tokens: 500
total_output_tokens: 800
total_tokens: 1300
tool_call_count: 2
total_tool_execution_time: 3000  -- 工具调用耗时3秒
total_cost: 0.0026
execution_success: true

-- 详情表示例数据 (同一 trace_id 的多条记录)
-- Step 1: 用户消息
{
  trace_id: "trace-abc123",
  sequence_no: 1,
  step_type: "USER_MESSAGE",
  message_content: "帮我查一下明天的天气",
  message_type: "USER_MESSAGE"
}

-- Step 2: LLM 决定调用工具
{
  trace_id: "trace-abc123",
  sequence_no: 2,
  step_type: "TOOL_CALL",
  tool_name: "weather_api",
  tool_request_args: '{"city": "北京", "date": "2025-12-09"}',
  tool_response_data: '{"temp": "5°C", "weather": "晴"}',
  tool_execution_time: 1200,
  tool_success: true,
  model_id: "Qwen/Qwen2.5-72B-Instruct",
  message_tokens: 50
}


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NEDONION/rag-agent-platform](https://github.com/NEDONION/rag-agent-platform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
