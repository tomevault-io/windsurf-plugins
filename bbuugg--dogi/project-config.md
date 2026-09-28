---
trigger: always_on
description: 面向 AI 编码助手与后来者的工作手册。**先读第一节（项目概览）与第二节（开发命令）；
---

# Dogi — 项目说明与开发约束

面向 AI 编码助手与后来者的工作手册。**先读第一节（项目概览）与第二节（开发命令）；
动手改某个模块前，读第四节的对应机制与第五节对应条目。**

第六节是本项目的踩坑库，每条都按「触发信号 → 根因/约束 → 正确做法 → 验证方式」组织，
全部是实测结论（不是推测、不是网上抄的通用建议）。**条目里写着「别改成 X」的地方，
都是有具体事故背景的，改之前先想清楚为什么。**

---

## 一、项目概览

### 1.1 定位

**Dogi** 是一个 AI 驱动的桌面运维工作台：把「连机器 → 干活 → 记下来 → 调接口 → 让 AI 代办」
收在一个 Electron 应用里。左侧是活动栏功能区，右侧是 VS Code 式的分屏标签组，
内置终端 / SSH / SFTP / 服务器监控 / 接口调试 / 笔记 / 脚本 / 插件宿主，
以及一个能读写文件、执行命令、调用技能的 AI Agent。

### 1.2 技术栈

| 层 | 选型 |
| --- | --- |
| 桌面容器 | Electron 44（`contextIsolation` + preload 白名单 IPC，无 nodeIntegration） |
| 渲染端 | React 19 + TypeScript 7 + Tailwind v4 + **antd 6**（唯一 UI 库） |
| 状态 | zustand（单一 store：`src/renderer/src/stores/app-store.ts`） |
| 终端 | `@xterm/xterm` v6（WebGL 渲染）+ `node-pty`（本地）/ `ssh2`（远程）+ `zmodem.js`（rz/sz 传文件） |
| 编辑器 | Monaco（本地资源，`scripts/copy-monaco.cjs` 拷贝到 `public/`） |
| AI | Vercel AI SDK v7（openai / anthropic / deepseek / google / openai 兼容）+ `@modelcontextprotocol/sdk`（MCP）+ `@agentclientprotocol/sdk`（外部 ACP agent） |
| 持久化 | `electron-store` + `safeStorage`（凭据加密，Windows 走 DPAPI） |
| 构建 | 自建三配置 Vite（见 3.1），无 electron-vite |
| 打包 | electron-builder（NSIS / dmg / AppImage+deb） |

### 1.3 当前功能

**活动栏功能区**（`src/renderer/src/app/activities.tsx`，顺序可拖拽、可隐藏）

| 功能区 | 侧边栏 | 主区域 |
| --- | --- | --- |
| **主机** | 主机列表（分组 / 拖拽 / 颜色）+ 下半区「脚本」分区（可折叠、可拖高） | 终端标签、SFTP 文件管理标签 |
| **AI Agent** | 工作区 → 会话两层树，会话行带状态图标（等回答 / 运行中 / 静止） | Agent 会话页（对话流 + 内嵌终端 + 工作区文件树/预览 + 快捷功能） |
| **笔记** | 笔记列表（分组 / 拖拽 / 搜索） | Monaco 编辑器标签，语言可选 |
| **接口请求** | 保存的请求列表（分组 / 拖拽 / 历史） | HTTP 调试页 / WebSocket 调试页 |
| **自动化** | 自动化脚本列表（分组 / 拖拽 / 搜索，按名称或代码搜） | 脚本页（左 Monaco 编辑器 + 右内嵌浏览器 + 底部运行日志） |
| **插件管理** | 已安装插件列表 | 插件视图（以标签页打开）；内置：Redis 客户端（🔴）、端口占用（🔌） |

**终端**

- 本地终端：`node-pty`，shell 由 `services/terminal/shells.ts` 探测（PowerShell / pwsh / CMD / Git Bash / WSL / bash / zsh / fish…），偏好里可指定默认 shell。
- SSH：`ssh2`，密码或私钥认证，握手阶段进度推送（`resolving → handshake → authenticating → opening-shell → retrying → ready`），失败自动重连。
- Mosh：主机可单独开启（新建 / 编辑主机表单的「使用 Mosh」开关，`SshProfile.useMosh`）—— SSH 只做引导，终端数据走本地 mosh-client 的 UDP；机制与前置条件见 4.12。
- 会话输出环形缓冲 **256KB/会话**（`MAX_OUTPUT_BUFFER`），供 AI 工具按偏移增量读取。
- `TERM=xterm-256color` 硬编码 —— 否则远端 ncurses 程序（htop / btop / lazygit）按 8 色渲染成黑白。
- 每会话独立的 AI 助手（内嵌在终端页底部，可折叠）。
- zmodem：`sz`/`rz` 走系统对话框选文件 / 存文件（`zmodem:*` 三个通道）。
- 终端配色方案、字号缩放、选中即复制、右键粘贴、命令预测（历史补全）等偏好。
- 执行的命令与输出自动记进主机日志（`terminal` 作用域，来源带 [AI] / [脚本] 标记），每会话另有逐字节原始输出文件（见 4.13）。

**主机与运维**

