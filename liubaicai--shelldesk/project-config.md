---
trigger: always_on
description: Tauri 2 + Rust + React 19 + TypeScript + Vite 图形化 SSH 客户端。pnpm（当前 `pnpm@11.8.0`）为包管理器。
---

# ShellDesk — AI Agent Instructions

Tauri 2 + Rust + React 19 + TypeScript + Vite 图形化 SSH 客户端。pnpm（当前 `pnpm@11.8.0`）为包管理器。

## 构建与运行

```bash
pnpm dev            # 通过 Tauri 启动 Vite (127.0.0.1:5173) 和桌面窗口
pnpm typecheck      # tsc --noEmit
pnpm build          # pnpm typecheck && vite build
pnpm test           # 合同检查/前端构建/UI 冒烟/Rust fmt+clippy+test/cargo check
pnpm preview        # Vite preview，绑定 127.0.0.1
pnpm check:contracts # IPC/桌面应用/i18n/Tauri/默认设置/发布脚本合同检查
pnpm check:ui       # Playwright UI 冒烟
pnpm check:rust     # cargo fmt --check + cargo clippy -D warnings + cargo test
pnpm check:rust:coverage # cargo-llvm-cov 覆盖率摘要
pnpm release        # Windows x64 NSIS 安装包
pnpm pack:dir       # Tauri debug bundle
pnpm pack           # Tauri 默认目标打包
pnpm pack:win-x64   # Windows x64
pnpm pack:win-arm64 # Windows arm64
pnpm pack:mac       # macOS
pnpm pack:linux     # Linux
pnpm pack:linux-x64 # Linux x64
pnpm pack:linux-arm64 # Linux arm64
```

**已知坑**：`pnpm dev` 退出后 Vite 可能残留占用 5173 端口。先用 `netstat -ano | findstr :5173` 找到占用 PID，再只对该 PID 执行 `taskkill /PID <pid> /F` 或 `Stop-Process -Id <pid>`，不要粗暴 kill 全部 node 进程。

## AI 开发测试变量

- 如果仓库根目录存在 `.env` 文件，AI Agent 在需要连接测试服务器进行开发、调试或验证时，应先读取其中的测试 SSH 变量，用于自动填充测试服务器连接信息。
- 推荐变量名：`SHELLDESK_TEST_SSH_HOST`、`SHELLDESK_TEST_SSH_PORT`、`SHELLDESK_TEST_SSH_USERNAME`、`SHELLDESK_TEST_SSH_PASSWORD`、`SHELLDESK_TEST_SSH_KEY_PATH`。
- `.env` 只用于本地测试凭据，不要提交到仓库；提交的样例请使用 `.env.example`，并只放占位值。
- 不要在日志、终端输出、提交信息或回复中明文展示 `SHELLDESK_TEST_SSH_PASSWORD`。如果 `.env` 不存在或变量为空，应跳过自动连接并说明缺少测试凭据，不要臆造服务器信息。

## 架构概览

