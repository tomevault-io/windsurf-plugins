---
trigger: always_on
description: > 本文件是给 AI 编码助手（pi / Claude Code / Cursor 等）看的项目说明书：
---

# AGENTS.md — pi-web-ui 项目指南

> 本文件是给 AI 编码助手（pi / Claude Code / Cursor 等）看的项目说明书：
> 结构、架构约定、开发流程、GitHub 上传与 npm 发布流程。
> 修改本文件后，在 pi 中运行 `/reload` 生效。

## 1. 项目是什么

pi-web-ui 是 pi 编码智能体（`@earendil-works/pi-coding-agent` SDK）的 Web 聊天界面：
浏览器里对话、查看文件树、附加文件、内置终端（xterm.js + node-pty）、模型管理、
声音提醒、中英文切换。一条命令可跑（`pi-web-ui`），可 Docker / systemd / launchd /
Windows 计划任务部署。

- 仓库（公开）：`git@github.com:xing-shuyin/pi-web-ui.git`
- npm 包：`pi-web-ui`（发布者 npm 账号 `xingshuyin`）
- Node 要求：**>= 22.19.0**（pi SDK 的 dist 使用了 `import … with { type: "json" }` 语法）
- 版本：`package.json` 与 `package-lock.json` 两处同步维护

## 2. 技术栈

| 层 | 技术 |
| --- | --- |
| 后端 | Node + Express（静态 + `/api/health`）+ `ws`（`/ws` WebSocket 协议） |
| 前端 | React 18 + Vite 6 + react-markdown + highlight.js + xterm.js |
| 智能体 | `@earendil-works/pi-coding-agent` SDK（进程内，读 `~/.pi/agent` 配置） |
| 终端 | node-pty（服务端 PTY）+ `@xterm/xterm`（浏览器渲染，经 terminal bridge 转发） |
| 样式 | 单文件 `web/src/styles.css`（CSS 变量主题，深色） |

## 3. 目录结构

