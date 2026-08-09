---
trigger: always_on
description: 本文件适用于整个仓库。若子目录以后出现更具体的 `AGENTS.md`，则子目录规则优先。
---

# AGENTS.md

本文件适用于整个仓库。若子目录以后出现更具体的 `AGENTS.md`，则子目录规则优先。

这是一个经过多轮演进的老项目：同一能力可能同时存在“当前分层实现、历史单体实现、实验性实现”。工作的首要原则是先确认真实运行链路，再做最小、可验证、可回退的修改。不要仅凭文件名或 README 中的描述判断代码是否在线上路径中。

## 1. 项目目标与不可破坏的底线

“心语”是面向情感支持场景的 AI 对话应用，提供情绪与意图识别、长期记忆、RAG、Agent/Skills、附件与流式回复、反馈和自动评估。

它不是医疗诊断或心理治疗产品。任何改动都必须遵守以下底线：

- 不把模型输出包装成诊断、处方或专业治疗结论。
- 不弱化自伤、自杀、暴力、危机或高风险意图的识别、升级和兜底逻辑。
- 危机安全策略优先于“更自然”“更简短”“更像真人”等风格目标。
- 不把真实对话、用户画像、记忆、附件、模型密钥或数据库凭据放进提交、测试快照和日志。
- 记忆、会话、情绪记录和向量数据必须按 `user_id`、`session_id` 隔离。任何读取、更新、删除都要验证归属。
- 外部 URL、上传文件、Hermes 工作区和工具调用都是不可信输入，不能扩大访问范围或默认开启危险能力。

安全策略资料位于 `knowledge_base/organization_policy/crisis_intervention_protocol.md`。涉及危机处理时，先阅读该文件以及相关敏感内容实现文档。

## 2. 技术栈与实际版本边界

- 后端：Python 3.10/3.11 为主要开发目标，FastAPI、Uvicorn。
- 数据模型：Pydantic v1，项目明确限制 `<2.0.0`。
- ORM：SQLAlchemy 1.4，项目明确限制 `<2.0.0`。
- 数据库：MySQL 为主要部署方案；本地可使用 SQLite；Redis 为可选能力。
- 向量检索：ChromaDB 0.4 系列、LangChain 0.2 系列。
- 模型接入：OpenAI-compatible API，可切换智谱、通义、OpenAI 或其他兼容网关。
- 前端：React 18、Create React App 5、Axios、styled-components、react-markdown。
- 格式约定：Black 100 列，Ruff Python 3.10。

注意仓库存在版本漂移：

- `pyproject.toml` 声明 Python `>=3.10`，但现有 Dockerfile 和 GitHub Actions 仍使用 Python 3.9。
- 不要在普通功能修改中顺手统一版本。若使用 Python 3.10+ 语法，必须同时说明 Docker/CI 的兼容影响；若任务是升级运行时，则一起更新 Dockerfile、CI、文档和依赖锁定。
- `requirements.txt` 是现有安装和 Docker 路径使用的依赖来源；`pyproject.toml` 同时承担项目元数据与开发工具配置。新增或删除 Python 依赖时，两者都要核对。

## 3. 阅读顺序

开始修改前，按任务范围阅读：

1. 根目录 `README.md` 和与功能对应的 `docs/` 文档。
2. `backend/app.py`，确认路由是否真实注册。
3. 对应的 `backend/routers/*.py`，确认请求、响应和异常行为。
4. 路由实际导入的 service、model 和 database 实现。
5. `frontend/src/services/ChatAPI.js` 及调用该接口的 hook/component。
6. 对应测试和迁移。

不要跳过第 2 步。这个仓库中“有实现”不等于“当前应用已注册”。

## 4. 当前权威入口与历史入口

### 4.1 启动入口

| 文件 | 当前角色 | 修改规则 |
| --- | --- | --- |
| `main.py` | 本地开发组合入口，同时启动后端和 React 开发服务器 | 只放进程编排，不放业务逻辑 |
| `run_backend.py` | 当前独立后端入口，检查依赖并初始化 RAG 后启动 | 使用 `backend.app:app`；启动副作用要谨慎 |
| `backend/app.py` | 当前 FastAPI 应用工厂、CORS、静态目录和路由装配入口 | 新路由必须在这里或其导入的聚合模块中注册 |
| `backend/main.py` | 历史单体 FastAPI 应用，保留旧的多模态和部分旧接口 | 除非任务明确要求兼容旧入口，否则不要继续添加功能 |
| `scripts/simple_backend.py` | 旧 Python 环境兼容后端 | 不是正式实现 |
| `scripts/quick_start.py` | 旧版全量初始化入口 | 不应成为新功能依赖 |

正式后端导入目标是：

```text
backend.app:app
```

不要把 `backend.main:app` 当作当前应用，也不要因为旧单体中存在某个接口就认为前端能访问它。

### 4.2 当前应用装配

`backend/app.py` 在模块导入时执行 `app = create_app()`，因此导入应用可能同时导入并初始化聊天、RAG、意图、Agent 等服务。修改模块级初始化时要考虑：

