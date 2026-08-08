---
trigger: always_on
description: > 本文件面向 AI 编码助手。阅读前假设你对本项目一无所知。内容基于当前仓库实际结构、代码与配置整理，请勿凭经验泛化。
---

# AGENTS.md — cyancat / DBStudio

> 本文件面向 AI 编码助手。阅读前假设你对本项目一无所知。内容基于当前仓库实际结构、代码与配置整理，请勿凭经验泛化。

---

## 1. 项目概述

**cyancat**（对外产品名 **DBStudio**）是一款面向开发者、数据工程师和轻量 DBA 的跨平台数据库桌面客户端，采用 Navicat 风格的对象树 + 表设计器 + SQL 编辑器交互模型。

核心能力：

- 多数据源连接管理（MySQL / MariaDB / PostgreSQL / SQLite / StarRocks）。
- 对象树浏览器：逐层懒加载 database/schema/table/字段/索引/外键。
- 可视化表设计器：字段、索引、外键网格编辑，所有结构变更先预览 DDL 再执行。
- SQL 编辑器：基于 Monaco Editor，支持执行、格式化、注释切换。
- 结果网格：虚拟滚动、列宽拖拽、分页，导出 TSV / INSERT SQL / CSV。
- 安全确认：高风险 DDL（DROP TABLE / DROP DATABASE）强制二次确认。
- 凭据加密：AES-GCM 加密存储连接密码，生产环境推荐配置主密钥。
- 跨平台桌面应用：macOS / Windows / Linux。

