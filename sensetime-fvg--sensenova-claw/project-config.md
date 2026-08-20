---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

Sensenova-Claw 是基于事件驱动架构的 AI Agent 平台，支持 Web、CLI、TUI 多种接入方式。

**技术栈**:
- 后端: FastAPI + Python 3.12 + asyncio + SQLite
- 前端: Next.js 14 + TypeScript + WebSocket
- 包管理: uv (Python), npm (Node.js)

## 常用命令

### 开发启动

```bash
# 一键启动后端 + 前端 dashboard（推荐）
sensenova-claw run
# 或: npm run dev  /  python3 -m sensenova_claw.app.main run

# 仅启动后端
sensenova-claw run --no-frontend
# 或: npm run dev:server

# 单独启动前端
npm run dev:web

# 启动 CLI 客户端（需后端已运行）
sensenova-claw cli
# 或: python3 -m sensenova_claw.app.main cli --port 8000

# 自定义端口
sensenova-claw run --port 9000 --frontend-port 3001
sensenova-claw cli --port 9000
```

### 测试

```bash
# 后端 e2e 测试（需要真实 API key）
npm run test:e2e
# 或: python3 -m pytest tests/e2e -q

# 前端 e2e 测试
npm run test:web:e2e

# 后端单元测试
python3 -m pytest tests/unit/ -q

# 全部测试
python3 -m pytest tests/ -q
```

### Python 环境

```bash
# 安装依赖
uv sync

# 安装开发依赖
uv sync --extra dev

# 运行 Python 脚本
python3 xxx.py
```

## 核心架构

### 事件驱动架构

所有模块通过 **PublicEventBus** 解耦通信，事件封装为 `EventEnvelope`：

```python
class EventEnvelope:
    event_id: str      # UUID
    type: str          # 事件类型，如 "ui.user_input", "llm.call_completed"
    session_id: str    # 会话隔离
    turn_id: str       # 对话轮次
    trace_id: str      # 关联请求/响应（如 llm_call_id, tool_call_id）
    payload: dict      # 事件数据
    source: str        # 来源: ui/agent/llm/tool/system
```

**事件流**:
```
user.input → agent.step_started → llm.call_requested → llm.call_completed
→ tool.call_requested → tool.call_completed → agent.step_completed
```

### Gateway 与 Channel

- **Gateway**: 管理多个 Channel，在 Channel 和 PublicEventBus 之间路由事件
- **Channel**: 用户接入抽象（WebSocketChannel、FeishuChannel）
- 每个 Channel 独立管理会话，通过 `session_id` 隔离

### 核心 Runtime 模块

所有 Runtime 订阅 PublicEventBus，通过 `session_id` 过滤事件：

- **AgentRuntime**: 对话流程编排，监听 `user.input`，发布 `agent.step_*`
- **LLMRuntime**: LLM 调用管理，监听 `llm.call_requested`，发布 `llm.call_completed`
- **ToolRuntime**: 工具执行，监听 `tool.call_requested`，发布 `tool.call_completed`
- **TitleRuntime**: 自动生成会话标题

### 配置管理

- **ConfigManager** (`platform/config/config_manager.py`): 统一配置写入入口，负责持久化 YAML、刷新内存、发布 `config.updated` 事件
- **ConfigFileWatcher**: 监听 config.yml 外部变更，通过 watchdog 实现
- 配置变更通过 EventBus 通知下游模块（LLMFactory、AgentRegistry、MemoryManager）自动刷新
- Gateway 将 `config.*` 事件广播给所有 WebSocket 客户端

### 状态管理

- **SessionStateStore**: 内存状态管理（Turn、Message、工具调用状态）
- **SQLite**: 持久化存储（sessions、turns、messages、events、agent_messages、cron_jobs、cron_runs 表）

### 工具系统

启动时注册核心工具（在 `_register_builtin()` 中），加上运行时条件注册的工具：