- 测试收集阶段不能依赖真实 LLM、MySQL、Redis 或外网。
- 可选依赖失败应安全降级，不应让无关路由和 `/health` 无法导入。
- 不要在 import 阶段执行不可逆写操作、长时间网络调用或全量重建向量库。
- `run_backend.py` 已承担启动前知识库初始化；不要在多个入口重复做昂贵初始化。

当前还有一个 Windows 基线问题：`backend/database.py` 会在 import 阶段探测 MySQL；MySQL 不可用时本应回退 SQLite，但 GBK 控制台可能无法输出警告中的 Unicode 符号，进而抛出 `UnicodeEncodeError`。排查应用导入时先设置 `PYTHONUTF8=1`，并显式选择隔离的 SQLite。不要把这个编码异常误判成路由或模型故障，也不要通过删除本地数据库来规避。

## 5. 后端目录与职责

### 5.1 主要目录

| 路径 | 职责 | 注意事项 |
| --- | --- | --- |
| `backend/routers/` | HTTP、SSE、WebSocket 路由 | 参数解析、状态码、调用 service；避免堆积业务逻辑 |
| `backend/services/` | 聊天、上下文、记忆、个性化、反馈、性能等业务编排 | 当前主聊天链路在这里 |
| `backend/modules/` | RAG、LLM、Intent、多模态、模块化 Agent | 模块内部可有自己的 models/routers/services |
| `backend/agent/` | 较早期但仍被当前 `/agent` 路由使用的 Agent 核心与工具 | 与 `backend/modules/agent/` 重复，先看真实 import |
| `backend/runtime/` | 新一代 Runtime + Skills、会话、策略、预算、工具协议 | 不是所有代码都接入主聊天链路 |
| `backend/core/` | 通用配置、异常、接口、校验和工具 | 其中配置系统不是主业务唯一配置源 |
| `backend/hermes/` | 受限工作区自动化、意图与 dispatch | 默认关闭；涉及文件/网页/shell 权限 |
| `backend/plugins/` | 天气等聊天引擎插件 | 第三方 API 失败必须降级 |
| `backend/ab_testing/` | A/B 分组、事件与分析 | 当前 `backend/app.py` 未注册其 router |
| `backend/tests/` | 当前自动测试 | 测试量较少，改动功能时补回归测试 |

### 5.2 同名实现陷阱

- `backend/routers/agent.py` 与 `backend/modules/agent/routers/agent_router.py` 都声明 `/agent`。当前应用注册的是前者。
- `backend/models.py` 与 `backend/schemas/` 都有 Pydantic 模型。当前主聊天路由从 `backend.models` 导入 `ChatRequest`/`ChatResponse`，不是 `backend.schemas.chat_schemas`。
- 根 `config.py` 与 `backend/core/config.py` 是两套配置体系，变量命名不同。
- 根 `backend/main.py` 与 `backend/app.py` 都能创建 FastAPI app，但当前运行路径是后者。
- `backend/agent/` 与 `backend/modules/agent/` 并存，修改前沿当前 router/service 的 import 链追踪。
- `alembic/versions/` 与 `backend/migrations/*.sql` 并存；正式增量迁移优先 Alembic，历史 SQL 文件不能自动视为当前迁移链。

不要为了“消除重复”直接删除其中一套。先证明没有导入方、脚本、部署或文档依赖，再单独进行迁移型重构。

## 6. 当前路由地图

以下是 `backend/app.py` 当前装配的路由族：

| 前缀 | 实现 | 用途 | 装配方式 |
| --- | --- | --- | --- |
| `/chat` | `backend/routers/chat.py` | 普通聊天、附件、SSE、会话历史和删除 | 必选 |
| `/memory` | `backend/routers/memory.py` | 用户记忆查询、搜索、重要度和画像 | 必选 |
| `/feedback` | `backend/routers/feedback.py` | 用户反馈与统计 | 必选 |
| `/evaluation` | `backend/routers/evaluation.py` | 自动评估、批量评估、人工校验 | 必选 |
| `/api/emotion` | `backend/routers/emotion_analysis.py` | 情绪分析、趋势和报告 | 必选 |
| `/api/personalization` | `backend/routers/personalization.py` | 角色模板和用户个性化配置 | 必选 |
| `/api/rag` | `backend/modules/rag/routers/rag_router.py` | 知识库初始化、上传、查询和搜索 | 必选 |
| `/enhanced-chat` | `backend/routers/enhanced_chat.py` | 增强聊天和洞察 | import 成功时注册 |
| `/agent` | `backend/routers/agent.py` | Agent 聊天、状态、记忆、工具和回访 | import 成功时注册 |
| `/hermes` | `backend/routers/hermes.py` | 工作区自动化状态与 dispatch | import 成功时注册，实际能力受配置开关限制 |
| `/intent` | `backend/modules/intent/routers/intent_router.py` | 意图检测、Prompt 构建和批处理 | import 成功时注册 |
| `/performance` | `backend/routers/performance.py` | 指标、缓存和优化配置 | 与性能模块一起可选注册 |
| `/streaming` | `backend/routers/streaming_chat.py` | 另一套流式 HTTP/WebSocket API | 与性能模块一起可选注册 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [congde/emotional_chat](https://github.com/congde/emotional_chat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
