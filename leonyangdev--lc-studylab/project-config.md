---
trigger: always_on
description: > 一个帮你「看文档、做 RAG、深度调研、规划任务」的一体化智能体系统，用来**把 LangChain v1 + LangGraph + DeepAgents 能玩的东西都串起来**。
---


# LC-StudyLab 智能学习 & 研究助手需求说明

## 0. **「LC-StudyLab 智能学习 & 研究助手」**

一句话：

> 一个帮你「看文档、做 RAG、深度调研、规划任务」的一体化智能体系统，用来**把 LangChain v1 + LangGraph + DeepAgents 能玩的东西都串起来**。

核心场景：

- 日常聊天 + 工具调用（create_agent）
- 对本地/在线文档做 RAG 问答与总结
- 启动「深度研究模式」，自动查网、读文档、分工给子智能体写研究报告（DeepAgents）
- 用 LangGraph 搭一个可视化的「学习/研究工作流」：可暂停、可回滚、带记忆

技术栈默认：Python + `langchain` v1 + `langgraph` v1 + `deepagents` + `langchain-openai` 等。

---

## 1. 你要覆盖的 LangChain 生态能力地图

简单先把「文档特性 → 项目里放哪」说清楚：

1. **LangChain v1 核心**

   - 模型统一接口、Runnable / LCEL、流式输出、工具调用、结构化输出、Guardrails、RAG、Agents ([LangChain Docs][1])
     → 用在所有对话 / RAG / 安全过滤上。

2. **LangGraph v1**

   - State / Node / Edge、自定义图、Checkpointer、Memory、Streaming、Human-in-the-loop ([LangChain Docs][2])
     → 用来做「学习任务工作流」和「复杂 RAG Agent 图」。

3. **DeepAgents**

   - 深度规划、多轮子任务、文件系统工具、SubAgent 中间件、长程记忆、Human-in-the-loop 研究模式 ([LangChain Docs][3])
     → 用一个「深度研究模式」单独练。

4. **Guardrails / 安全**

   - LangChain Guardrails（输入/输出过滤、策略）、或集成 GuardrailsAI / Pangea / NeMo Guardrails 等第三方 ([LangChain Docs][4])
     → 用在对话 & RAG 输出上，顺手练「安全 + 结构化输出」。

5. **RAG / 向量库**

   - Document Loaders、Text Splitters、Vector Stores、Retrievers、Agentic RAG（LangGraph 版） ([LangChain Docs][5])
     → 「文档问答」子系统。

你可以把项目理解成：**一套统一 API / CLI / Web UI，后面挂着三类智能体：普通 Agent、RAG Agent、Deep Agent**。

---

## 2. 项目结构建议（一个 repo 玩全家桶）

示意结构（Python backend 角度）：

```text
lc-studylab/
  backend/  # 后端项目
    pyproject.toml / requirements.txt

    config/
      settings.py              # 模型、向量库、数据路径等统一配置
      logging.py

    core/
      models.py                # openai:gpt-4o, gpt-5 等模型封装
      prompts.py               # 系统提示词、模板
      tools/
        web_search.py          # 网络搜索工具（如 tavily）
        filesystem.py          # 读写本地知识库的工具
        rag_tools.py           # retriever 封装成 tool
        task_tools.py          # 创建学习任务、记录进度之类
      guardrails/
        content_filters.py     # 简单 guardrails，实现输入/输出校验
        schemas.py             # Pydantic / JSON schema 用于结构化输出

    rag/
      loaders.py               # 各种文档加载
      index_builder.py         # text split + embedding + vector store
      retriever.py             # retriever 构造
      agent.py                 # RAG Agent (LangChain create_agent + retriever tool)
      graph.py                 # Agentic RAG 的 LangGraph 图 (可选)

    agents/
      base_agent.py            # 通用 create_agent 封装，带 streaming + guardrails
      study_planner_agent.py   # 学习计划/任务规划 Agent
      coding_helper_agent.py   # 简单代码助手 Agent（练工具调用）

    workflows/  (LangGraph)
      study_flow_graph.py      # 「从问题 → 找资料 → 生成学习计划 → 生成练习题」的图
      eval_quiz_graph.py       # 「答题 → 评分 → 反馈」图（可选）

    deep_research/
      deep_agent.py            # DeepAgents create_deep_agent，深度研究模式
      middleware.py            # subagents / filesystem / 人类在环 中间件配置

    api/
      http_server.py           # FastAPI / Flask 暴露 HTTP 接口
      routers/
        chat.py                # /chat -> 统一入口，内部路由到不同 Agent / Graph / DeepAgent
        rag.py                 # /rag-index, /rag-query
        deep_research.py       # /deep-research

    scripts/
      build_index.py           # 初次构建向量索引
      demo_cli.py              # CLI 入口，方便你本地调试

  frontend/ # 前端项目
```

