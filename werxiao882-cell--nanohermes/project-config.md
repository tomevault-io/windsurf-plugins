---
trigger: always_on
description: Python 自进化 AI Agent 系统，参考 Hermes Agent 架构。16 个核心模块，~1100 个测试。
---

# AGENTS.md - NanoHermes

## 项目概况

Python 自进化 AI Agent 系统，参考 Hermes Agent 架构。16 个核心模块，~1100 个测试。

## 关键命令

```bash
# 启动
python -m src.main              # TUI 交互模式（默认）
python -m src.main --debug      # debug 模式（输出完整请求/响应 JSON + 思考内容）
python -m src.main --resume     # 恢复最近会话
python -m src.main --resume <id>  # 按 ID 恢复
python -m src.main --resume-title "标题"  # 按标题恢复
python -m src.main --list-sessions  # 列出所有历史会话

# MCP 服务器启动
python -m src.mcp.server                    # Stdio 模式（默认）
python -m src.mcp.server --transport streamable-http --port 8000  # HTTP 模式
python -m src.mcp.server --transport sse --port 8000              # SSE 兼容模式

# 测试
python -m pytest tests/ -v              # 全部测试
python -m pytest tests/provider/ -v     # 单个模块
python -m pytest tests/tools/ -v
python -m pytest tests/loop/ -v         # 循环模块
python -m pytest tests/test_e2e.py -v -s  # 端到端（-s 显示输出）
```

## 环境与配置

- **Python >= 3.11**，依赖通过 `pyproject.toml` 管理
- **`.env` 文件**（gitignored）：`DASHSCOPE_API_KEY`、`DASHSCOPE_BASE_URL`、`MODEL_NAME`
- 支持多提供商：DashScope（默认）、OpenAI、Anthropic
- 代码中调用 `load_dotenv()` 自动加载 `.env`
- 安装依赖（国内用清华镜像）：`pip install -i https://pypi.tuna.tsinghua.edu.cn/simple openai anthropic pyyaml pydantic python-dotenv rich prompt_toolkit better-sqlite3`
- 开发依赖：`pip install pytest pytest-asyncio pytest-cov`

## 数据存储路径（不在项目目录内）

- **SQLite**: `~/.nanohermes/sessions.db`（会话元数据、FTS5 搜索）
- **JSONL**: `~/.nanohermes/sessions/<session_id>.jsonl`（完整消息历史）
- **Memory**: `~/.nanohermes/memory/`（MEMORY.md / USER.md）
- **MCP Config**: `~/.nanohermes/mcp_servers.json`（外部 MCP 服务配置）

## 架构边界

```
src/
├── main.py / __main__.py    # 入口（组合根，依赖注入 + 模块组装）
├── provider/                # LLM 提供商运行时（凭证/API路由/客户端/回退链）
├── tools/                   # 工具运行时（注册表/分发器/搜索引擎/终端/文件/技能等）
├── mcp/                     # MCP 协议支持（服务器/客户端/桥接/注册表）
├── session/                 # 会话存储（SQLite + JSONL 双存储）
├── memory/                  # 记忆系统（文件提供者 + 编排器）
├── skills/                  # 技能系统（SKILL.md 解析 + Curator 自进化）
├── compression/             # 上下文压缩（摘要预算 + 头尾保护）
├── prompt/                  # 系统提示组装（三层：stable/context/volatile）
├── conversation/            # 核心对话循环 + 事件总线 + 责任链拦截机制
├── delegation/              # 多 Agent 委托（leaf/orchestrator 角色）
├── insights/                # 指标引擎（token 聚合 + 成本估算）
├── cli/                     # TUI 聊天界面 + 事件处理器 + 流式组件
├── hooks/                   # 责任链拦截器（危险命令拦截、ScriptHook、配置加载）
├── loop/                    # 循环执行（间隔解析/维护提示/生命周期管理）
└── config/                  # 配置加载（模型定义/凭证解析/提供商注册）
```

## 工具系统

### 工具注册

- 工具通过 AST 自动发现：`discover_tools(tools_dir)` 扫描 `src/tools/` 下的模块
- 或显式初始化：`ToolRegistry.init_all_tools()` 导入所有工具模块
- 每个工具在模块加载时调用 `register_tool()` 自动注册

### 延迟加载（Tool Search）

工具分为两类：

**始终加载**（`defer_loading=False`，6 个）：启动时加入 LLM 上下文
- `read_file`, `write_file`, `search_files`, `patch`, `terminal`, `search_tools`

**延迟加载**（`defer_loading=True`，11 个）：通过 `search_tools` 工具按需发现
- `execute_code`, `process`, `todo`, `memory`, `session_search`, `clarify`, `skill_view`, `skills_list`, `skill_manage`, `delegate_task`, `cronjob`

搜索引擎使用 BM25（自然语言）+ Regex（精确匹配）双引擎，Auto 模式自动选择策略。

