---
trigger: always_on
description: 本文件面向在本仓库内工作的 AI Agent 与开发者。目标是先理解项目边界，再用最小改动完成需求，并保留可验证、可回退的交付路径。
---

# LightAgent 项目协作指南

本文件面向在本仓库内工作的 AI Agent 与开发者。目标是先理解项目边界，再用最小改动完成需求，并保留可验证、可回退的交付路径。

## 项目概览

LightAgent 是一个以 Python 为主的多渠道 Agent Harness 项目，包含：

- 后端运行入口：`app.py`
- 配置中心：`config.py`、`config-template.json`
- 消息渠道层：`channel/`
- 模型、语音、翻译路由：`bridge/`、`models/`、`voice/`、`translate/`
- Agent 核心协议、工具、技能、记忆、知识库：`agent/`
- 插件系统：`plugins/`
- CLI：`cli/`
- 历史桌面端归档：`desktop/`（停止维护，不参与构建发布）
- 文档站内容：`docs/`
- 回归测试：`tests/`

项目核心数据流：

1. `app.py` 加载配置并启动 `ChannelManager`。
2. `channel/channel_factory.py` 根据 `channel_type` 创建 Web、IM 或终端渠道。
3. 渠道把消息包装为 `bridge.context.Context`。
4. `bridge/bridge.py` 根据配置选择聊天模型、语音、翻译或 Agent 模式。
5. Agent 模式通过 `bridge/agent_bridge.py` 进入 `agent/`，按工具、技能、记忆与知识库上下文执行任务。
6. 回复通过原渠道发送回用户。

## 主要目录职责

- `agent/protocol/`：Agent 执行协议、流式执行、动作与结果模型。
- `agent/tools/`：内置工具实现。新增工具时优先继承 `BaseTool`，并确认 `agent/tools/__init__.py` 与 `ToolManager` 加载路径。
- `agent/tools/mcp/`：MCP 客户端与动态工具注册。修改时注意并发加载、热更新和子进程生命周期。
- `agent/skills/`：技能加载、过滤、启停配置与 prompt 格式化。内置技能在根目录 `skills/`，用户技能通常在 workspace 的 `skills/`。
- `agent/memory/`、`agent/knowledge/`：长期记忆、向量/关键词索引、知识库服务。
- `bridge/`：模型、语音、翻译、Agent 模式的统一路由层。改动这里会影响所有渠道。
- `channel/`：不同平台渠道。公共逻辑在 `channel/channel.py`、`channel/chat_channel.py`；新增渠道需接入 `channel/channel_factory.py`。
- `channel/wechat_group/`：个人微信群通道实现。Python 层负责 LightAgent 渠道适配、配置读取、上下文包装和回复发送；`sidecar/` 下的 Node.js Wechaty 进程负责扫码登录、群列表、群消息事件和微信侧真实发送。
- `models/`：不同 LLM Provider 的 Bot 与 Session。新增 Provider 要同步 `common/const.py`、`models/bot_factory.py` 和相关配置/文档。
- `plugins/`：聊天命令插件与插件管理器。不要把 Agent 工具和插件混为一类。
- `voice/`、`translate/`：ASR/TTS 与翻译 Provider。
- `desktop/`：已停止维护的 Electron 历史源码归档，不再开发、修复、构建或发布；Python 后端 `app.py` 不属于桌面端，继续作为项目主入口维护。
- `docs/`：英文、中文。涉及用户可见能力变更时，优先补充对应文档。
- `tests/`：`unittest` 风格回归测试，很多测试通过 stub/mocking 避免真实网络和外部服务。

## 本地运行与验证

默认在 Windows PowerShell 中工作。不要使用 `&&` 串联命令。

访问 GitHub 时如果直连请求超时或不稳定，可以使用本地代理 `http://192.168.3.5:1082` 重试；仅在网络访问场景使用该代理，不要把代理地址写入项目运行配置或代码默认值。

整理或创建 GitHub issue 时，一律提交到 `yideng966/LightAgent` 项目；标题和正文描述应使用简体中文，避免默认写英文；提交时必须注明合适的 label，至少明确是 `bug`、功能需求、文档或其他类型；不要默认使用当前 remote、fork 或其他仓库；docs\images目录不要提交开发过程截图。

后端依赖：

```powershell
python -m pip install -r requirements.txt
python -m pip install -r requirements-optional.txt
python -m pip install -e .
```

启动后端：

```powershell
python app.py
```

开发任务的验证约束：

- 自动化回归继续使用 `python -m unittest ...` 等本地测试命令。
- 需要验证真实运行链路时，必须在当前本地仓库使用 `python app.py` 启动实际环境并完成验证。
- 不得把远端 Docker 部署环境作为本地开发测试环境；除非用户针对部署或运维另行明确授权，否则不得为验证代码而连接、更新、重启远端容器或操作远端数据。
- 下文 Docker 内容只作为构建与部署参考，不属于默认开发验证流程。

