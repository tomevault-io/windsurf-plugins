---
trigger: always_on
description: > 本文件用于帮助快速理解仓库结构与运行方式，便于开发、重构、测试与排障。
---

# RoadbookMaker 代码库架构概览（给 Agent / 开发者）

> 本文件用于帮助快速理解仓库结构与运行方式，便于开发、重构、测试与排障。

## 1. 项目概览

RoadbookMaker 是一个基于网页的地图标记与行程规划工具：

- **前端**：`static/` 下的纯原生 JavaScript + Leaflet，直接在浏览器运行，无构建步骤。核心文件：
  - `static/index.html`：入口 HTML。
  - `static/style.css`：样式定义（含 CSS 变量实现的明亮/暗色模式）。
  - `static/script.js`：`RoadbookApp` 主入口与类骨架，保留全局配置、`constructor`、`init()`、`bindEvents()`、分享导入/移动端/调试等主流程。
  - `static/app_*.js`：按职责拆分的前端模块，通过 `RoadbookApp.prototype` 扩展主类。当前包含 `app_utils.js`、`app_theme.js`、`app_history.js`、`app_search.js`、`app_ai.js`、`app_date_filter.js`、`app_date_notes.js`、`app_sidebar.js`、`app_detail_panels.js`、`app_tooltips.js`、`app_map.js`、`app_io.js`。
  - `static/help_tour.js`：新手引导（罩子高亮 + tooltip + 自动演示点/线/日程编辑）。
  - `static/online_mode.js`：在线模式 API 交互。
  - `static/ai_assistant.js`：AI 助手功能，实现自然语言驱动的地图操作。
  - `static/debug.js`：调试模块，提供应用状态快照、日志捕获与环境信息查看（见第 10 节）。
  - `static/html_export.js`：负责生成包含完整数据与交互逻辑的独立 HTML/TXT 导出文件。
- **后端**：`backend/` 下的 Go + Gin API 服务（入口 `backend/cmd/roadbook-api/main.go`），提供：
  - 在线模式（登录/JWT、计划 CRUD、分享读取）
  - 地图搜索聚合与代理（`/api/cnmap/search`、`/api/tianmap/search`）
  - AI 助手代理（`/api/v1/ai/*`），负责与大语言模型交互
  - `trafficpos` 等辅助 API（`/api/trafficpos`）
- **部署**：Nginx 提供静态资源，并反向代理 `/api/` 到后端（见 `nginx.prod.conf:25`）；Docker 镜像把前端 + 后端打包在一起（见 `Dockerfile`、`Dockerfile.local`、`docker-entrypoint.sh`）。

在线模式数据持久化是“文件系统仓库”：计划以 `data/<id>.json` 形式写入磁盘（见 `backend/internal/plan/repository.go:16`）。

## 2. 构建与常用命令

### 前端

- 纯静态资源：直接打开 `static/index.html` 或由 Nginx 提供。
- 前端对后端基址的判定：
  - `localhost/127.0.0.1/file://` 时使用 `http://127.0.0.1:5436`（见 `static/script.js:5`）
  - 否则使用当前域名（见 `static/script.js:8`）

### 后端

- 生成后端配置（交互式）：`./scripts/generate_config.sh`（写入 `backend/configs/config.json`，见 `scripts/generate_config.sh:23`）
- 管理 Agent 软链接：`./scripts/link_agents.sh`（用于同步 `.agent` 配置到 `.coco/.gemini` 等目录，见 `scripts/link_agents.sh`）
- 编译后端：`./backend/scripts/build.sh`（产物：`backend/roadbook-api`，见 `backend/scripts/build.sh:34`）
- 启动后端：在 `backend/` 目录执行 `./roadbook-api`（README 与 `backend/cmd/roadbook-api/main.go:41`）

### Docker

- 从项目根目录构建本地镜像：`./build.sh`（会先在 `backend/` 里 `go build` 再 `docker build -f./Dockerfile.local`，见 `build.sh:16`、`build.sh:21`）
- 镜像运行示例：`docker run -d -p 5215:80 roadbook`（见 `build.sh:26`）

容器启动逻辑：`docker-entrypoint.sh` 会启动 Nginx，并在 `/app` 下启动后端二进制（见 `docker-entrypoint.sh:4`、`docker-entrypoint.sh:7`）。

## 3. 代码风格与约定（基于仓库现状）

### Go（`backend/`）

- 模块：`module github.com/chenxuan520/roadmap/backend`，`go 1.18`（见 `backend/go.mod:1`、`backend/go.mod:3`）。
- 框架与库：Gin（HTTP）、`golang-jwt/jwt/v4`（JWT）、`google/uuid`（计划ID）、`golang.org/x/time/rate`（限流）（见 `backend/go.mod:6`-`backend/go.mod:9`）。
- 路由组织：在 `backend/internal/server/server.go` 内组装 Gin Engine；`/api/v1` 下是认证/计划接口，`/api` 下保留搜索与其它公共接口（见 `backend/internal/server/server.go:65`、`backend/internal/server/server.go:97`）。

