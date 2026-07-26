---
trigger: always_on
description: Agent 是 Agentica 的核心组件——它将模型、工具和记忆连接在一起，能够思考、决策和执行动作。
---

# Agent

Agent 是 Agentica 的核心组件——它将模型、工具和记忆连接在一起，能够思考、决策和执行动作。

## 基本结构

```
Agent
├── Model          # 大脑：LLM 提供推理能力
├── Tools          # 双手：与外部世界交互的能力
├── Memory         # 运行时记忆：当前会话消息历史
├── Workspace      # 持久化记忆：跨会话长期记忆
├── Knowledge      # 知识库：外部文档检索 (RAG)
├── Instructions   # 指令：定义 Agent 的行为和风格
└── Runner         # 执行引擎（自动创建，无需手动管理）
```

## 创建 Agent

### 最小示例

```python
from agentica import Agent, ZhipuAI

agent = Agent(model=ZhipuAI())
result = agent.run_sync("一句话介绍北京")
print(result.content)
```

### 完整配置示例

```python
import asyncio
from agentica import Agent, OpenAIChat
from agentica.tools.duckduckgo_tool import DuckDuckGoTool
from agentica.tools.url_crawler_tool import UrlCrawlerTool
from agentica.agent.config import PromptConfig, ToolConfig, WorkspaceMemoryConfig
from agentica import Workspace

async def main():
    agent = Agent(
        # 核心定义
        name="Research Assistant",
        model=OpenAIChat(id="gpt-4o"),
        instructions=[
            "你是一个专业的研究助手，擅长收集和整理最新信息",
            "搜索时使用多个关键词进行验证，确保信息准确",
            "用中文回答，条理清晰，附上信息来源",
        ],
        tools=[DuckDuckGoTool(), UrlCrawlerTool()],

        # Prompt 配置
        prompt_config=PromptConfig(
            markdown=True,               # 输出 Markdown 格式
            add_datetime_to_instructions=True,  # 注入当前日期
        ),

        # 工具配置
        tool_config=ToolConfig(
            tool_call_limit=20,          # 单次最多调用 20 次工具
            compress_tool_results=True,  # 自动压缩大工具结果
        ),

        # 长期记忆
        workspace=Workspace("./workspace"),
        long_term_memory_config=WorkspaceMemoryConfig(
            load_workspace_memory=True,
            max_memory_entries=5,
        ),

        # 会话历史
        add_history_to_context=True,
        num_history_turns=5,
    )

    result = await agent.run("2025 年 AI 领域最重要的进展有哪些？")
    print(result.content)

asyncio.run(main())
```

## 参数分层

### 第一层：核心定义

| 参数 | 类型 | 说明 |
|------|------|------|
| `model` | `Model` | LLM 模型实例（必填） |
| `name` | `str` | Agent 名称，注入到 System Prompt |
| `description` | `str` | Agent 描述，作为 System Prompt 开头 |
| `instructions` | `str \| List[str] \| Callable` | 行为指令，支持动态计算 |
| `tools` | `List[Tool \| Callable]` | 工具列表 |
| `knowledge` | `Knowledge` | RAG 知识库 |
| `workspace` | `Workspace` | 持久化工作空间 |
| `session_id` | `str` | 会话 ID（用于会话持久化和恢复） |
| `response_model` | `Type[BaseModel]` | 结构化输出（Pydantic 模型） |

### 第二层：会话历史

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `enable_long_term_memory` | `bool` | `False` | 启用长期记忆工具、检索和相关 hooks |
| `enable_experience_capture` | `bool` | `False` | 启用 experience 捕获与自进化 hooks |
| `add_history_to_context` | `bool` | `False` | 将历史消息加入上下文 |
| `num_history_turns` | `int` | `3` | 保留最近 N 轮历史 |
| `use_structured_outputs` | `bool` | `False` | 严格 JSON 结构化输出（部分 API 支持） |
| `debug` | `bool` | `False` | 启用调试日志 |
| `enable_tracing` | `bool` | `False` | 启用 Langfuse tracing |

### 第三层：打包配置

| 参数 | 类型 | 说明 |
|------|------|------|
| `prompt_config` | `PromptConfig` | 提示词工程配置 |
| `tool_config` | `ToolConfig` | 工具调用行为配置 |
| `long_term_memory_config` | `WorkspaceMemoryConfig` | 工作空间记忆配置 |

## 运行方式

Agentica 采用 **async-first** 架构：

```python
import asyncio
from agentica import Agent, ZhipuAI

agent = Agent(model=ZhipuAI())

# 1. 异步非流式（推荐，生产环境使用）
async def example_async():
    result = await agent.run("你好")
    print(result.content)

# 2. 异步流式（实时显示，长响应适用）
async def example_stream():
    async for chunk in agent.run_stream("写一首诗"):
        if chunk.content:
            print(chunk.content, end="", flush=True)

# 3. 同步非流式（脚本/Jupyter 适用）
result = agent.run_sync("你好")
print(result.content)

# 4. 同步流式（Jupyter / 非 async 环境）
for chunk in agent.run_stream_sync("写一首诗"):
    if chunk.content:
        print(chunk.content, end="", flush=True)

asyncio.run(example_async())
```

| 方法 | 类型 | 适用场景 |
|------|------|----------|
| `run(msg)` | `async` | FastAPI、异步服务、生产环境 |
| `run_stream(msg)` | `async generator` | 实时流式响应 |
| `run_sync(msg)` | `sync` | 脚本、Jupyter、CLI |
| `run_stream_sync(msg)` | `sync iterator` | 同步代码中的流式输出 |
| `print_response(msg)` | `async` | 快速测试（自带格式化打印） |

## Instructions（指令）

指令定义 Agent 的行为、风格和约束。

### 静态指令

```python
agent = Agent(
    instructions=[
        "你是一个 Python 代码审查专家",
        "重点检查：安全漏洞、性能瓶颈、代码可读性",
        "发现问题时，提供具体的修复建议和代码示例",
        "使用中文回复，格式清晰",
    ],
)
```

### 动态指令（基于上下文计算）

```python
from datetime import date

def get_instructions(agent) -> list:
    """运行时动态计算指令"""
    instructions = ["你是一个智能助手"]
    today = date.today()
    if today.weekday() >= 5:  # 周末
        instructions.append("今天是周末，优先考虑娱乐和生活类问题")
    if agent.session_state.get("expert_mode"):
        instructions.append("使用专业术语，提供深度分析")
    return instructions

agent = Agent(instructions=get_instructions)
# 每次 run() 前，Callable 会被调用并传入 agent 实例
```

### 完整 System Prompt

如果需要完全控制 System Prompt（绕过 Agentica 的 prompt 组装逻辑）：

```python
from agentica.agent.config import PromptConfig

agent = Agent(
    prompt_config=PromptConfig(
        system_prompt="你是一个友好的中文助手。回答要简洁准确，控制在 200 字以内。",
    ),
)

# 也可以是 Callable，运行时动态计算：
agent = Agent(
    prompt_config=PromptConfig(
        system_prompt=lambda agent: f"你是 {agent.name}，今天是{date.today()}。",
    ),
)
```

## PromptConfig（进阶）

`PromptConfig` 控制 System Prompt 的构建细节：

```python

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shibing624/agentica](https://github.com/shibing624/agentica) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