或安装 CLI 后：

```powershell
lightagent start
lightagent status
lightagent logs
```

运行全部 Python 测试：

```powershell
python -m unittest discover -s tests
```

运行单个测试文件：

```powershell
python -m unittest tests.test_models_handler
```

`desktop/` 已停止维护并仅作历史归档，不再作为本地开发或验证目标。源码运行继续使用 `python app.py`，图形管理入口使用随 Python 后端启动的 Web 控制台。

## Docker 构建与部署

> **连接实际部署环境**：如需登录已部署的服务器进行运维、日志调查或更新部署，请参考 [SERVER_ACCESS.md](./SERVER_ACCESS.md)。该文档包含 SSH 连接信息、Docker 容器管理命令及常见运维场景。

LightAgent 通过 `docker/Dockerfile.latest` 提供全功能 Docker 镜像，包含 Python 运行时、微信侧车 (Node.js Wechaty) 和可选的 Playwright/Chromium 浏览器引擎。简化的根 `Dockerfile` 基于预构建镜像 `ghcr.io/yideng966/lightagent:latest`，仅用于快速继承上游。

### 关键文件

| 文件 | 用途 |
|------|------|
| `Dockerfile`（根） | 基于上游预构建镜像的简化入口，仅设置 `ENTRYPOINT` |
| `docker/Dockerfile.latest` | 多阶段构建文件，从源码生成独立运行镜像 |
| `docker/docker-compose.yml` | 本地启动编排，映射端口 9899，挂载配置和数据卷 |
| `docker/entrypoint.sh` | 容器入口脚本，负责初始配置生成、密码管理和运行时目录准备 |
| `docker/build.latest.sh` | 构建脚本，从 `docker/` 目录运行 `cd .. && docker build -f docker/Dockerfile.latest ...` |
| `docker/.env.example` | 环境变量参考，可覆盖 Web 控制台密码 |
| `.dockerignore` | 排除 `.git`、`.worktrees/`、`node_modules`、`__pycache__`、日志、临时文件等 |

### 构建参数

`docker/Dockerfile.latest` 支持以下 `--build-arg`：

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `INSTALL_BROWSER` | `true` | 是否安装 Playwright/Chromium。设为 `false` 可大幅减小镜像体积并加速构建 |
| `USE_CN_MIRROR` | `false` | 是否使用清华 apt/pip/Playwright 镜像（国内构建更快） |
| `TZ` | `Asia/Shanghai` | 容器时区 |
| `CHATGPT_ON_WECHAT_VER` | 无 | LightAgent 版本标签 |

### 构建镜像

**完整构建（含 Chromium，镜像约 2–3 GB，耗时 10–30 分钟）：**

```bash
docker build -f docker/Dockerfile.latest -t yideng966/lightagent .
```

**快速构建（跳过浏览器，适用于仅需 API/CLI/Web 功能的场景，镜像约 800 MB–1.2 GB）：**

```bash
docker build -f docker/Dockerfile.latest \
  --build-arg INSTALL_BROWSER=false \
  -t yideng966/lightagent .
```

**使用中国镜像加速（可选）：**

```bash
docker build -f docker/Dockerfile.latest \
  --build-arg INSTALL_BROWSER=false \
  --build-arg USE_CN_MIRROR=true \
  -t yideng966/lightagent .
```

### Docker Compose 启动（推荐）

1. 确保已构建或拉取镜像 `yideng966/lightagent:latest`
2. 进入 `docker/` 目录并启动：

```bash
cd docker
docker compose up -d
```

3. 查看启动日志，获取自动生成的 Web 控制台密码（如未通过 `WEB_PASSWORD` 环境变量预设）：

```bash
docker compose logs lightagent | grep "Web console password"
```

4. 浏览器访问 `http://localhost:9899`，使用上述密码登录 Web 控制台。

首次启动时，`docker compose` 会自动在宿主机 `docker/` 下创建 `config/` 和 `data/` 目录，分别映射到容器内的 `/home/agent/.lightagent` 和 `/home/agent/lightagent`。

### Docker CLI 直接启动

```bash
docker run -d \
  --name lightagent \
  --security-opt seccomp:unconfined \
  -p 9899:9899 \
  -e WEB_HOST="0.0.0.0" \
  -v ./docker/config:/home/agent/.lightagent \
  -v ./docker/data:/home/agent/lightagent \
  yideng966/lightagent:latest
```

### 环境变量

| 变量 | 默认值 | 说明 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yideng966/LightAgent](https://github.com/yideng966/LightAgent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
