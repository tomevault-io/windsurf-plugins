---
trigger: always_on
description: 可扩展的 AI Agent 框架，专注于深度研究任务。基于 MCP 工具协议，支持多 LLM 提供商。
---

# Mem Deep Research Framework

可扩展的 AI Agent 框架，专注于深度研究任务。基于 MCP 工具协议，支持多 LLM 提供商。

## 项目结构

```
mem_deep_research_core/              # 框架核心代码
├── deep_research.py                 # 主入口 (DeepResearch 类)
├── config_schema.py                 # Pydantic 配置验证
├── exceptions.py                    # 异常定义
├── core/                            # 核心模块
│   ├── orchestrator.py              # Agent 编排器（组件初始化 + 任务协调）
│   ├── main_loop.py                 # 主执行循环 (MainLoopRunner + MainLoopContext)
│   ├── prompt_builder.py            # Prompt 构建（system prompt + skill 注入 + hint）
│   ├── constants.py                 # 框架常量 + 工具函数（单一来源）
│   ├── context_manager.py           # 上下文管理（masking + dedup + source registry）
│   ├── window_strategy.py           # 窗口压缩策略（ObservationMasking/LLMSummarize/BinaryReduction）
│   ├── hooks.py                     # 钩子系统（HookRegistry, HookContext）
│   ├── secure_context.py            # 隐私数据保护（_secure 字段 → 占位符）
│   ├── tool_executor.py             # 工具执行器
│   ├── tool_result_formatter.py     # 工具结果格式化
│   ├── llm_call_handler.py          # LLM 调用 + 重试 + 摘要生成
│   ├── sub_agent_runner.py          # 子 Agent（复用 MainLoopRunner，隔离上下文）
│   ├── stream_handler.py            # SSE 流式输出
│   ├── monitoring.py                # 执行监控 + 循环检测
│   ├── task_planner.py              # LLM 任务分解（模板化）
│   ├── memory.py                    # 记忆系统（SessionMemory + LongTermMemory）
│   ├── todo_tracker.py              # 任务追踪（内置 update_todo 工具）
│   ├── pipeline.py                  # 任务执行 Pipeline（组件初始化 + 错误处理）
│   ├── agent_factory.py             # Agent 工厂（统一创建 Orchestrator/LLM/ToolManager）
│   ├── deferred_tools.py            # 延迟工具加载（工具数超阈值时仅暴露名称+描述）
│   ├── input_compiler.py            # 输入编译链（URL 提取 + @file 展开 + on_query_compile hook）
│   ├── transcript.py                # 结构化事件日志（UUID + 类型 + 时间戳，JSONL 导出）
│   ├── message_utils.py             # 消息历史纯函数（提取工具名、hash/去重）
│   ├── message_interceptor.py       # 消息拦截
│   ├── user_context.py              # 用户上下文构建（可选工具类，通过 hook 注入）
│   ├── answer_handler.py            # 最终答案后处理
│   └── interceptor_config.py        # 拦截器配置 + 预设
├── llm/                             # LLM 客户端
│   ├── provider_client_base.py      # Provider 基类
│   └── providers/                   # 多 Provider 实现
├── prompts/                         # Prompt 系统
│   ├── agent_prompt.py              # AgentPrompt 统一类
│   ├── template_loader.py           # 模板加载器
│   └── templates/                   # Markdown 模板（base/presets/planning/reflection/...）
├── tool/                            # 工具模块
│   ├── manager.py                   # ToolManager（MCP 工具管理）
│   └── mcp_servers/                 # 内置 MCP 服务器
├── skills/                          # Skill 系统
│   ├── matcher.py                   # 规则匹配 + 注入
│   ├── llm_selector.py              # LLM Skill 选择
│   └── inline_selector.py           # Inline Skill 选择（零额外开销）
├── utils/
│   ├── external_loader.py           # 配置加载器（全局 config_loader/external_loader）
│   ├── tool_utils.py                # 工具辅助
│   ├── parsing_utils.py             # JSON 解析工具（智能截断、JSON5 支持）
│   ├── io_utils.py                  # 输入输出工具（用户输入处理、文件引用）
│   ├── summary_utils.py             # 摘要生成工具
│   └── stream_parsing_utils.py      # 流式解析（StructuredTagExtractor, TextInterceptor）
└── mem_deep_research_logging/       # 日志基础设施
    ├── logger.py                    # Bootstrap logger（级别控制）
    └── task_tracer.py               # 任务执行追踪（StepRecord 结构化日志）
config/                              # 框架默认配置
├── agent_example.yaml               # Agent 配置示例
├── tool/                            # 内置工具配置 YAML
└── skills/definitions/              # Skill 定义
tests/                               # 测试
```

## 核心概念

### 执行流程

```
DeepResearch.run(query)
  → Pipeline.run()
    → AgentFactory 创建 Orchestrator + LLM Client + ToolManager
      → Orchestrator.run_main_agent():
          1. PromptBuilder 构建 system prompt + skill 注入
          2. 创建 MainLoopRunner
          → MainLoopRunner.run():
              while turn < max_turns:
                1. on_agent_start hook（首轮）+ 语言自动检测
                2. Monitor.pre_turn_check()
                3. LLM 调用（on_before_llm_call / on_after_llm_call guardrail）
                4. Monitor.post_turn_check()（循环检测 + 升级策略）
                5. Inline Skill: 解析 <next_skills>
                6. 解析工具调用 + 跨轮次去重
                7. 执行工具（SecureContext 自动解占位符）
                   - 普通工具 → ToolExecutor
                   - 子 Agent → SubAgentRunner → MainLoopRunner（隔离上下文）
                8. Context 管理（ObservationMasking → LLMSummarize → BinaryReduction）
                9. Hook: on_turn_end
               10. 反思检查点（deep_research 模式）
          → 生成最终摘要（SummaryHandler）
      → post_process_final_answer → ResearchResult
```

### 使用方式

```python
from mem_deep_research import DeepResearch

# 方式 1: 从项目目录加载
dr = DeepResearch.from_project("./my_project")
result = await dr.run("研究任务")

# 方式 2: 代码配置
dr = DeepResearch(
    llm_provider="anthropic",
    model="Codex-sonnet-4-20250514",
    api_key="your-key",
)
result = await dr.run("任务")

# 同步
result = dr.run_sync("任务")
```

### 项目目录结构

用户项目通过 `DeepResearch.from_project()` 加载：

```
my_project/
├── config/
│   ├── agent.yaml              # Agent 配置（LLM、工具、参数）
│   ├── tool/                   # 项目级工具配置（覆盖框架默认）
│   ├── skills/definitions/     # 项目级 Skill 定义

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cjhyy/mem-deep-research](https://github.com/cjhyy/mem-deep-research) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
