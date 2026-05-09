---
trigger: always_on
description: 项目通过4个FunctionTool为LLM提供记忆能力：
---

# AstrBot Angel Memory Plugin - 项目上手简报

## 核心设计模式

### LLM工具注入机制
项目通过4个FunctionTool为LLM提供记忆能力：
- **core_memory_remember**（[`CoreMemoryRememberTool`](tools/core_memory_remember.py:18)）：主动记忆工具，永久保存重要信息
- **core_memory_recall**（[`CoreMemoryRecallTool`](tools/core_memory_recall.py:20)）：主动回忆工具，加权随机抽取核心记忆
- **note_recall**（[`NoteRecallTool`](tools/note_recall.py:19)）：笔记展开工具，查看完整笔记内容
- **transfer_to_researcher**（[`build_research_handoff_tool`](tools/research_subagent.py:11)）：上游 Handoff 研究子代理，启动独立研究 Agent

这些工具在插件初始化时通过[`context.add_llm_tools()`](main.py:139)注入，成为LLM的主动能力。

## 核心设计模式

### 三层认知架构
项目采用「灵魂-潜意识-主意识」三层认知架构：
- **灵魂层**（[`SoulState`](core/soul/soul_state.py:13)）：AI的精神状态管理器，通过4维能量槽控制行为参数
- **潜意识层**（[`DeepMind`](core/deepmind.py:33)）：后台自动处理记忆的检索、整理和巩固
- **主意识层**（LLM）：通过工具接口主动记忆和回忆

这种设计模拟人类认知过程，实现了「观察→回忆→反馈→睡眠→灵魂共鸣」的完整认知工作流。

### 依赖注入与组件工厂
使用[`ComponentFactory`](core/component_factory.py:26)统一管理核心组件创建，确保：
- 主线程创建所有组件实例，避免跨线程问题
- 通过[`PluginContext`](core/plugin_context.py:23)统一资源管理
- 组件间依赖关系清晰，便于测试和维护

### 统一记忆运行时
通过[`MemoryRuntime`](core/memory_runtime/protocol.py)协议统一记忆接口：
- **VectorMemoryRuntime**：向量实现，内部委托`CognitiveService`
- **SimpleMemoryRuntime**：SQL+tags实现，基于`MemorySqlManager`
- `ComponentFactory`根据`enable_simple_memory`开关在初始化时选择实现
- 上层（DeepMind/tools）仅依赖统一接口，不感知底层实现

### 异步初始化架构
采用「极速启动+后台预初始化」模式：
- [`InitializationManager`](core/initialization_manager.py:29)管理状态转换
- [`BackgroundInitializer`](core/background_initializer.py:20)异步执行耗时操作
- 系统毫秒级启动，不阻塞主线程

## 数据流与控制流

### LLM工具调用流程
```
LLM决策调用工具 → FunctionTool.run()
→ 从event.plugin_context获取memory_runtime组件
→ 调用统一记忆接口（MemoryRuntime）
→ 返回结果给LLM
```

### 记忆存储流程
```
用户输入 → DeepMind.organize_and_inject_memories()
→ memory_runtime.remember() → 底层实现（向量/SQL）
```

### 记忆检索流程
```
查询输入 → 预处理(实体提取) → memory_runtime.chained_recall()
→ 向量模式（VectorMemoryRuntime）：
  向量相似度检索 → 可选二阶段重排（rerank_provider）→ 实体优先召回 → 类型分组 → 结果融合
→ Simple模式（SimpleMemoryRuntime）：
  实体提取结果会并入查询词（query + entities）后执行 FTS5 检索（jieba 预分词）
  FTS5 候选按 rank 归一后与默认向量分融合（记忆阈值0.5，笔记阈值0.6）
  当前不执行二阶段语义重排；保留实体优先与类型分组阶段；最终返回合并结果
```

### 记忆反馈流程
```
LLM响应 → DeepMind.async_analyze_and_update_memory()
→ 小模型分析 → memory_runtime.feedback() → 记忆强化/新增/合并 → 睡眠巩固
```

### 灵魂状态流程
```
记忆检索 → SoulState.resonate() (旧记忆冲击当前状态)
→ 小模型分析 → SoulState.update_energy() (根据16态代码更新)
→ 动态参数注入 → LLM请求参数调整
→ 新记忆生成 → 注入灵魂快照
```

## 开发场景导航

### 新增LLM工具
1. 创建新的`FunctionTool`子类，继承自[`astrbot.api.FunctionTool`](tools/core_memory_remember.py:18)
2. 定义`name`、`description`、`parameters`（JSON Schema格式）
3. 实现`async def run()`方法，从`event.plugin_context`获取所需服务
4. 在[`main.py`](main.py:139)的`context.add_llm_tools()`中注册

### 新增记忆类型
1. 在[`MemoryType`](llm_memory/models/data_models.py:24)枚举中添加新类型
2. 在[`MemoryHandlerFactory`](llm_memory/service/memory_handlers.py:102)中注册处理器
3. 更新[`process_feedback()`](llm_memory/service/memory_manager.py:398)方法处理新类型

