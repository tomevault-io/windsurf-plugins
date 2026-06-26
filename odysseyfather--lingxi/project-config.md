---
trigger: always_on
description: 本文件为 AI 助手（Codex / Cursor / Copilot 等）提供项目上下文，帮助快速理解系统全貌并高效开发。
---

# AGENTS.md — 灵犀 AI Agent 项目指南

本文件为 AI 助手（Codex / Cursor / Copilot 等）提供项目上下文，帮助快速理解系统全貌并高效开发。

---

## 项目简介

**灵犀 AI Agent** 是一个本地优先的桌面 AI Agent 工作台，采用 Electron + React + Go 三层架构。支持多模型接入、智能体工厂、技能管理、知识库、MCP 工具、IM 集成等能力。

---

## 技术栈

### 前端 `frontend-desktop/`
- **React 19** + **Vite 8**（构建需 Node.js ≥ 20.19 或 ≥ 22.12）
- **Tailwind CSS 3.4** — 全局样式，6 套主题通过 CSS 变量切换
- **Zustand 5** — 全局状态管理（`src/state/useStore.js`，模块化切片：auth/ui/session/chat/nexus）
- **Framer Motion 12** — 页面过渡、列表动画
- **Lucide React** — 图标（不使用 emoji）
- **prism-react-renderer 2** — 代码高亮
- **@tanstack/react-virtual 3** — 虚拟滚动
- **react-markdown + remark-gfm** — Markdown 渲染
- **Recharts 3** — 用量图表

### 后端 `backend-desktop/`
- **Go 1.24** + **Gin 1.10**
- **Gorilla WebSocket** — 流式对话
- **ncruces/go-sqlite3** — 纯 Go SQLite（无 CGO 依赖）
- **ledongthuc/pdf** — PDF 文本提取
- **nguyenthenguyen/docx** — DOCX 文本提取

### 桌面壳 `electron/`
- **Electron 36** + **electron-builder 25**
- 打包目标: macOS arm64

### 手机端 `mobile-flutter/`
- **Flutter 3.24+** + **Dart 3.5+**
- **Provider 6** — 状态管理
- **mobile_scanner 5** — QR 扫码配对
- **flutter_markdown** — Markdown 渲染
- **web_socket_channel 3** — WebSocket（流式对话 + 实时事件）
- **shared_preferences 2** — 本地持久化（pair_token / 连接信息）
- **image_picker 1** + **flutter_image_compress 2** — 图片附件
- 瘦客户端架构：所有 AI 计算和数据存储在 PC 端，手机端通过 LAN 直连或 WAN 隧道代理连接

---

## 项目结构