**核心工具（始终注册）：**
- `bash_command`: 执行 shell 命令
- `serper_search`: 网络搜索（需 SERPER_API_KEY）
- `image_search`: 图片搜索（需 SERPER_API_KEY）
- `brave_search`: 网络搜索（需 BRAVE_SEARCH_API_KEY）
- `baidu_search`: 网络搜索（需 BAIDU_APPBUILDER_API_KEY）
- `tavily_search`: 网络搜索（需 TAVILY_API_KEY）
- `fetch_url`: 获取网页内容
- `read_file`: 读取文件
- `write_file`: 写入文件
- `create_agent`: 动态创建新 Agent 配置
- `ask_user`: 向用户提问（需确认/输入时使用）

**条件注册的工具：**
- `send_message`: 向其他 Agent 发送消息（`delegation.enabled=true` 时注册）
- `cron_tool`: 定时任务管理（`cron.enabled=true` 时注册）
- `memory_search`: 记忆搜索（`memory.enabled=true` 且 `memory.search.enabled=true` 时注册）
- 邮件工具（6 个）: `send_email`、`list_emails`、`read_email`、`download_attachment`、`mark_email`、`search_emails`（`tools.email.enabled=true` 时注册）

工具注册在 `ToolRegistry`，搜索工具在未配置对应 API key 时不暴露给 LLM。

#### 邮件工具配置

邮件工具默认禁用，需在 `config.yml` 中配置：

```yaml
tools:
  email:
    enabled: true
    smtp_host: smtp.gmail.com
    smtp_port: 587
    imap_host: imap.gmail.com
    imap_port: 993
    username: ${EMAIL_USERNAME}
    password: ${EMAIL_PASSWORD}
    max_attachment_size_mb: 10
    timeout: 30
```

**常见邮箱配置：**

Gmail:
- SMTP: smtp.gmail.com:587
- IMAP: imap.gmail.com:993
- 需使用应用专用密码: https://myaccount.google.com/apppasswords

Outlook:
- SMTP: smtp-mail.outlook.com:587
- IMAP: outlook.office365.com:993

QQ 邮箱:
- SMTP: smtp.qq.com:587
- IMAP: imap.qq.com:993
- 需在设置中开启 IMAP/SMTP 服务并获取授权码

#### Obsidian 工具

Obsidian 工具支持本地和远程 vault。

**本地配置（可选，支持自动检测）：**
```yaml
tools:
  obsidian:
    vaults:
      - ~/Documents/MyVault
```

**远程配置（需安装 Local REST API 插件）：**
```yaml
tools:
  obsidian:
    remote:
      - name: work-vault
        url: http://192.168.1.100:27123
        api_key: your-api-key
```

**知识库 Skill 配置（可选）：**
```yaml
skills:
  knowledge-base:
    enabled: true
    root_folder: "Knowledge"
    auto_load_profile: true
    auto_save: true
    index_on_start: true      # 会话开始时生成知识库索引
    index_limit: 30           # 索引笔记数上限（按修改时间排序）
```

自动检测位置: `~/Documents/Obsidian`, `~/Obsidian`, macOS iCloud 等

远程设置: 安装 [Local REST API](https://github.com/coddingtonbear/obsidian-local-rest-api) 插件并配置 API key

#### 使用 obsidian_locate_and_setup 工具自动配置

如果未在 `config.yml` 中配置 Obsidian vault，可以使用 `obsidian_locate_and_setup` 工具自动定位或创建：

**system-admin 中的使用**：

```bash
# system-admin 会自动调用此工具并提示用户选择 vault
obsidian_locate_and_setup
```

**手动调用**（如果需要）：

```python
from sensenova_claw.capabilities.tools.obsidian_locate import ObsidianLocateTool

tool = ObsidianLocateTool()
result = await tool.execute()
# 返回格式：{
#   "success": bool,
#   "vaults": [{"name", "path", "source", "has_structure", "created_now", "accessible"}, ...],

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SenseTime-FVG/sensenova-claw](https://github.com/SenseTime-FVG/sensenova-claw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
