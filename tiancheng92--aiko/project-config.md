---
trigger: always_on
description: AI 编码助手指引，供 Claude Code 在此项目中使用。
---

# Desktop Pet — CLAUDE.md

AI 编码助手指引，供 Claude Code 在此项目中使用。

## 项目概览

**Aiko** 是一个 macOS 原生 AI 桌面宠物应用，采用 Wails v2 架构。Go 后端通过 `app.go` 暴露绑定方法给 Vue 3 前端；前端资源 embed 进 Go 二进制。核心特色是点击穿透（通过 `macos.go` 的 Objective-C cgo 实现）和基于 eino ReAct Agent 的智能对话系统。

### 核心依赖

**后端技术栈：**
- [Wails v2](https://wails.io/) - 跨平台桌面应用框架
- [eino](https://github.com/cloudwego/eino) - 字节跳动 Agent Development Kit  
- [chromem-go](https://github.com/philippgille/chromem-go) - 纯 Go 向量数据库
- [robfig/cron/v3](https://pkg.go.dev/github.com/robfig/cron/v3) - Cron 任务调度器
- [modernc.org/sqlite](https://pkg.go.dev/modernc.org/sqlite) - 纯 Go SQLite 驱动

**前端技术栈：**
- [Vue 3](https://vuejs.org/) + Composition API - 响应式前端框架
- [Vite](https://vitejs.dev/) - 现代构建工具
- [marked](https://marked.js.org/) - Markdown 解析渲染 
- [highlight.js](https://highlightjs.org/) - 代码语法高亮
- [KaTeX](https://katex.org/) - 数学公式渲染

**AI 生态：**
- OpenAI API 兼容接口 (OpenRouter, DeepSeek, 通义千问等)
- MCP (Model Context Protocol) - 工具协议标准
- lark-cli - 飞书命令行工具集成

## 架构设计

```
main.go → app.go (Wails bindings)
              ↓
    internal/agent/agent.go        # eino ReAct Agent 核心
    internal/agent/middleware/     # 日志/重试/错误恢复中间件
    internal/tools/                # 内置工具 + 权限管理
    internal/skill/                # YAML 自定义技能
    internal/memory/               # 短期(SQLite) + 长期(chromem-go) 记忆
    internal/knowledge/            # RAG 知识库
    internal/llm/                  # ChatModel / Embedder 抽象层
    internal/config/               # 配置持久化(SQLite)
    internal/scheduler/            # Cron 定时任务
    internal/mcp/                  # MCP 协议实现
    internal/sms/                  # 短信监听（fsnotify + 验证码识别）
    internal/db/                   # Schema 迁移管理
```

**前端架构：**
```
frontend/src/
├── components/        # Vue 组件 (ChatPanel, ChatBubble, SettingsWindow, etc.)
├── composables/       # 可复用逻辑 (useModelPath.js)
└── wailsjs/          # Wails 自动生成的 Go 绑定
```

## 开发规范

### Go 后端规范

- 所有导出函数必须有 `// FuncName ...` doc comment
- 错误处理用 `fmt.Errorf("context: %w", err)` 包装上下文
- 涉及 `a.cfg` / `a.petAgent` / `a.longMem` / `a.knowledgeSt` / `a.ttsSpeaker` / `a.mcpClosers` 的字段读写必须持有 `a.mu`（`RLock` 读，`Lock` 写）；`GetConfig` 返回 `*a.cfg` 的值拷贝，不返回原指针
- `sched.Start(a.ctx)` 与 `engine.Start(a.ctx)` 必须在 `a.mu.Unlock()` 之后调用——cron 回调会 `EventsEmit` 触发 Wails cgo，持锁时 emit 可能死锁
- 新增 Wails 绑定方法写在 `app.go`，签名遵循已有模式
- 新增内置工具：实现 `internaltools.Tool` 接口，在 `registry.go` 的 `All()` 中注册；运行时依赖工具加到 `AllContextual`；无需再在 `app.go startup` 里硬编码权限行——统一走 `internaltools.AllPermissionDeclarations()`
- `execute_shell` / `execute_code` 的 `working_dir` 参数必须经 `checkPath(workingDir, t.Cfg.AllowedPaths)` 白名单校验，并在 `t.Cfg == nil` 时直接返回错误提示（避免 nil deref）
- **Wails 绑定结构体中禁止使用 `time.Time`**，改用 RFC3339 字符串（`string`）——Wails TS 绑定生成器不识别 `time.Time`

### Vue 前端规范

- 全部使用 `<script setup>` 语法
- 包管理用 `yarn`，不用 npm
- 调用后端方法从 `../../wailsjs/go/main/App` import
- 监听 Wails 事件用 `EventsOn`，emit 用 `EventsEmit`（from `../../wailsjs/runtime/runtime`）
- **`EventsOn(event, handler)` 返回的 off 函数必须存到组件作用域变量**，在 `onUnmounted` 中统一 `off?.()` 解绑；禁止用 `EventsOff('event-name')` 传字符串（它会移除同事件所有监听器，影响其他组件）
- 拖拽类交互（mousedown → mousemove/mouseup）必须同时监听 `window.blur` 作为兜底解绑路径，防止鼠标拖出窗口后监听器悬空
- composable 里的 `setTimeout` / `setInterval` 必须通过 `onScopeDispose` 清理，避免卸载后回调仍作用于已销毁组件
- 组件内不直接操作全局状态，通过 composables 共享逻辑

### 核心 Wails 事件

| 事件名 | 方向 | 含义 |
|---|---|---|
| `chat:token` | backend→frontend | 流式 token 传输 |
| `chat:done` | backend→frontend | AI 响应结束 |
| `chat:error` | backend→frontend | 错误信息传递 |
| `chat:clear` | frontend→frontend | 清空聊天历史 |
| `bubble:toggle` | any | 切换聊天气泡显示/隐藏 |
| `pet:state:change` | any | 宠物状态变更 (idle/thinking/speaking/error) |
| `knowledge:progress` | backend→frontend | 知识库导入进度更新 |
| `config:model:changed` | frontend→frontend | Live2D 模型切换通知 |
| `config:chat:size:changed` | frontend→frontend | 聊天框尺寸变更 |
| `notification:show` | backend→frontend | 显示通知气泡 |
| `settings:open` | any | 打开设置界面 |
| `voice:start` | backend→frontend | 开始录音（Option 长按触发）|
| `voice:transcript` | backend→frontend | 实时 partial STT 结果 |
| `voice:final` | backend→frontend | isFinal STT 结果（可触发自动发送）|
| `voice:end` | backend→frontend | 录音结束（Option 释放时立即触发）|
| `voice:error` | backend→frontend | 语音识别错误 |
| `sms:verification_code` | backend→frontend | 检测到验证码短信 |
| `config:voice:auto-send:changed` | frontend→frontend | 语音自动发送开关状态变更 |
| `tool:confirm` | backend→frontend | 工具执行需用户确认（shell/code） |
| `tool:executing` | backend→frontend | 工具开始执行（显示进度条） |
| `tool:executed` | backend→frontend | 工具执行结束（隐藏进度条） |

## 开发命令

```bash
make run               # 构建 + ad-hoc 签名 + 启动（推荐，权限持久化）
make build             # 仅构建 + 签名
wails dev              # 开发模式（前端热重载，权限每次重置）
go build ./...         # 仅检查 Go 编译
cd frontend && yarn build   # 仅构建前端资源
wails generate module  # 重新生成 Wails bindings
```

> **权限说明**：使用 `make run` 构建会自动 ad-hoc 签名（`codesign --sign -`），配合 `wails.json` 中固定的 `bundleidentifier: com.xutiancheng.aiko`，macOS TCC 权限授权后跨重新编译持久有效。`wails dev` 不签名，每次重启可能重新弹权限窗。

## 数据目录结构

`~/.aiko/`
- `pet.db` — SQLite 数据库（settings、messages、knowledge_sources、cron_jobs、model_profiles、tool_permissions）
- `vectors/` — chromem-go 持久化向量数据存储
- `USER.md` — 用户画像文档（由 `update_user_profile` 工具自动维护）
- `auto-skills/` — Agent 自动沉淀的可复用技能（YAML 格式，由 `save_skill` 工具写入）

## 重要注意事项

### macOS 平台特定
- `macos.go` 中的 Objective-C 代码负责按像素判断鼠标事件响应，实现点击穿透功能

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tiancheng92/Aiko](https://github.com/tiancheng92/Aiko) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