```
pi-web-ui/
├── server/                     # 后端（Node ESM，编译到 dist/server/）
│   ├── index.ts                # 入口：express 静态 + /ws 端点、消息分发、心跳、优雅停机
│   ├── protocol.ts             # ★ 唯一事实源：wire 协议类型（client↔server 消息）
│   ├── agent-service.ts        # 核心：ClientSession（每客户端一个会话组，可并行多个对话）+ AgentService
│   │                           #   · 多对话并发：convs Map<convId, Conversation>，每个对话独立
│   │                           #     AgentSessionRuntime（new_chat 不再杀旧对话，switch_conversation
│   │                           #     只换 activeId；模型共享一个 ModelRuntime；消息序列化缓存按对话隔离；
│   │                           #     set_cwd 切到目标项目自己的对话，不重建；「运行的对话」列表按项目、
│   │                           #     运行中被挤到后台才入列，打开后不继续再切走才移出）
│   │                           #   · WebUIContext：把扩展的 widget/status/dialog 桥接到浏览器
│   │                           #   · 附件构建（inline/reference/lines 三种模式）
│   │                           #   · readFile 预览（512KB 上限、二进制检测、路径越界拦截）
│   │                           #   · 文件列表按平台拆分（readDirForUI）：win32 稳定+全量优先——ACL
│   │                           #     保护目录降级为空列表+警告（不炸面板）、目录符号链接/junction 跟随、
│   │                           #     上限 2000 并上报 truncated；posix（mac/linux）保持原逻辑（上限 500）。
│   │                           #     IGNORED_ENTRIES 也分平台：win 只藏 node_modules/.git/.pi-web/垃圾文件，
│   │                           #     dist/.next/venv 等全部可见
│   │                           #   · 模型管理 / auth.json / models.json / 会话列表 / cwd 切换
│   │                           #   · 每客户端持久化 lastCwd + 最近项目（<dataDir>/client-state.json，
│   │                           #     重启后恢复上次工作目录；projects 消息推送最近项目列表）
│   │                           #   · 编辑重问（edit_message）：按消息 id 解析 entryId → runtime.fork
│   │                           #     新建分支会话（保留该问题之前的历史，原对话不动）→ 重新 prompt
│   │                           #   · 自更新（check_update/update_app）：读自身 package.json 版本，
│   │                           #     对比 npm registry，npm i -g 升级后自动重启（launchd KeepAlive /
│   │                           #     systemd Restart 退出即拉起；前台派生子进程等旧进程释放端口后接管；
│   │                           #     Docker 容器内提示外部重启）
│   ├── serialize.ts            # SDK 消息 → UiMessage 序列化（截断、稳定 id、对象缓存）
│   └── terminals.ts            # TerminalManager（PTY 生命周期）+ .pi/commands.json 读写
├── web/                        # 前端（React + Vite，编译到 web/dist/）
│   ├── vite.config.ts          # dev 端口 5173，/ws 代理到后端
│   ├── src/
│   │   ├── App.tsx             # 顶层布局：TopBar / LeftPanel / MessageList / ChatInput /
│   │   │                       #   RightPanel / FooterBar / Dialog / 各 Modal / FilePreview
│   │   ├── use-chat.ts         # ★ useChat()：WebSocket 连接管理、reducer 状态机、
│   │   │                       #   终端输出 bridge（未挂载终端的输出先缓冲）
│   │   ├── types.ts            # ★ wire 协议镜像（与 server/protocol.ts 手工同步，仅类型）
│   │   ├── i18n.tsx            # ★ 中英文案（zh 默认），新增 key 必须两处都加
│   │   ├── styles.css          # ★ 全部样式（按组件分区，带注释分隔线）
│   │   ├── sounds.ts           # WebAudio 提示音
│   │   ├── download.ts         # 下载：downloadFile（fetch→blob→objectURL，绕开 Chrome
│   │   │                       #   Safe Browsing 对 HTTP 下载的拦截，错误可读；Chromium 安全上下文
│   │   │                       #   下优先 showSaveFilePicker（Windows 下 blob 下载仍会被静默拦截时
│   │   │                       #   的解法），Windows 自动清洗非法保存名）
│   │   └── components/         # 见下
│   └── dist/                   # 构建产物（gitignore，但打进 npm 包）
├── bin/pi-web-ui.mjs           # CLI：前台启动（就绪后自动打开浏览器，--no-browser 关闭）/ --port --cwd --data-dir / server install|uninstall|start|stop|restart|status
│                               #   （macOS→launchd，Linux→systemd，Windows→schtasks 计划任务、隐藏窗口）
├── deploy/                     # 部署示例：launchd plist / systemd unit / Windows 任务 XML
├── Dockerfile / docker-compose.yml
├── freeze-test.mjs 等           # 仓库根的手写 Playwright E2E 脚本（chromium headless）
└── tsconfig.server.json / web/tsconfig.json
```

`web/src/components/` 速览：

| 组件 | 职责 |
| --- | --- |
| `FilePreview.tsx` | 文件预览弹窗：行号、点选/拖拽/Shift 选区、添加到对话（lines 附件） |
| `LeftPanel.tsx` | 左栏：最近项目（点击切换 cwd）+ 运行的对话（≥1 个时显示，活跃高亮、流式绿点，按当前项目过滤；固定在历史列表上方独立滚动）+ 历史对话（标题不随列表滚动） |
| `RightPanel.tsx` | 文件树浏览（list_files，目录过大时显示截断提示），文件名点击→预览，📎/🔗/👁 附件按钮；服务端对**当前列出目录**做 fs.watch，改动推 `file_changed`（新协议消息）→ 立即静默重列，失败/不支持的文件系统回落 10s 轮询 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xing-shuyin/pi-web-ui](https://github.com/xing-shuyin/pi-web-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
