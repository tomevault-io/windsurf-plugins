---
trigger: always_on
description: 个人关系管理器。Go + React 单仓库。
---

# AGENTS.md — Bonds

个人关系管理器。Go + React 单仓库。

## 构建 / 测试命令

```bash
# --- Go 后端（工作目录：server/）---
cd server
go build ./...                            # 编译所有包
go test ./... -v -count=1                 # 运行所有后端测试
go test ./internal/services -run TestCreateNote -v -count=1  # 运行单个测试
go test ./internal/handlers -v -count=1   # 仅运行 handler 集成测试
go vet ./...                              # 静态检查

# --- React 前端（工作目录：web/）---
cd web
bun run build                             # 类型检查 (tsc -b) + vite 构建
bun run test                              # vitest run（所有单元测试）
bun run test -- src/test/Login.test.tsx    # 运行单个测试文件
bun run lint                              # eslint

# --- E2E 测试（工作目录：web/）---
cd web && bunx playwright test            # 运行所有 e2e 用例（自动启动 server + vite）
bunx playwright test e2e/auth.spec.ts     # 运行单个 e2e 文件

# --- Makefile 快捷方式（从项目根目录）---
make test                                 # 后端 + 前端测试
make test-server / make test-web / make test-e2e
make build                                # 后端 + 前端分别构建
make build-all                            # 构建内嵌前端的单二进制文件
make dev                                  # 开发模式同时启动前后端
make swagger                              # 生成 Swagger 文档（swag init）
make gen-api                              # swagger + 生成前端 TypeScript API client
make setup                                # 安装依赖（go mod download + bun install）
```

**Go 代理（中国网络必须）：** 始终使用 `GOPROXY=https://goproxy.cn,direct` 执行 `go mod download`。

**包管理器：** 使用 `bun`，禁止 `npm` 或 `yarn`。

### 代码生成管线

前端 TypeScript API 客户端从后端 OpenAPI/Swagger 规范**自动生成**，生成的文件不纳入 git 版本控制。

```
Go handlers (swag 注解) → make swagger → server/docs/swagger.json
                        → make gen-api → web/src/api/generated/  (gitignored)
                                       → web/src/api/index.ts    (入口，引用 generated/)
```

- **修改后端 API 后**必须运行 `make gen-api` 重新生成前端客户端
- CI/Dockerfile 会在构建前自动生成，无需手动提交
- 生成工具：`swagger-typescript-api`（dev 依赖），配置在 `web/package.json` 的 `gen:api` 脚本
- **禁止手动修改 `web/src/api/generated/` 目录下的任何文件**

## 项目结构

```
server/                    # Go 后端（模块：github.com/naiba/bonds）
  cmd/server/main.go       # 入口 — Echo + GORM + Cron 初始化 + SPA 服务 + 信号优雅关闭
  internal/
    calendar/               # 多历法抽象：Converter 接口 + 注册表，gregorian.go（直通）、lunar.go（农历，6tail/lunar-go）
    config/                 # 基于环境变量的配置加载（含 SMTP/OAuth/Telegram/Geocoding/Bleve/WebAuthn）
    cron/                   # Cron 调度器（robfig/cron v3），支持数据库锁防重复执行
    database/               # GORM Connect + AutoMigrate
    dav/                    # CardDAV/CalDAV 服务器（emersion/go-webdav），Basic Auth + Backend 接口实现
    frontend/               # 内嵌前端静态文件（go:embed dist）
    i18n/                   # 国际化：embed 加载 en.json/zh.json，中间件解析 Accept-Language
    models/                 # 55 model 文件，registry.go 列出所有迁移模型
      seed.go               # 全局种子：SeedCurrencies（货币表）
      seed_account.go       # 账户级种子：SeedAccountDefaults（注册时调用）
      seed_vault.go         # Vault 级种子：SeedVaultDefaults（创建 vault 时调用）
    dto/                    # 请求/响应结构体（json + validate 标签）
    search/                 # 全文搜索引擎（Bleve v2），CJK 中文分词，Engine 接口 + NoopEngine
    services/               # 业务逻辑，每个领域一个文件
      calendar_convert.go   # 共享历法转换辅助函数 applyCalendarFields()
    handlers/               # HTTP 处理器（Echo），routes.go 统一注册路由
    middleware/              # JWT 认证、CORS、locale、vault 权限校验
    testutil/               # SetupTestDB（内存 SQLite）、TestJWTConfig
  pkg/
    avatar/                 # 头像生成：首字母 + 确定性颜色 → PNG（纯 stdlib image）
    response/               # API 响应封装：OK、Created、Paginated、各种错误

web/                       # React 前端（Vite + TypeScript）
  src/
    api/                    # API 客户端（自动生成）
      generated/            # swagger-typescript-api 生成的模块（gitignored，禁止手动修改）
      index.ts              # API 入口：实例化 HttpClient + 所有生成模块
    components/             # 共享组件（Layout.tsx、SearchBar.tsx、CalendarDatePicker.tsx）
    locales/                # 前端 i18n：en.json、zh.json（react-i18next）
    pages/                  # 按领域组织的路由页面（含 TwoFactor/Invitations/AcceptInvite/OAuthCallback）
    stores/                 # AuthProvider 上下文 + ThemeProvider（dark/light/system）
    types/                  # TypeScript 类型声明（仅 lunar-javascript.d.ts），DTO 类型统一从 @/api 导入
    utils/                  # 工具函数（calendar.ts — 前端多历法抽象 + 注册表）
    test/                   # Vitest 单元测试 + setup.ts
    i18n.ts                 # react-i18next 初始化 + 语言检测
  e2e/                      # Playwright 测试用例

Dockerfile                 # 多阶段构建：bun build → go embed → 单二进制
docker-compose.yml         # 单容器部署
.github/workflows/
  test.yml                 # CI：任何 push / PR 触发
  release.yml              # CD：test.yml 成功 + v* tag 时 workflow_run 触发
```

## Go 后端约定

### 架构：Handler → Service → DTO

每个功能遵循：**handler**（HTTP 层）→ **service**（业务逻辑）→ **dto**（请求/响应）→ **model**（GORM）。

- Handler 绑定请求、校验、委托给 Service，通过 `response.*` 辅助函数返回。
- Service 接收 DTO、返回 DTO。持有 `*gorm.DB`，负责所有查询逻辑。
- Model 是纯 GORM 结构体，不含业务逻辑。

### ID 类型

- `Account`、`User`、`Vault`、`Contact` 使用 **UUID string** 主键（`gorm:"primaryKey;type:text"` + `BeforeCreate` 钩子）。
- 其余所有模型使用 **自增 uint**（`gorm:"primaryKey;autoIncrement"`）。

### 错误处理

- Service 定义哨兵错误：`var ErrNoteNotFound = errors.New("note not found")`
- Handler 通过 `errors.Is(err, services.ErrXxxNotFound)` 判断 → `response.NotFound(c, "...")`
- 通用错误 → `response.InternalError(c, "...")`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [naiba/bonds](https://github.com/naiba/bonds) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
