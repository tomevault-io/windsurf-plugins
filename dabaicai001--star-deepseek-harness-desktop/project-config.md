---
trigger: always_on
description: > 本文件供 AI Agent(以及人类贡献者)快速理解项目结构、技术栈、约定与工作方式。
---

# StarHub — Agent 协作指引

> 本文件供 AI Agent(以及人类贡献者)快速理解项目结构、技术栈、约定与工作方式。
> 任何架构级变更请同步更新 `docs/` 与本文件。

**快速导航**:[1. 项目定位](#1-项目定位) · [2. 仓库信息](#2-仓库信息) · [3. 目录结构](#3-目录结构v032-实际快照) · [4. 技术栈](#4-技术栈)([4.4 设计系统](#44-设计系统design-system)) · [5. 关键命令](#5-关键命令) · [6. 开发约定](#6-开发约定)([6.5 版本发布](#65-版本发布强制) / [6.6 必 commit](#66-修改后必-commit强制)) · [7. 测试/构建](#7-测试--构建)([7.3 UI 回归](#73-真实布局浏览器回归ui-改动强制)) · [8. 沟通协作](#8-沟通与协作) · [9. 文档维护](#9-文档维护强制) · [10. 已知坑索引](#10-已知坑与注意事项索引) · [11. 路线图](#11-路线图与任务优先级) · [12. 协作 Tips](#12-agent-协作-tips)

---

## 1. 项目定位

**StarHub** 是一款跨平台(Windows / macOS / Linux)的桌面应用,把开发运维日常所需的多种工具整合到一个窗口:

- 🗄️ 数据库客户端(MySQL / PostgreSQL / SQLite / Redis / ClickHouse / SQL Server / Oracle / 国产库)
- 🖥️ SSH 终端(跳板机、隧道、命令广播、批量执行)
- 📁 SFTP 文件传输(三栏布局、ZMODEM/SCP、断点续传)
- 🐳 Docker 面板(容器/镜像、SSH 通道连远程 Docker、镜像加速)
- 🤖 AI 助手(自然语言驱动本机与远程运维,Function Calling)

详细功能矩阵见 [`docs/技术方案.md`](./docs/技术方案.md) 第 3 章(280+ 子功能,P0/P1/P2/P3 标注)。

---

## 2. 仓库信息

| 项 | 值 |
|---|---|
| GitHub | https://github.com/dabaicai001/starhub |
| 主分支 | `main` |
| 协议 | MIT |
| 立项时间 | 2026-06-04 |
| 当前版本 | v0.92.1(产物行「+ N 个文件」改为打开右侧贴边 drawer(撤回 v0.91.0 行内展开):按新增/修改分组列出本轮全部变更文件(完整路径 + +/- 行数),分组标题可折叠,行点击走与徽章相同的壳内查看窗优先打开器;Esc / 遮罩 / × 三种方式关闭且焦点回到「+ N」按钮;「在文件夹中显示」移入 drawer 底栏(loopback + `canOpenPath` 门禁不变);v0.91.0 的 `.list` / `.collapse` / `.listRow` / 新增修改标记等行内展开代码与样式全部删除。) |

---

## 3. 目录结构(v0.32 实际快照)

```
starhub/
├── .github/                  # GitHub 配置(ISSUE_TEMPLATE / PR_TEMPLATE / CI)
├── .gitignore
├── AGENTS.md                 # 本文件
├── CHANGELOG.md
├── LICENSE
├── README.md
├── docs/
│   ├── 技术方案.md            # 完整技术方案
│   ├── 设计系统.md            # 设计系统规范(token / 组件类 / 反模式)
│   ├── 踩坑记录.md            # 已知坑详细内容
│   ├── 已知坑索引.md          # 已知坑主题索引(见第 10 节)
│   └── 架构图.html            # 可视化架构图
│
├── legacy-core/              # 脱离 Vue 的纯 TypeScript 工具与服务,由 Node 测试覆盖
├── vendor/deepseek-harness/  # DSH 主壳与 StarHub React 工作台
│   ├── apps/starhub-window/  # 独立 React 资产工作台构建入口
│   └── packages/starhub/     # React 导航、设置、资产管理和静态托管插件
│
├── src-tauri/                # 桌面壳与主进程 - Rust
│   ├── src/
│   │   ├── main.rs            # 入口
│   │   ├── mcp.rs             # MCP Server 管理
│   │   ├── commands/          # 全部 Tauri Command(ssh/sftp/db/docker/ai/mcp/asset/audit/alert/local/secret/sidecar/broker/file)
│   │   ├── ssh/               # SSH 会话(russh):auth / session / known_hosts / sftp_transport
│   │   ├── sftp/              # SFTP 会话与传输(russh-sftp)
│   │   ├── db/                # 本地 SQLite 持久化(sqlx)
│   │   ├── ai/                # AI Gateway
│   │   ├── keyring/           # 系统 Keyring 封装
│   │   └── sidecar/           # Go Sidecar 启动器(路径解析见 docs/踩坑记录.md 第 4 节)
│   ├── capabilities/          # Tauri 权限(含 detach-* 窗口)
│   ├── Cargo.toml
│   ├── tauri.conf.json
│   └── icons/                 # 打包图标(见 docs/踩坑记录.md 第 6 节)
│
├── sidecar/                  # Go Sidecar - 数据库/中间件代理
│   ├── main.go                # 入口(stdio JSON-RPC server)
│   ├── adapters/              # 各 DB / 中间件适配器
│   │   ├── mysql.go / postgres.go / sqlite.go / redis.go
│   │   ├── clickhouse.go / mssql.go / elasticsearch.go
│   │   ├── broker.go          # Kafka / NSQ 元数据
│   │   ├── docker.go / docker_compose.go / docker_ssh.go
│   │   ├── excel.go / csv.go / backup.go
│   │   └── handlers.go        # RPC 分发(各域 *_handlers.go)
│   ├── pool/                  # 连接池
│   ├── rpc/                   # JSON-RPC 协议
│   ├── bin/                   # 构建输出(starhub-sidecar[.exe])
│   ├── winres/                # Windows 资源(rsrc_*.syso)
│   ├── go.mod
│   └── go.sum
│
├── scripts/                  # 构建脚本(build-sidecar.mjs、Linux 打包与校验)
├── tests/                    # node --test 单测(utils、AI 上下文/滚动)
└── vendor/                   # 上游源码引用(git submodule)
    ├── univer/               # DreamNum Univer v0.25.1
    └── univer-presets/       # DreamNum Univer Presets v0.25.1
```

---

## 4. 技术栈

### 4.1 前端

| 类别 | 选型 | 备注 |
|---|---|---|
| 框架 | React + DeepSeek Harness | DSH Web 主壳、StarHub React 工作台与设置分区 |
| 构建 | Vite 5+ | `starhub-window` 生成 `dist-starhub-react/` |
| 语言 | TypeScript 5+ | strict 模式 |
| 终端 | xterm.js 6 | React SSH 工作台 |
| SQL 编辑 | CodeMirror 6 | React 数据库工作台 |
| Markdown | DSH UI renderer | AI 对话渲染 |
| 资产管理 | React `NewConnectionDialog` | Tauri IPC 创建、编辑、删除和测试连接 |

### 4.2 桌面壳与主进程(Rust)

| 类别 | Crate | 用途 |
|---|---|---|
| 桌面壳 | `tauri` 2.x | 多窗口、权限、Updater |
| 自动更新 | `tauri-plugin-updater` 2 | 检查/下载/安装应用更新 |
| 进程管理 | `tauri-plugin-process` 2 | 更新后重启应用 |
| 异步 | `tokio` | 全异步 |
| SSH | `russh` + `russh-sftp` | SSH / SFTP |
| SFTP | `russh-sftp` 2.x | SFTP client |
| Docker | `bollard` | Docker API |
| HTTP | `reqwest` | LLM API / Webhook |
| 持久化 | `sqlx` (SQLite) | 本地资产/配置 |
| 序列化 | `serde` + `serde_json` | |
| 加密 | `aes-gcm` / `argon2` | 敏感数据 |
| 系统监控 | `sysinfo` | CPU/内存/进程 |
| 密钥 | `keyring-rs` | 系统 Keyring |
| 日志 | `tracing` | |
| 错误 | `thiserror` + `anyhow` | |

### 4.3 Sidecar(Go 1.25+)

| 类别 | 包 | 用途 |
|---|---|---|
| MySQL | `github.com/go-sql-driver/mysql` | |
| PostgreSQL | `github.com/jackc/pgx/v5` | 性能之王,流式一等公民 |
| SQLite | `modernc.org/sqlite` | 纯 Go,无 CGO,跨平台编译无坑 |
| Redis | `github.com/redis/go-redis/v9` | 官方维护 |
| ClickHouse | `github.com/ClickHouse/clickhouse-go/v2` | 官方 |
| SQL Server | `github.com/microsoft/go-mssqldb` | 微软官方 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dabaicai001/star-deepseek-harness-desktop](https://github.com/dabaicai001/star-deepseek-harness-desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