源码仓库：[https://github.com/cyan-daimao/cyancat](https://github.com/cyan-daimao/cyancat)  
许可证：MIT

---

## 2. 技术栈

| 层级 | 技术 |
|------|------|
| 桌面框架 | Wails v2（Go + 原生 WebView） |
| 后端 | Go 1.25+，分层架构（adapter / application / domain / infra） |
| 本地存储 | SQLite（GORM），路径 `~/.cyancat/cyancat.db` |
| 数据库驱动 | go-sql-driver/mysql、jackc/pgx/v5、mattn/go-sqlite3 |
| 日志 | rs/zerolog |
| 前端 | React 18 + TypeScript + Vite |
| 样式 | Tailwind CSS + shadcn/ui（Radix UI 原语），支持浅色/深色/跟随系统主题（`frontend/src/lib/theme.tsx`） |
| 状态管理 | Zustand |
| 编辑器 | Monaco Editor（`@monaco-editor/react`） |
| 表格 | TanStack Table + TanStack Virtual |

---

## 3. 架构与代码组织

### 3.1 整体目录结构

```
cyancat/
├── main.go                    # 入口 + 依赖注入（DI）组装
├── wails.json                 # Wails 应用配置
├── go.mod / go.sum            # Go 依赖
├── internal/                  # Go 后端源代码
│   ├── adapter/               # Wails API 绑定、DTO、转换函数
│   ├── application/           # 应用服务（连接 / 查询 / Schema）
│   ├── domain/                # 领域模型与仓库接口
│   └── infra/                 # 基础设施实现
├── frontend/                  # 前端源码
│   ├── src/
│   │   ├── components/        # 按功能组织的 React 组件
│   │   ├── lib/api/           # Wails 绑定封装
│   │   ├── stores/            # Zustand 状态管理
│   │   └── ...
│   └── wailsjs/               # Wails 自动生成的 JS/TS 绑定（禁止手动编辑）
├── scripts/build-all.sh       # 一键交叉编译脚本
├── doc/                       # 产品需求与构建文档
│   ├── BUILD.md               # macOS/Windows 打包指南
│   └── DBStudio-product-requirements.md
└── README.md
```

### 3.2 DDBD 四层架构

```
adapter → application → domain → infra
```

| 层 | 路径 | 职责 |
|----|------|------|
| **adapter** | `internal/adapter/` | Wails 绑定入口、HTTP API 结构体（实为命名约定）、DTO 与转换函数 |
| **application** | `internal/application/<biz>service/` | 服务接口与实现、Cmd/Query/BO 对象、编排业务逻辑 |
| **domain** | `internal/domain/<biz>/` | 充血模型实体、Repository 接口 |
| **infra** | `internal/infra/` | GORM 仓库实现、数据库驱动抽象、会话管理、加密、日志、事件总线 |

当前三个垂直切片：

- `connectionservice`：连接 CRUD、测试连接、打开/关闭会话。
- `queryservice`：SQL 执行、流式结果、查询历史。
- `schemaservice`：Schema 浏览、DDL 生成与执行、表设计器支撑。

### 3.3 驱动抽象（核心基础设施）

`internal/infra/driver/driver.go` 定义数据库驱动抽象：

- `Driver` / `Conn`：建立连接、执行 SQL、流式查询。
- `Dialect`：方言差异（标识符引号、参数占位符、默认 LIMIT）。
- `Inspector`：元数据查询（database / schema / table / 字段 / 索引 / 外键）。
- `DDLGenerator`：方言专属 DDL 生成。
- `RowStream`：大结果集流式游标。

已注册驱动：

- `internal/infra/driver/mysql`
- `internal/infra/driver/postgres`
- `internal/infra/driver/sqlite`
- `internal/infra/driver/starrocks`（MySQL 协议兼容）

驱动注册在 `main.go` 中完成：

```go
driver.Register(mysqldriver.New())
driver.Register(postgresdriver.New())
driver.Register(sqlitedriver.New())
driver.Register(starrocksdriver.New())
```

### 3.4 数据转换链

每层边界使用显式 `ToXxx` 转换函数，**不使用反射**：

```
读取:  DO → Domain → BO → DTO
写入:  Request → Cmd → Domain → Repository → DO
```

转换函数位置：

- `adapter/dto/*_dto.go`：Request↔Cmd，BO→DTO。
- `application/<biz>service/*_convert.go`：Cmd→Domain，Domain→BO。
- `infra/db/<biz>repo/*_convert.go`：DO↔Domain。

### 3.5 关键运行时组件

- **`infra/session/manager.go`**：运行时连接池，维护 `connectionID → driver.Conn` 的长连接。注意：`session.Manager` 属于基础设施层，不是领域层。
- **`infra/eventbus/bus.go`**：封装 Wails `runtime.EventsEmit`，用于后端向前端推送流式查询事件（`query:rows`、`query:done`、`query:error`、`connection:state`）。`Init(ctx)` 必须在 `OnStartup` 中调用，因为 `ctx` 在启动前不可用。
- **`infra/crypto/aes.go`**：AES-GCM 加解密，用于连接密码存储。
- **`infra/keychain/keychain.go`**：OS Keychain 封装（V1.0 当前为 AES 兜底实现）。
- **`infra/logger/logger.go`**：zerolog 日志，应用退出时显式 `Close()` 以保证 Windows GUI 子系统下日志落盘。

### 3.6 前端组织

- `frontend/src/lib/api/*.ts`：封装 `wailsjs/go/http/*` 绑定，统一解包 `{code, message, data}`，非 200 时抛出错误。
- `frontend/src/stores/*.ts`：Zustand 状态管理（`connection`、`query`、`schema`、`designer`、`sql-hints`）。
- `frontend/src/components/connection/`：连接列表与对话框。
- `frontend/src/components/object-tree/`：对象树浏览器与右键菜单。
- `frontend/src/components/object-designer/`：表设计器、DDL 预览、风险确认。
- `frontend/src/components/sql-editor/`：SQL 编辑器工作区。
- `frontend/src/components/data-table/`：结果网格。
- `frontend/src/components/ui/`：shadcn/ui 基础组件。
- 路径别名 `@` 指向 `frontend/src`。

---

## 4. 构建与运行命令

### 4.1 前置依赖

| 依赖 | 版本/说明 |
|------|----------|
| Go | 1.25+ |
| Node.js | 现代 LTS |
| Wails CLI | v2.12.0（`go install github.com/wailsapp/wails/v2/cmd/wails@v2.12.0`） |
| macOS | WebKit（系统内置）、Xcode Command Line Tools |
| Windows（交叉编译） | mingw-w64（`brew install mingw-w64`） |
| Linux（交叉编译） | musl-cross（`brew install FiloSottile/musl-cross/musl-cross`） |

自检命令：

```bash
wails doctor
which x86_64-w64-mingw32-gcc
```

### 4.2 开发模式

```bash
# 启动后端热重载 + 前端热更新
wails dev
```

### 4.3 构建

```bash
# 当前平台
wails build

# 指定平台
wails build -platform darwin/universal    # macOS 通用二进制（推荐分发）
wails build -platform darwin/arm64        # Apple Silicon

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cyan-daimao/cyancat](https://github.com/cyan-daimao/cyancat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