### 修改数据模型
- 记忆模型：[`BaseMemory`](llm_memory/models/data_models.py:61)
- 笔记模型：[`NoteData`](llm_memory/models/note_models.py)
- 配置模型：[`MemorySystemConfig`](llm_memory/config/system_config.py:16)

### 灵魂状态配置
灵魂系统4维能量槽配置：
- **RecallDepth**：回忆量倾向，控制RAG检索数量
- **ImpressionDepth**：记住量倾向，控制记忆生成数量
- **ExpressionDesire**：发言长度倾向，控制LLM输出长度
- **Creativity**：思维发散倾向，控制LLM温度参数

每个维度支持min/mid/max三级配置，通过橡皮筋算法(Tanh)映射到具体参数。

### 配置环境变量
关键配置项：
- `MEMORY_EMBEDDING_MODEL`：嵌入模型选择
- `MEMORY_COLLECTION_NAME`：记忆集合名称
- `MEMORY_STRENGTH_THRESHOLD`：记忆强度阈值
- `soul_*_min/mid/max`：灵魂参数范围配置

### 调试记忆系统
1. 检查插件状态：[`plugin.get_plugin_status()`](main.py:352)
2. 查看组件初始化：[`ComponentFactory.get_components()`](core/component_factory.py:276)
3. 监控记忆流程：DeepMind日志输出
4. 灵魂状态调试：[`soul.get_state_description()`](core/soul/soul_state.py:323)

## 核心技术栈

### 工具层
- **FunctionTool框架**：AstrBot官方工具协议，支持JSON Schema参数定义
- **PluginContext注入**：通过`event.plugin_context`访问所有核心服务
- **参数验证**：工具层执行严格的参数验证（长度、格式、范围）
- **错误处理**：友好的错误提示，指导LLM正确使用工具

### 存储层
- **ChromaDB**：向量数据库，存储记忆和笔记（向量模式）
- **SQLite**：标签管理、文件索引、SimpleMemory存储（`MemorySqlManager`）

### 检索层
- **向量模式**：语义检索（向量相似度 + 可选rerank_provider重排）+ 链式召回（实体优先+类型分组）
- **Simple模式**：FTS5（jieba 预分词）检索 + rank归一融合 + 阈值过滤

### 嵌入层
- **本地模型**：SentenceTransformers（BAAI/bge-small-zh-v1.5）
- **API提供商**：支持多种第三方嵌入服务
- **缓存机制**：内存缓存提升重复查询性能

### 灵魂系统
- **状态管理**：4维能量槽实时状态维护
- **橡皮筋算法**：Tanh函数实现非线性映射
- **共鸣机制**：旧记忆状态快照冲击当前状态
- **16态编码**：二进制状态码表示AI精神状态

## 关键决策点

### 1. 工具即能力：双层记忆架构
项目采用「被动记忆+主动记忆」双层架构：
- **被动记忆**：DeepMind自动管理，LLM无感知，通过事件钩子自动注入
- **主动记忆**：通过工具暴露，LLM主动调用，显式的记忆操作

工具设计原则：
- **最小化参数**：judgment/reasoning/tags三元组，50字符限制
- **加权随机**：recall工具按strength加权抽样，避免确定性偏见
- **上下文传递**：通过`event.plugin_context`统一获取服务实例
- **错误友好**：详细的错误提示，帮助LLM自我纠正

### 2. 统一三元组记忆结构
所有记忆类型采用`(judgment, reasoning, tags)`三元组：
- **judgment**：核心论断，用于向量化
- **reasoning**：解释背景，提供上下文
- **tags**：分类标签，支持实体识别

这种设计简化了记忆模型，同时保持了表达力。

### 3. 主动记忆与被动记忆
- **主动记忆**(`is_active=True`)：永不衰减，用户重要偏好
- **被动记忆**：会随时间和使用衰减，一般性知识

这种区分模拟人类记忆，确保重要信息长期保存。

### 4. 灵魂状态动态参数调整
通过4维能量槽实现AI行为参数动态调整：
- **能量累积**：历史刺激累积影响当前行为
- **自然衰减**：能量值随时间自动回归中庸
- **橡皮筋映射**：非线性映射保证参数稳定
- **状态共鸣**：旧记忆状态影响当前决策

### 5. 异步反馈队列
使用[`FeedbackQueue`](core/utils/feedback_queue.py)处理记忆反馈：
- 不阻塞主对话流程
- 批量处理提升性能
- 失败重试保证可靠性

### 6. 向量↔Simple双向同步
- **备份（向量→SQL）**：每次睡眠巩固后，自动将向量库记忆备份到`simple_memory.db`
- **回灌（SQL→向量）**：在睡眠维护管线中按模式/供应商变更条件触发，将SQL库中缺失的记忆回灌到向量库并重新向量化
- 实现供应商无关的记忆持久化，切换嵌入模型不丢数据

### 7. 文件监控与笔记系统
- 自动监控文档变化，实时更新索引

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kawayiYokami/astrbot_plugin_angel_memory](https://github.com/kawayiYokami/astrbot_plugin_angel_memory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