## 重要约定

- **每个 `src/<module>/` 必须包含 `ARCHITECTURE.md`**（详见 `openspec/specs/project-conventions/spec.md`）
- 代码注释使用**中文**，说明"为什么"而非仅"做什么"
- pytest 配置 `asyncio_mode = "auto"`，无需手动标记异步测试
- 事件驱动解耦：`ConversationLoop` 通过 `EventBus` 发布事件，外部处理器（记忆、TUI、调试）订阅接入
- 责任链拦截机制：`EventBus.intercept()` 注册拦截器，可修改数据或阻断流程，拦截器阻断后观察者仍触发

## 编码规范

### 核心原则：低耦合、高聚合、单一职责

所有代码必须遵循以下原则：

#### 1. 单一职责原则 (SRP)

- **每个类/函数只做一件事**：一个类只有一个改变的理由，一个函数只完成一个明确的任务
- **模块职责清晰**：每个模块有明确的边界，不越权处理其他模块的职责
- **入口文件瘦身**：`main.py` 只负责依赖注入和模块组装，不包含业务逻辑或直接操作 SDK
- **文件大小控制**：单个文件不超过 300 行，超过需拆分

#### 2. 低耦合

- **通过接口交互**：模块间通过抽象类/协议交互，不直接依赖具体实现
- **依赖注入**：对象通过构造函数或参数注入，不在内部创建其他模块实例
- **事件驱动解耦**：使用 `EventBus` 解耦核心循环与外部处理器（记忆、调试、TUI 等）
- **责任链拦截**：`EventBus.intercept()` 注册拦截器，`emit()` 返回 `ChainResult`，可检查 `blocked` 状态
- **禁止跨模块调用**：不直接调用其他模块的内部方法或访问私有属性

#### 3. 高聚合

- **相关功能放在一起**：将职责相关的代码组织在同一模块中
- **消除重复定义**：相同的数据类型、枚举、常量只定义一次，在共享模块中维护
- **避免数据冗余**：定价数据、模型元数据、配置类型等集中管理，不多处维护

#### 4. 代码简洁易读

- **函数简短**：单个函数不超过 50 行，复杂逻辑拆分为多个小函数
- **命名清晰**：变量/函数名自解释，减少注释依赖
- **减少嵌套**：使用提前返回 (early return) 减少 if/else 嵌套层级
- **避免魔法数字**：使用命名常量替代硬编码的数字和字符串

### 注释规范

#### 核心原则：解释"为什么"而非仅"做什么"

代码注释使用**中文**，重点解释设计决策、边界情况和潜在陷阱，而非简单描述代码功能。

#### 必须添加注释的场景

1. **复杂 Python 语法**：
   - 生成器（`yield` 语义、背压、调用栈连续性）
   - 装饰器（工作原理、适用场景、副作用）
   - 异步代码（`async/await` 使用理由、事件循环管理）
   - 闭包（词法作用域、状态捕获、生命周期）
   - 类型注解（泛型、联合类型、可调用类型的设计理由）

2. **设计模式实现**：
   - 工厂模式（为什么选择工厂而非直接实例化）
   - 单例模式（类变量 vs 实例变量、线程安全）
   - 观察者模式（事件订阅机制、解耦理由）
   - 策略模式（算法选择依据、扩展点）
   - 责任链模式（链式调用、回退策略）

3. **架构决策**：
   - 模块边界划分理由
   - 依赖注入 vs 内部创建
   - 同步 vs 异步选择
   - 缓存策略设计
   - 错误处理和恢复机制

4. **算法和公式**：
   - 数学公式的推导过程
   - 阈值选择的理论依据
   - 性能优化的权衡
   - 边界情况处理

5. **安全考量**：
   - 输入验证和防护策略
   - 权限控制机制
   - 数据隔离和隐私保护
   - 原子操作和并发安全

#### 注释格式要求

```python
# ✅ 正确：解释设计决策
# 设计理由：
# 使用 SHA256 哈希判断 stable 层内容是否变化。
# - SHA256 碰撞概率极低，适合缓存失效判断
# - 只取前 16 位（64 bit），足够唯一性且节省存储空间
# - 64 bit 的碰撞概率约为 1/2^64，对于实际应用足够安全

# ✅ 正确：解释边界情况
# 注意：当 stable 层为空时，返回空字符串而非 None
# 这样调用方无需检查返回值类型，简化使用

# ❌ 错误：仅描述功能
# 计算哈希值
hash = hashlib.sha256(content.encode()).hexdigest()
```

#### 模块级文档字符串

每个模块文件开头必须包含文档字符串，说明：
- 模块职责和边界
- 主要组件和数据流
- 关键设计决策
- 依赖关系

```python
"""系统提示组装模块。

三层架构：

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [werxiao882-cell/NanoHermes](https://github.com/werxiao882-cell/NanoHermes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