```
lingxi-agent/
├── backend-desktop/          # Go 后端
│   ├── main.go               # 入口 + 路由注册
│   ├── config/               # 配置管理
│   ├── logger/               # 结构化日志（slog JSON + LOG_LEVEL 环境变量）
│   ├── db/                   # SQLite 数据层（模块化拆分）
│   │   ├── db.go             # 初始化 + 迁移（schema_version 版本化）
│   │   ├── session.go        # 会话/任务/挂起任务 CRUD
│   │   ├── knowledge.go      # 知识库/分类 CRUD
│   │   ├── provider.go       # Provider/APIProfile CRUD
│   │   ├── usage.go          # 用量记录/配额 CRUD
│   │   ├── scheduled.go      # 定时任务 CRUD
│   │   ├── auth.go           # 用户/OAuth 配置 CRUD
│   │   ├── im_connector.go   # IM 连接器 CRUD
│   │   ├── evolution.go      # 自我进化日志 CRUD + InsertMemory
│   │   ├── nexus.go          # Nexus 表 CRUD（peers/contacts/a2a）
│   │   ├── group_chat.go     # 群聊 CRUD（group_chats/group_members/group_messages，含微信风扩展列）
│   │   ├── agent_personality.go  # 群聊 Agent 人格（agent_personalities 表）
│   │   ├── mcp_agent.go      # MCP-Agent 关联
│   │   └── screen_agent.go   # Screen Agent screen_actions 表 CRUD
│   ├── handler/              # HTTP Handlers
│   │   ├── agent_distill.go  # 人格蒸馏（dot-skill SSE + apply）
│   │   ├── agent_avatar.go   # 智能体头像上传
│   │   ├── dot_skill.go      # dot-skill 安装与路径
│   │   ├── agent.go          # 智能体 CRUD（含 API 缓存）
│   │   ├── cache.go          # TTL 缓存（sync.RWMutex，30s）
│   │   ├── chat.go           # 对话 + WebSocket 流式
│   │   ├── knowledge.go      # 知识库（支持 .md/.txt/.csv/.json/.pdf/.docx）
│   │   ├── session.go        # 会话管理 + 消息搜索
│   │   ├── provider.go       # 模型接入点（含 API 缓存）
│   │   ├── skill.go          # 技能管理（含 API 缓存 + 增强导出）
│   │   ├── mcp.go            # MCP 服务管理
│   │   ├── usage.go          # 用量统计
│   │   ├── im_connector.go   # IM 连接器
│   │   ├── scheduled.go      # 定时任务 CRUD
│   │   ├── auth.go           # SSO 登录（OAuth code 换 token + 游客登录）
│   │   ├── nexus.go          # Nexus 对外 API + 设置 + WAN API
│   │   ├── a2a_conversation.go # A2A 对话管理（发起/接受/拒绝/暂停/接管/终止/审批）
│   │   ├── agent_nexus_config.go # Agent 对外设置 CRUD
│   │   ├── evolution.go      # 自我进化引擎 + API（分析/提取/日志）
│   │   ├── screen_agent.go   # Screen Agent API（截屏分析/规划/OTA循环/安全确认）
│   │   ├── backup.go         # 数据库备份（VACUUM INTO + 导出）
│   │   ├── health.go         # 结构化健康检查
│   │   ├── middleware.go     # CORS + Body Size + Rate Limiter
│   │   ├── memory.go         # 长期记忆 CRUD + 消息固定
│   │   ├── transcribe.go     # 语音识别（本地 whisper.cpp 优先，回退远端 API）
│   │   ├── group_chat.go     # 群聊 HTTP API（创建/列表/发言/撤回/分页消息/邀请处理）
│   │   ├── group_upload.go   # 群聊图片上传（POST /api/group-chats/upload）
│   │   ├── agent_personality.go # Agent 群聊人格 CRUD（GET/PUT /api/agents/:id/personality）
│   │   ├── pair_auth.go      # 手机配对认证中间件 + WS 一次性票据 + 配对 API
│   │   ├── push_notify.go    # 推送通知（FCM v1 + /api/push/config + /api/push/test）
│   │   ├── proactive.go      # 主动式 Agent（日报 + 未完成任务追踪 + 定时调度）
│   │   ├── deep_search.go    # 深度联网搜索（DuckDuckGo + Wikipedia 多源 + LLM 综合 + SSE）
│   │   ├── knowledge_web.go  # 网页知识采集（go-readability + 自动入库索引）
│   │   ├── token_stats.go    # Token 水位 + 自动摘要压缩（/api/sessions/:id/token-stats + summarize）
│   │   ├── terminal.go       # PTY 终端 WebSocket handler（creack/pty）
│   │   ├── permission.go      # 权限分级（low/medium/high）配置 + 安全 hooks
│   │   ├── pty_unix.go       # Unix PTY 实现
│   │   ├── pty_windows.go    # Windows PTY 实现
│   │   ├── quick_chat.go     # Spotlight 快捷对话
│   │   ├── tasks.go          # 后台任务列表 + 删除
│   │   ├── router_status.go  # 路由器状态查询
│   │   └── ws_hub.go         # WebSocket Hub
│   ├── connector/            # IM 平台对接（企微/钉钉/飞书，飞书支持流式卡片消息）
│   ├── model/                # 数据模型
│   ├── nexus/                # Agent 间对话引擎（无 Token 认证，无联系人机制）
│   │   ├── discovery.go      # mDNS 发现服务 + 广域网信令客户端启动
│   │   ├── conversation.go   # 对话执行引擎（第一人称自然对话提示词）
│   │   ├── http_client.go    # HTTP 通信工具（PostJSON + 重试）
│   │   ├── transport.go      # Transport 接口（Send 无 token 参数）
│   │   ├── lan_transport.go  # 局域网 HTTP 直连传输

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OdysseyFather/lingxi](https://github.com/OdysseyFather/lingxi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
