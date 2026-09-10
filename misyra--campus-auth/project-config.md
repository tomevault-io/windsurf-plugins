---
trigger: always_on
description: Campus-Auth 是一个基于 Playwright、FastAPI 和 Vue 3 的校园网自动认证工具。通过 Web 控制台管理配置和任务，定时探测网络状态，断网时自动触发浏览器认证流程重连。支持多网络配置方案自动切换、系统托盘、开机自启动和实时日志推送。
---

# Campus-Auth 校园网自动认证工具

## 项目概述

Campus-Auth 是一个基于 Playwright、FastAPI 和 Vue 3 的校园网自动认证工具。通过 Web 控制台管理配置和任务，定时探测网络状态，断网时自动触发浏览器认证流程重连。支持多网络配置方案自动切换、系统托盘、开机自启动和实时日志推送。

当前版本：v4.2.3

## 技术栈

| 层级 | 技术 |
|------|------|
| 运行时 | Python 3.12（严格版本约束，不支持 3.13+） |
| Web 框架 | FastAPI + Uvicorn |
| 浏览器自动化 | Playwright（Chromium） |
| 数据校验 | Pydantic v2 |
| 前端 | Vue 3 SPA（单文件，无构建步骤，原生 ES Module） |
| 包管理 | uv（镜像源：清华 PyPI + npmmirror Python） |
| 代码检查 | Ruff（lint + format） |
| 测试 | pytest + pytest-asyncio |
| 日志 | loguru |
| 密码加密 | cryptography（Fernet） |

## 开发命令

```bash
# 安装依赖
uv sync

# 安装 pre-commit hook
uvx pre-commit install

# 安装 Playwright 浏览器
uv run playwright install chromium

# 启动服务
python main.py

# 运行全部测试
uv run pytest

# 运行指定测试文件
uv run pytest tests/test_task_executor.py -v

# 代码检查（lint）
uv run ruff check .

# 代码检查并自动修复
uv run ruff check --fix .

# 代码格式化
uv run ruff format .

# 完整检查流程（pre-commit 自动执行）
uvx pre-commit run --all-files
```

## 代码规范

### 命名约定

| 类型 | 风格 | 示例 |
|------|------|------|
| 变量/函数 | `snake_case` | `get_profile`, `check_interval` |
| 类名 | `PascalCase` | `RuntimeConfig`, `ScheduleEngine` |
| 常量 | `UPPER_SNAKE_CASE` | `DEFAULT_TIMEOUT`, `PROJECT_ROOT` |

### Ruff 规则集

启用规则：`E`（pycodestyle 错误）、`F`（pyflakes）、`B`（bugbear）、`SIM`（简化）、`UP`（pyupgrade）、`PT`（pytest 风格）、`RUF`（ruff 特有）、`I`（isort）

已忽略规则及原因：
- `RUF001/RUF002/RUF003`：中文项目全角字符正常
- `B008`：FastAPI `Depends` 在默认参数中是标准做法
- `E501`：行长度不强制（改动量大）
- `PT019`：pytest fixture 参数注入风格

### 注释与文档

- 所有注释、docstring、文档均使用**中文**
- 每个 `.py` 文件必须有模块级 docstring（一行摘要）
- 公共 API（类和函数）必须有 docstring
- 行内注释解释"为什么"而非"是什么"，写在代码上方
- 标记约定：`TODO:`、`FIXME:`、`HACK:`（全大写 + 冒号）

### 字符串风格

- 普通字符串：双引号（Ruff format 默认）
- Docstring：三双引号 `"""..."""`

## 项目结构