```
src-tauri/
  tauri.conf.json # Tauri 应用、窗口、打包、图标和更新产物配置
  Cargo.toml      # Rust 后端依赖
  src/
    main.rs            # 极薄入口，加载 modules 并调用 bootstrap::run()
    modules.rs         # Rust 后端模块声明与共享 re-export
    bootstrap.rs       # Tauri builder、插件、状态与 command 注册
    ipc.rs             # window.guiSSH 使用的频道分发器
    state.rs           # 共享应用状态、活跃会话和 UI prompt 通道
    connection.rs      # SSH/本地连接生命周期和连接信息
    connection/host_keys.rs # 主机密钥扫描、分类、信任与 known_hosts 同步
    russh_client.rs    # 纯 Rust SSH 客户端、认证、host key 校验、exec、跳板/代理传输
    ssh_transport.rs   # runCommand 高层包装、提权、重试与 host key 刷新
    ssh_tunnel.rs      # russh direct-tcpip 隧道（数据库、浏览器、VNC、HTTP）
    terminal.rs        # 远程 russh PTY 终端和本地 shell 终端生命周期
    remote_fs.rs       # SFTP、文件读写/传输、压缩解压、权限操作
    database/          # MySQL/PostgreSQL/ClickHouse/MongoDB/Redis/SQLite 管理
    browser_proxy.rs   # 远程浏览器 URL 解析与本地反向代理
    http_tunnel.rs     # 基于 SSH 转发的远程 HTTP 请求隧道
    vnc.rs             # VNC 探测、SSH 隧道、noVNC/WebSocket 代理
    vault.rs           # 本地 vault/config/bookmark/settings 存取与校验
    vault_storage.rs   # config/secrets 拆分存储和平台密钥保护
    vault/normalize.rs # 设置、主机、密钥、代理、known_hosts 规范化
    sync_backend.rs    # WebDAV 同步
    updater.rs         # GitHub release 检查与 Tauri updater 安装
    system.rs          # 系统字体与 known_hosts 读取
src/
  App.tsx                  # 主页：主机/密钥/日志/设置、vault 同步、连接入口
  main.tsx                 # React 入口，导入全局样式
  i18n.ts                  # zh-CN / en-US 文案与 useShellDeskI18n
  fontUtils.ts             # 字体工具
  RemoteDesktop.tsx        # re-export RemoteDesktopShell
  RemoteDesktopShell.tsx   # 远程桌面：桌面图标/文件夹/Launchpad、多窗口管理器、Dock、布局设置
  styles/
    index.scss      # 全局样式入口，按级联顺序 @use 模块
    _tokens.scss    # 字体、CSS 变量、主题 token
    foundations/    # reset、基础元素、全局行为
    layout/         # 应用壳、顶部栏、侧边导航
    pages/          # 主机、密钥、日志、设置等页面样式
    remote-desktop/ # 远程桌面 shell、Dock、终端、文件、数据库、监控、运维工具样式
    themes/         # 浅色主题与远程应用主题覆盖
  assets/
    desktop-icons/  # 远程桌面应用图标
    images/         # 应用图标、默认桌面壁纸
    os-icons/       # 主机系统图标
  vite-env.d.ts     # window.guiSSH 类型定义（ShellDeskApi）和全局类型
  components/
    navigation/NavIcon.tsx
    remote-desktop/
      index.ts            # 远程桌面组件 barrel export
      types.ts            # RemoteConnectionInfo / RemoteSystemType
      remoteSystem.ts     # Windows/Linux 命令差异与 PowerShell stdin command 工具
      desktopUtils.ts     # formatDateTime, getErrorMessage
      terminalPresets.ts  # xterm 主题与字体栈
      *Utils.ts / *Providers.ts / *Parsers.ts # 各远程应用的解析、provider、工具函数
      RemoteTerminal.tsx       # xterm.js 多会话终端
      RemoteFileExplorer.tsx   # SFTP 文件管理器（Windows 风格）
      RemoteFilePicker.tsx     # 远程文件选择器（SQLite 等复用）
      RemoteNotepad.tsx        # 远程记事本（highlight.js 代码高亮）
      RemoteBrowser.tsx        # Tauri iframe + Rust browser proxy 浏览器
      RemoteVncViewer.tsx      # noVNC 远程桌面查看器
      RemoteMonitor.tsx        # 系统状态监控
      RemoteMySQL.tsx / RemotePostgres.tsx / RemoteRedis.tsx / RemoteSqlite.tsx
      RemoteProcessManager.tsx / RemoteServiceManager.tsx / RemoteContainerManager.tsx
      RemotePortManager.tsx / RemoteFirewallManager.tsx / RemoteNetworkDiagnostics.tsx
      RemoteDiskAnalyzer.tsx / RemoteDiskManager.tsx / RemotePackageManager.tsx / RemoteScheduledTasks.tsx
      RemoteSecurityAudit.tsx / RemoteLogViewer.tsx / SettingsLoginSessionsPanel.tsx
      RemoteApiDebugger.tsx / RemoteSettings.tsx
  pages/
    KeysPage.tsx    # SSH 密钥管理
    LogsPage.tsx    # 本地日志页
    SettingsPage.tsx # 应用设置页
  types/
    novnc.d.ts      # noVNC 类型补充
```

## 关键设计决策

### IPC 通信模式
- **Rust 后端**：`src-tauri/src/bootstrap.rs` 注册 Tauri command，`src-tauri/src/ipc.rs` 按 channel 分发到各 Rust 模块。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [liubaicai/ShellDesk](https://github.com/liubaicai/ShellDesk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
