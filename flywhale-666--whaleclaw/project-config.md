---
trigger: always_on
description: - 运行时: Python 3.12 (项目内嵌独立 Python，路径 `./python/bin/python3.12`)
---

# WhaleClaw — 项目编码指南

- 项目名: WhaleClaw
- 仓库根目录: 当前目录
- 运行时: Python 3.12 (项目内嵌独立 Python，路径 `./python/bin/python3.12`)
- 包管理: pip + pyproject.toml
- Web 框架: FastAPI + uvicorn
- 异步运行时: asyncio

## 项目结构

```
whaleclaw/                    # Python 包根目录
  __init__.py
  entry.py                    # 主入口 (uvicorn 启动)
  version.py                  # 版本号
  config/                     # 配置管理 (Pydantic v2 模型)
    __init__.py
    schema.py                 # 配置 schema 定义
    loader.py                 # 配置加载/合并/校验
    paths.py                  # 路径常量 (~/.whaleclaw/)
  gateway/                    # Gateway WebSocket 控制面
    __init__.py
    app.py                    # FastAPI 应用工厂
    ws.py                     # WebSocket 协议处理
    protocol.py               # WS 消息协议定义
    middleware.py              # 中间件 (认证/CORS/日志)
  agent/                      # Agent 运行时
    __init__.py
    loop.py                   # Agent 主循环 (消息 -> LLM -> 工具 -> 回复)
    context.py                # Agent 上下文 (会话状态/工具注册)
    prompt.py                 # PromptAssembler 分层提示词组装器 (静态/动态/延迟三层)
  providers/                  # LLM 模型提供商适配层
    __init__.py
    base.py                   # Provider 抽象基类
    anthropic.py              # Anthropic Claude
    openai.py                 # OpenAI GPT
    deepseek.py               # DeepSeek
    qwen.py                   # 通义千问
    zhipu.py                  # 智谱 GLM (glm-4.7 / glm-5)
    minimax.py                # MiniMax (海螺 AI)
    moonshot.py               # 月之暗面 Kimi
    google.py                 # Google Gemini
    nvidia.py                 # NVIDIA NIM (免费模型)
    router.py                 # 模型路由 + failover
  channels/                   # 消息渠道
    __init__.py
    base.py                   # Channel 抽象基类/协议
    webchat/                  # WebChat 渠道
      __init__.py
      handler.py
    feishu/                   # 飞书渠道
      __init__.py
      bot.py                  # 飞书 Bot 事件处理
      client.py               # 飞书 API 客户端
      card.py                 # 卡片消息构建
      media.py                # 媒体上传下载
      webhook.py              # Webhook 安全验证
      tools/                  # 飞书工具集
        docx.py               # 文档操作
        wiki.py               # 知识库操作
        drive.py              # 云盘操作
        bitable.py            # 多维表格操作
        perm.py               # 权限管理
  routing/                    # 消息路由
    __init__.py
    router.py                 # 路由引擎
    rules.py                  # 路由规则
  sessions/                   # 会话管理
    __init__.py
    manager.py                # 会话生命周期管理
    store.py                  # 会话持久化 (SQLite)
    context_window.py         # 上下文窗口管理
  tools/                      # 内置工具
    __init__.py
    base.py                   # Tool 抽象基类
    registry.py               # 工具注册表
    bash.py                   # Bash 命令执行
    file_read.py              # 文件读取
    file_write.py             # 文件写入
    file_edit.py              # 文件编辑
    browser.py                # 浏览器控制 (Playwright)
  plugins/                    # 插件系统
    __init__.py
    sdk.py                    # 插件 SDK (WhaleclawPluginApi)
    loader.py                 # 插件发现/加载
    registry.py               # 插件注册表
    evomap/                   # EvoMap 内置插件 (GEP-A2A 协作进化市场)
      __init__.py
      plugin.py               # EvoMap 插件主入口
      client.py               # A2A 协议客户端 (hello/publish/fetch)
      identity.py             # 节点身份管理 (sender_id 持久化)
      publisher.py            # 资产打包发布 (Gene + Capsule + EvolutionEvent)
      fetcher.py              # 资产拉取 + 本地缓存
      bounty.py               # 赏金任务 (claim/complete)
      hasher.py               # SHA256 content-addressable ID 计算
      models.py               # Pydantic 数据模型 (Gene/Capsule/EvolutionEvent/Task)
      config.py               # EvoMap 配置 (hub_url/webhook/sync_interval)
  skills/                     # 技能系统
    __init__.py
    manager.py                # 技能发现/安装/管理
    parser.py                 # SKILL.md 解析 (含 YAML frontmatter)
    router.py                 # SkillRouter 按需路由 (关键词匹配，不调用 LLM)
    summary.py                # AGENTS.md 摘要生成器
  memory/                     # 记忆系统
    __init__.py
    base.py                   # Memory 抽象基类
    vector.py                 # 轻量记忆存储 (JSON 持久化 + 关键词检索)
    manager.py                # 记忆编排 (Recall/Capture/Organizer/全局风格)
    summary.py                # 自动摘要
  media/                      # 媒体处理
    __init__.py
    pipeline.py               # 媒体处理管道
    transcribe.py             # 音频转文字
    vision.py                 # 图片理解
  security/                   # 安全
    __init__.py
    pairing.py                # DM 配对机制
    sandbox.py                # Docker 沙箱
    auth.py                   # 认证 (Token/密码)
  cli/                        # CLI 命令
    __init__.py
    main.py                   # Typer 应用入口
    gateway_cmd.py            # gateway 子命令
    agent_cmd.py              # agent 子命令
    message_cmd.py            # message 子命令
    config_cmd.py             # config 子命令
    doctor_cmd.py             # doctor 诊断命令
  tui/                        # 终端 UI
    __init__.py
    progress.py               # 进度条/spinner (Rich)
    table.py                  # 表格输出
  web/                        # WebChat 前端静态资源
    static/
  utils/                      # 通用工具
    __init__.py
    log.py                    # structlog 日志配置

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [flywhale-666/whaleclaw](https://github.com/flywhale-666/whaleclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