### 前端（`static/`）

- 无框架：主入口是 `static/script.js`，核心业务已拆到 `static/app_*.js`；`static/online_mode.js` 负责在线模式/云端保存/登录，`static/ai_assistant.js` 负责 AI 助手。
- 前端模块约定：`script.js` 必须先于 `app_*.js` 加载，由 `script.js` 定义 `RoadbookApp`，再由各模块扩展 `RoadbookApp.prototype`。
- 若修改 `index.html` 脚本顺序，必须保证 `window.app = new RoadbookApp()` 执行前，所有 `app_*.js` 已经完成注册。
- 新手引导：`static/help_tour.js`（罩子高亮 + 步骤引导；入口在帮助面板 `❓` 内）
- 调试逻辑独立于 `static/debug.js`，通过 `DebugModule` 类实现，仅在需要时被调用。
- 在线 token 存储：`localStorage` 的 key 为 `online_token`（见 `static/online_mode.js:6`）。
- 在线模式快捷键：在在线模式且已登录时，`Ctrl/Cmd+S` 触发 `saveToCloud()` 云端保存（见 `static/online_mode.js:41`）。
### 维护规则

- **命名约定**：不要使用下划线 `_` 开头命名“私有”函数、变量或绑定的事件处理器（推荐使用 `boundHandler` 形式），保持简洁。
- **注释约定**：修改代码时，必须保留原有的关键注释（特别是数据结构定义、业务逻辑说明等），除非该逻辑已被完全移除。
- **语法检查**：每次修改代码后，必须自行检查语法，确保代码可编译/可运行。
  - **Go 后端**：需要检查 unused imports 等编译错误。
  - **前端 JavaScript**：必须使用 `node -c <filename>.js` 命令检查语法。
- **变量作用域检查**：在重构代码（特别是移动代码块）时，必须严格检查变量的作用域（Block Scope）。
  - **错误案例**：将定义在 `try` 块内部的 `const` 变量（如 `assistantMsg`）引用的代码移动到 `try...finally` 块外部，导致 ReferenceError。
  - **避免方法**：移动代码前，先确认所引用的变量在目标位置是否可见。如果需要在块外使用，请将变量声明提升到块外（使用 `let`）。

本仓库未提供统一格式化/校验工具配置文件（例如 ESLint/Prettier/Go fmt hook），修改时建议保持周边代码的现有缩进与命名风格。

## 4. 测试

### Go 单元测试

- 计划仓库测试：`backend/internal/plan/repository_test.go`（覆盖 Save/FindAll/Delete 等；测试会把 `dataDir` 指向临时目录，见 `backend/internal/plan/repository_test.go:14`-`backend/internal/plan/repository_test.go:34`）。
- 限流器惰性清理测试：`backend/internal/middleware/middleware_test.go`（覆盖 TTL/清理间隔/阈值等，见 `backend/internal/middleware/middleware_test.go:22`）。

运行方式（在 `backend/` 下）：`go test ./...`。

### 后端集成测试脚本

`backend/scripts/backend_test.sh` 会：

- 生成临时 `backend/configs/config.json`（包含 `jwtSecret` 与用户 salted SHA256 hash）
- 编译并启动后端，然后用 `curl`/`jq` 走一组 API（含登录、计划 CRUD、分享）

该脚本依赖：`openssl`、`sha256sum`、`jq`、`curl`（见 `backend/scripts/backend_test.sh:47`、`backend/scripts/backend_test.sh:102`、`backend/scripts/backend_test.sh:115`）。

### 前端端到端测试（Playwright）

前端 E2E 测试位于 `test/`，通过一个轻量静态服务器加载 `static/index.html` 做 UI 交互验证（测试用例会 mock 搜索等外部请求，避免依赖后端）。

- 本地运行（不下载浏览器内核，复用系统 Chrome）：在仓库根目录执行 `cd test && npm run setup:local && npm run test:local`
- CI 运行（runner 上安装 Chromium 后 headless 跑）：`cd test && npm run setup:ci && npm run test:ci`（见 `/.github/workflows/ci_frontend.yml:1`）

更多说明见 `test/README.md`。

## 5. 安全与数据保护

### 配置与密钥

- 后端必须配置 `jwtSecret` 且用户列表不能为空（见 `backend/internal/config/config.go:42`-`backend/internal/config/config.go:50`）。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chenxuan520/roadbook](https://github.com/chenxuan520/roadbook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