```
Campus-Auth/
├── main.py                    # 统一启动入口（CLI + 启动编排）
├── pyproject.toml             # 项目元数据、依赖、工具配置
├── app/
│   ├── application.py         # FastAPI 主应用（工厂模式 create_app()）
│   ├── container.py           # ServiceContainer 依赖注入容器
│   ├── deps.py                # FastAPI Annotated 类型别名依赖注入
│   ├── schemas.py             # Pydantic 数据模型
│   ├── constants.py           # 共享常量（路径、超时、容量）
│   ├── version.py             # 版本读取（从 pyproject.toml）
│   ├── api/                   # API 路由层（16+ 个模块）
│   │   ├── monitor.py         # 监控控制
│   │   ├── config.py          # 配置管理
│   │   ├── tasks.py           # 任务管理
│   │   ├── profiles.py        # 配置方案
│   │   ├── debug.py           # 调试会话
│   │   ├── ws.py              # WebSocket 处理
│   │   └── ...                # 其他路由
│   ├── services/              # 业务服务层
│   │   ├── engine.py          # ScheduleEngine 统一后台引擎
│   │   ├── task_executor.py   # 任务执行器（双线程池）
│   │   ├── login_orchestrator.py # 登录编排
│   │   ├── login_attempt.py   # 登录尝试处理
│   │   ├── login_runner.py    # 登录执行（login_once 模式）
│   │   ├── scheduler_service.py  # 定时调度
│   │   ├── monitor_service.py    # 网络监控核心
│   │   ├── profile_service.py    # 配置方案管理
│   │   ├── websocket_manager.py  # WebSocket 管理
│   │   ├── config_builder.py  # 配置构建
│   │   ├── debug_service.py   # 调试会话管理
│   │   ├── debug_session.py   # 调试会话状态
│   │   ├── retry_policy.py    # 重试策略
│   │   ├── autostart.py       # 自启动服务
│   │   ├── login_history_service.py # 登录历史
│   │   ├── task_registry.py   # 定时任务注册表
│   │   ├── launcher.py        # 启动器
│   │   └── uninstall.py       # 卸载功能
│   ├── network/               # 网络检测（独立模块）
│   │   ├── probes.py          # TCP/HTTP/URL 探测
│   │   ├── decision.py        # 网络决策层
│   │   └── detect.py          # 网关/SSID 检测
│   ├── tasks/                 # 任务模型
│   │   ├── __init__.py        # TaskManager
│   │   └── models.py          # TaskConfig, StepConfig 等
│   ├── workers/               # Playwright 工作线程
│   │   ├── playwright_worker.py    # Actor 模型工作线程
│   │   ├── playwright_bootstrap.py # 环境准备
│   │   └── script_runner.py       # 脚本执行器
│   ├── system_tray.py         # 系统托盘
│   └── utils/                 # 工具模块
│       ├── logging.py         # 日志系统（loguru 封装）
│       ├── crypto.py          # 密码加密（Fernet）
│       ├── browser.py         # 浏览器上下文管理
│       └── ...                # 其他工具（cancel_token, concurrent, files 等）
├── frontend/                  # Vue 3 SPA（无构建步骤）
├── config/                    # 运行时配置
│   ├── settings.json          # 主配置文件
│   └── profiles/              # 配置方案文件
├── tasks/                     # 任务定义（JSON 驱动）
│   ├── browser/               # 浏览器任务
│   ├── scripts/               # 脚本任务
│   └── scheduled/             # 定时任务
├── tests/                     # pytest 测试
├── docs/                      # 文档
├── resources/                 # 资源文件
│   ├── icons/                 # 图标、背景
│   └── tools/                 # 辅助工具源码（git-puller、start、task-recorder）
├── debug/                     # 日志与截图（按日期归档）
└── temp/                      # 临时文件（启动时自动清理）
```

## 架构要点

### 分层架构

```
API 层（app/api/）
  ↓ 通过 FastAPI Depends 获取服务
Services 层（app/services/）
  ↓ 调用
Workers 层（app/workers/）
```

- **API 层**：纯路由定义，无业务逻辑，通过 `deps.py` 中的 Annotated 类型别名注入服务
- **Services 层**：核心业务逻辑，由 ServiceContainer 统一管理生命周期
- **Workers 层**：Playwright Actor 模型工作线程，所有浏览器操作统一收归

### ServiceContainer 依赖注入

`app/container.py` 中的 `ServiceContainer` 是整个后端的 DI 容器：

1. 接收 `project_root` 和 `mode` 参数

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Misyra/Campus-Auth](https://github.com/Misyra/Campus-Auth) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
