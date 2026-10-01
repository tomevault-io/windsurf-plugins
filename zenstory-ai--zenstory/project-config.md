---
trigger: always_on
description: 本文档描述 zenstory Agent 系统的架构、流程和关键组件。
---

# Agent 模块架构文档

本文档描述 zenstory Agent 系统的架构、流程和关键组件。

## 目录结构

```
agent/
├── service.py              # 主服务入口
├── suggest_service.py      # 智能建议生成服务
├── stream_adapter.py       # LangGraph 事件适配器
├── context/                # 上下文组装模块
│   ├── assembler.py        # 上下文组装器
│   ├── budget.py           # Token 预算管理
│   ├── compaction.py       # 上下文压缩（长会话总结）
│   └── prioritizer.py      # 优先级管理
├── core/                   # 核心基础设施
│   ├── events.py           # SSE 事件定义
│   ├── llm_client.py       # OpenAI 兼容 LLM 客户端
│   ├── message_manager.py  # 消息和系统提示管理
│   ├── session_loader.py   # 会话加载器
│   └── stream_processor.py # 文件流处理器
├── graph/                  # LangGraph 工作流
│   ├── state.py            # 工作流状态定义
│   ├── writing_graph.py    # 图执行入口
│   ├── nodes.py            # 流式节点实现
│   └── router.py           # 意图路由
├── llm/                    # LLM 集成
│   └── openai_agents/     # openai-agents-python / DeepSeek 写作 Agent 适配层
├── prompts/                # 提示词模板
│   ├── base.py             # 基础提示
│   ├── novel.py            # 小说项目提示
│   ├── screenplay.py       # 剧本项目提示
│   ├── short_story.py      # 短篇故事提示
│   ├── subagents.py        # 子代理提示 (planner/writer/quality_reviewer)
│   └── suggestions.py      # 建议生成提示
├── schemas/                # 数据模型
│   ├── context.py          # 上下文数据模型
├── skills/                 # 技能系统（标准 SKILL.md，渐进式加载，永不执行脚本）
│   ├── active_skills.py    # 当前用户启用中的技能视图（目录/工具/显式选择共用）
│   ├── context_injector.py # L1 技能目录（只含名称 + 用途）
│   ├── loader.py           # 内置技能加载（builtin/<id>/SKILL.md）
│   ├── package.py          # SKILL.md 解析、zip 导入安全检查、导出打包
│   └── builtin/            # 官方技能（由 services/builtin_skill_seed.py 写入 PublicSkill）
└── tools/                  # 工具实现
    ├── tool_schemas.py     # provider-neutral 工具 schema 定义
    ├── file_executor.py    # 文件操作执行器
    ├── mcp_tools.py        # MCP 格式工具函数
    └── permissions.py      # 权限检查
```

## 核心流程

### 1. 请求处理流程

```
用户消息
    │
    ▼
┌─────────────────────────────────────────────────────────┐
│  AgentService.process_stream() [service.py]             │
│  - 设置 ToolContext                                      │
│  - 组装上下文 (ContextAssembler)                         │
│  - 加载会话历史 (SessionLoader)                          │
│  - 构建系统提示 (MessageManager)                         │
│  - 调用工作流                                            │
└─────────────────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────────────────┐
│  run_writing_workflow_streaming() [writing_graph.py]    │
│  - 路由策略选择初始 agent（默认 llm，可配置 off）           │
│  - 循环执行 agent 直到完成或达到最大迭代次数              │
└─────────────────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────────────────┐
│  router (llm / off) [router.py]                         │
│  - llm: 调用 router_node()（DeepSeek Chat Completions）   │
│  - off: 固定从 writer 开始                               │
│  - 返回: initial_agent + workflow_plan + workflow_agents │
└─────────────────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────────────────┐
│  run_streaming_agent() [nodes.py]                       │
│  - 组合基础提示 + 专业 agent 提示                        │
│  - 调用 openai_agents.runner                            │
│  - 通过 openai-agents-python 处理工具调用和 handoff       │
└─────────────────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────────────────┐
│  StreamAdapter.adapt_langgraph_events() [stream_adapter]│
│  - 转换 LangGraph 事件为 SSE 事件                        │
│  - 处理文件流式写入 (<file>...</file>)                   │
│  - 发送事件到前端                                        │
└─────────────────────────────────────────────────────────┘
```

### 2. 多 Agent 协作流程

```
┌─────────────┐
│   Router    │ ─── 分析意图，确定工作流
└─────────────┘
       │
       ▼
   ┌───────────────────────────────────────┐
   │         工作流类型 (workflow_plan)      │
   ├───────────────────────────────────────┤
   │ quick       : writer（必要时再 review）  │
   │ standard    : planner → writer（必要时再 review）│
   │ full        : planner → hook_designer → writer（必要时再 review）│
   │ hook_focus  : hook_designer → writer（必要时再 review）│
   │ review_only : quality_reviewer          │
   └───────────────────────────────────────┘
       │
       ▼
┌─────────────┐     handoff      ┌─────────────┐     handoff      ┌─────────────┐
│   Planner   │ ───────────────► │   Writer    │ ───────────────► │ Quality Reviewer │
│  大纲规划师  │                  │  内容创作者  │                  │   质量审稿人     │
└─────────────┘                  └─────────────┘                  └─────────────┘
```

### 3. Agent 交接机制

Agent 可以通过两种方式交接：

1. **显式 handoff**: Agent 调用 `handoff_to_agent` 工具
2. **工作流自动交接**: 按照 router 规划的 workflow_agents 顺序执行

## 关键组件详解

### AgentService (service.py)

主服务类，处理用户消息的流式响应。

```python
async def process_stream(
    session: Session,
    project_id: str,
    user_message: str,
    ...
) -> AsyncIterator[str]:
    # 1. 设置工具上下文
    ToolContext.set_context(session, user_id, project_id, session_id)

    # 2. 组装项目上下文
    context_data = session_loader.assemble_context(...)

    # 3. 构建系统提示
    system_prompt = message_manager.build_system_prompt(...)

    # 4. 执行工作流
    async for event in run_writing_workflow_streaming(state):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zenstory-ai/zenstory](https://github.com/zenstory-ai/zenstory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