后面你学每一章文档，都能「往这个项目某个模块里加东西」，而不是练完就丢。

---

## 3. 分阶段路线：按文档章节循序渐进

### 第 1 阶段：基础 Agent + Streaming + 工具

**目标：** 做出一个「通用聊天 + 工具调用」的智能体，完全基于 `create_agent`，支持流式输出。
重点练：**LangChain v1 概念 & Agents & Streaming**。([LangChain Docs][1])

功能：

- `/chat` 接口：

  - 默认使用 `create_agent(model="openai:gpt-4o", tools=[web_search, get_time,...])`
  - 支持 `stream=True`，前端或 CLI 一边打印 token，一边看 reasoning & tool calls 流。

实现要点（放到 `core/` + `agents/base_agent.py`）：

1. 封装模型

   - `langchain-openai` 或其他 provider 封装 chat model。

2. `tools` 模块：

   - `@tool` 写 2–3 个真实工具（如：当前时间、简单计算、网络搜索）。

3. `create_agent`：

   - `system_prompt` 写清「你是学习助手」「工具使用说明」。
   - 打开 streaming，体验 v1 的 streaming event（tokens + tool calls + reasoning traces）。([LangChain Docs][6])

这一阶段你可以只管「直接用 create_agent」——内部已经帮你接好了 LangGraph 的循环执行逻辑。([LangChain Docs][7])

---

### 第 2 阶段：RAG 知识库模块（向量库 + Retrievers + RAG Agent）

**目标：** 完整实现「上传文档 / 指定文件夹 → 构建向量索引 → RAG 问答 Agent」。
重点练：**Retrieval / VectorStores / Agentic RAG**。([LangChain Docs][5])

功能设计：

- `/rag-index`：

  - 读取指定目录 PDF / Markdown / MDX / HTML。
  - 用 text splitter 分 chunk。
  - 用 OpenAI embeddings + InMemoryVectorStore / FAISS / Milvus 建索引。

- `/rag-query`：

  - 基于 retriever 的 RAG Agent：

    - retriever 封装成一个 `@tool`。
    - 用 `create_agent(model, tools=[retriever_tool])` 得到一个**RAG Agent**。([LangChain][8])

推荐路径：

1. `rag/loaders.py`

   - 用 LangChain 文档的 loaders & splitters。

2. `rag/index_builder.py`

   - 先用 `InMemoryVectorStore` 或 `FAISS`，再换成 Milvus/Pinecone 等。([LangChain Docs][9])

3. `rag/retriever.py`

   - `.as_retriever()`，再包装成 `@tool`。

4. `rag/agent.py`

   - 按「RAG Agent」文档里的套路写一个 agent（本质就是 create_agent + retriever tool）。([LangChain][8])

你已经在问 LangGraph Agentic RAG 文档了，可以顺势在 `rag/graph.py` 用 LangGraph 重写一版「多步骤 RAG」：
比如：**重写问题 → 多检索 → 压缩上下文 → 回答** 的 agentic RAG 图。([LangChain Docs][9])

---

### 第 3 阶段：LangGraph 自定义工作流（Stateful + Checkpointer + HITL）

**目标：** 用 LangGraph 搭一个真正「多步骤、可恢复」的学习任务工作流。
比如流程：

> 用户提问 → 规划学习路径 → 选文档做 RAG → 生成学习计划 → 生成小测验 → 等用户答题 → 评分 + 反馈。


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [leonyangdev/lc-studylab](https://github.com/leonyangdev/lc-studylab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