- 主机分组 / 强调色 / 拖拽排序；`kind: 'ssh' | 'local'` 统一建模。
- **SFTP**：浏览、上传文件 / **上传文件夹（递归，目录内每个文件一笔独立传输）**、下载（含目录递归）、远端复制 / 移动、删除 / 新建目录、重命名；进度经 `sftp:progress` 广播到状态栏的传输托盘；用户取消不算错误（`TransferCancelledError`）。
- **传输托盘**（状态栏右下）：结束的任务**不自动移除**（用户要求：完成后留在面板里，由用户逐条 × 或「清除已完成」清理）；已完成的上传 / 下载条目带**「打开文件位置」图标**（`shell:revealPath`：目录直接打开，文件在所在目录中选中；复制 / 移动没有本地侧、已取消 / 失败不给图标）。
- **服务器监控**：经 SSH `exec` 周期采集 `/proc` + `df`（CPU / 内存 / 负载 / 网速 / 磁盘 / uptime），间隔可配（`monitorInterval`）。
- **脚本**：侧边栏分区管理，命令面板可「运行脚本」（有终端直接写入执行，无终端则弹框选主机连上去跑）。
- **主机日志**：SSH 连接（按来源标签分终端会话 / Mosh 引导 / SFTP / SSH 隧道 / 连接测试）、会话就绪 / 关闭 / 断线重试、隧道启停与转发失败、SFTP 连接、主机指纹记录与重置统一记成结构化条目；终端里执行的命令与输出也在内（`terminal` 作用域，机制见 4.13）。

**AI 两条产品线**（共用一套流式事件与渲染组件，别各写一份）

1. **终端 AI 助手**（`AiPanel`）：挂在终端页面，工具作用于当前终端会话 —— `run_in_terminal` / `send_keys` / `read_terminal_output` / `list_terminal_sessions`。
2. **工作区 Agent**（`AgentPage` / `AgentConversationView`）：绑定本地目录，工具为 `list_files` / `find_files` / `search_files` / `read_file` / `write_file` / `edit_file` / `delete_file` / `execute_command` / `read_skill` / `browser_*`（见下），另有工作区文件树与图片 / 视频 / SVG 预览，以及可折叠的**内嵌浏览器面板**（看 Agent 正在操作哪个页面）。

两者共同支持：

- 后端二选一：`ai-sdk`（内置，复用模型配置）或 `acp`（连接外部 ACP agent，如 codex-acp）。
- **模型 / 后端按会话独立**（见 4.3）。
- 权限模式 `full` / `confirm`（确认模式下执行命令前弹确认卡）。
- `ask_followup_question`：AI 在回合中途向用户发**结构化选择题**，答完同一回合继续（见 4.5）。
- **MCP**：任意 stdio MCP server，工具自动合并给 AI。
- **技能（Skills）**：发现 `SKILL.md` 目录，渐进式披露 + `read_skill` 工具按需读（见 4.7）。
- 思考内容（reasoning）与工具调用渲染成**可折叠横条**，不是卡片（见 6.5 第 18 条）。
- 一轮结束且应用不在前台时发系统通知。

**浏览器自动化**（Playwright，机制见 4.11）

- **脚本管理**：脚本列表（分组 / 拖拽 / 搜索）+ Monaco 编辑器；一个脚本 = 一个标签 = 一个浏览器会话。
- **内嵌浏览器**：浏览器**无窗口**跑（headless），画面走 CDP screencast 镜像进面板；面板里的鼠标 / 键盘 / 滚轮再转发回页面，所以「用官方引擎录制」和「画面在面板内」能同时成立。
- **录制**：官方 codegen（`context._enableRecorder({ recorderMode: 'api' })`），操作实时生成 Playwright 代码写进编辑器；停止录制时统一落盘。
- **运行**：逐行执行脚本（`page` / `context` / `browser` / `expect` / `log` 注入作用域），每步与结果广播到底部日志，可中途中止。
- **浏览器来源**：偏好 `browserChannel`（`auto` / `bundled` / `msedge` / `chrome`），auto 按「自带 → Edge → Chrome」逐个尝试启动。
- **Agent 浏览器工具**：`browser_navigate` / `browser_snapshot` / `browser_click` / `browser_type` / `browser_press` / `browser_wait_for` / `browser_evaluate` / `browser_screenshot` / `browser_close`；定位用可访问性快照里的 `[ref=eN]`（`aria-ref` 选择器引擎），页面一变 ref 失效需重新快照。

**接口调试**

- HTTP：方法 / 头（键值对数组，保留空行）/ body、超时、代理、跳过 TLS 校验、手动取消、cURL 导入、请求历史、响应耗时与体积。
- WebSocket：长连接、附加握手头、子协议、`wss` 自签证书、文本 / 二进制帧（base64）、按 `connId` 隔离多标签。
- 与终端同款的多标签 / 分屏；一个请求 = 一个标签（新建即落盘）。

**应用外壳**

- VS Code 式**面板树分屏**（`app/layout/pane-layout.ts`）：向上下左右拆分、拖拽调比例、标签跨组移动、标签条溢出时激活标签自动滚入可视区。
- **命令面板**（`Ctrl+Shift+P`）：命令 / 脚本 / 主机 / 插件的统一入口；插件可注册命令。
- **应用内快捷键**可改（偏好 → 快捷键），带冲突检测。
- 自定义标题栏、状态栏（保存状态 / 监控条 / AI 开关 / 传输托盘 / 左下角全局菜单）、二次确认关闭标签。
- 主题：明暗 + 强调色方案 + 终端独立配色；**首帧不闪**（见 4.6）。
- 托盘常驻、单实例锁、最小化到托盘。
- **数据导入 / 导出**：主机 / 笔记 / 接口请求打包成 zip（自实现，见 6.7）。

### 1.4 目录地图

```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bbuugg/dogi](https://github.com/bbuugg/dogi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-28 -->
