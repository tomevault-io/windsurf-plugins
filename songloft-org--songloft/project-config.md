---
trigger: always_on
description: 本文件为 AI 编程助手提供 Songloft 项目的**入口信息**：项目结构、常用命令、铁律与踩坑总结。代码本身就是真实来源的内容（目录树、依赖、API 表、表结构）请直接看代码或下方链接的详细文档。
---

# AGENTS.md

本文件为 AI 编程助手提供 Songloft 项目的**入口信息**：项目结构、常用命令、铁律与踩坑总结。代码本身就是真实来源的内容（目录树、依赖、API 表、表结构）请直接看代码或下方链接的详细文档。

> **详细文档**：
> - 架构：[整体](docs/architecture.md) · [后端](docs/architecture_backend.md) · [前端](docs/architecture_frontend.md)
> - 专题：[数据库操作](docs/database_migrations.md) · [颜色系统](docs/color_system.md) · [API 响应格式](docs/api_response.md) · [快速上手](docs/quick-start.md)
> - 插件开发：见 `plugin-toolchain/README.md`（独立仓库）
> - 插件源制作：[插件源制作指南](docs/plugin_registry.md)
> - API：开发模式启动后访问 `/swagger/index.html`

---

## 项目概述

Songloft 是自托管本地音乐服务器，多仓库结构：

| 目录 | 技术 | 说明 |
|------|------|------|
| `/` | Go 1.26 + Chi v5 + SQLite | 后端 API 服务（默认端口 58091，账号 admin/admin） |
| `/songloft-player` ([独立仓库](https://github.com/songloft-org/songloft-player)) | Flutter 3.29+ / Dart 3.7+ | 跨平台前端（6 平台） |
| `/plugin-toolchain` ([独立仓库](https://github.com/songloft-org/plugin-toolchain)) | TS + pnpm | JS 插件开发工具链（SDK / Builder / 脚手架） |
| `/jsplugins-src` | TS | JS 插件源码（子模块集合，每个插件在自己仓库下分发 release） |
| `/pkg/tag` | Go | 音频元数据**读写**库（基于上游 tag 库扩展 MP3/FLAC 写入） |

---

## 常用命令

```bash
# 后端
make run            # 启动（dev 模式，含 Swagger）
make build          # 编译开发版（完整版，嵌入前端）
make build-lite     # 编译开发版（精简版，不嵌入前端）
make build-prod     # 编译生产版（完整版，嵌入前端）
make build-prod-lite # 编译生产版（精简版，不含前端）
make test           # 测试
make check          # fmt + vet + test
make sqlc           # 重新生成 sqlc 代码（改了 queries/*.sql 后必跑）
make swagger        # 重新生成 API 文档

# 前端构建（产物落到 songloft-player-build/，供后端嵌入或独立部署）
make build-frontend-web-embedded   # 嵌入 Go 二进制用（隐藏 API 地址 UI）
make build-frontend-web            # 独立部署 web
make build-frontend-{linux,windows,macos,android,ios,all}

# 前端开发
cd songloft-player && flutter run -d chrome          # standalone
cd songloft-player && flutter run -d chrome --dart-define=DEPLOY_MODE=embedded
```

---

## 数据库规范（铁律）

> 完整操作步骤见 [docs/database_migrations.md](docs/database_migrations.md)。

访问栈：**goose 迁移 + sqlc 固定 SQL + squirrel 动态 SQL + Repository + UnitOfWork**。

- **改 schema** → `internal/database/migrations/000N_xxx.sql`，启动时 `goose.Up` 自动执行；**禁止**手动 `ALTER data/songloft.db`
- **加固定 SQL** → `database/queries/{table}.sql` + `make sqlc`；生成产物 `database/sqlc/` 必须入库
- **动态 SQL（变长 WHERE/SET）** → 在 `*_repository.go` 内用 squirrel，禁止拼字符串
- **跨表写** → `db.RunInTx(ctx, func(ctx, uow))` 拿同一 `*sql.Tx` 下的 `uow.Songs/Playlists/...`；**禁止** service 层手 `BeginTx`，否则会 SQLITE_BUSY
- **错误语义** → 仓储未命中统一 `database.ErrNotFound`；service 用 `errors.Is` 判别
- **测试** → `testutil.OpenMemoryDB(t)` 跑真实 `:memory:` + 真实 Repository；**禁止**手写 mockDB
- **内置数据** → 迁移预置歌单 id=1「收藏」、id=2「电台收藏」（`labels=["built_in"]`），及 `music_path / jwt_secret / source_*` 默认 config。测试行数断言记得扣掉

---

## 后端编码约定

- 标准 Go layout（`internal/` 防外部依赖），Chi v5 路由，JWT 双 Token
- 依赖注入：service 层只接收 Repository 接口，**不接收** `DB`
- 日志：标准库 `slog`；HTTP 错误：统一 `respondError`
- **API 响应格式**：RESTful 直返，**禁止** `{code, data, message}` 信封；错误统一 `{"error","detail"}`。完整规范见 [docs/api_response.md](docs/api_response.md)
- 不用 ORM：固定 SQL → sqlc，动态 SQL → squirrel，跨表写 → `RunInTx + UnitOfWork`
- 测试文件 `*_test.go` 与源码同目录

---

## API 文档规范（铁律）

**所有在 `internal/app/routers.go`（含 `RegisterStaticRoutes` / `RegisterAPIRoutes` 等子注册函数）里注册的 handler 方法，必须有 swag 注释**。后端 API 文档由 [swaggo/swag](https://github.com/swaggo/swag) 从注释生成，是前端开发与外部集成的唯一来源。

### 必填字段（每个 handler 至少有这 7 项）

```go
// @Summary <一行中文摘要>
// @Description <详细描述，可多行；说清楚副作用 / 默认值 / 错误码触发条件>
// @Tags <业务分组，中文>
// @Produce json
// @Success 200 {object} <返回类型> "<说明>"
// @Security BearerAuth
// @Router /<path> [<method>]
func (h *XxxHandler) Method(w http.ResponseWriter, r *http.Request) { ... }
```

- 有请求体的接口额外加 `@Accept json` 和 `@Param request body <type> true "<说明>"`
- 错误路径明显的接口加 `@Failure 400/404/500 {object} map[string]string "..."`
- 路径参数 / 查询参数用 `@Param <name> path/query <type> true/false "<说明>"`
- **公开端点**（无需 token，如健康检查）省略 `@Security BearerAuth`
- **业务 tag 命名**：复用现有 tag（「歌曲管理」「歌单管理」「电台与 HLS」「扫描管理」「配置管理」「缓存管理」「JS 插件」「数据备份」「设置」「升级」「认证」），不要随手造新 tag

### 多别名 / catch-all 路由

- 一个 handler 注册了多条 alias 路径（如 `/songs/{id}/play` 与 `/songs/{id}/play.m3u8`）→ 每条 alias 单写一行 `@Router`
- HEAD 是 GET 的子集，**不单独列**；OpenAPI 不强制
- `r.HandleFunc(...)` 这种接受 ANY HTTP 方法的 catch-all → 列出所有实际可能的方法（`[get] [post] [put] [delete]`），每个一行 `@Router`
- 动态路径（`{entryPath}` 由运行时按已安装插件决定的）→ 在 `@Description` 里注明「动态路由，{xxx} 由运行时决定，OpenAPI 仅作占位」

### 改完必跑

修改 / 新增 handler 注释后必须跑 `make swagger`：会重新生成 `docs/swagger.json`、`docs/swagger.yaml`、`docs/docs.go`，**这些产物必须入库**。否则 `/swagger/index.html` 与代码不同步，前端按旧文档对接会踩坑。

### 验证

- `make swagger` 输出里搜索新加的 `@Router` 路径，确认 `Generating <Type>` 包含你新写的请求/响应类型
- `grep '<your-new-path>' docs/swagger.json` 应有命中
- 启动 `make run`，访问 `http://localhost:58091/swagger/index.html` 在 UI 里点开新端点目测

### 没有豁免

「凡 routers 注册即必注释」是绝对规则。哪怕是动态路由 catch-all、静态资源 handler、反代端点，也要写 swag——`@Description` 里把"它是什么、为什么 OpenAPI schema 不精确"说清楚即可。

---

## 配置接口规范（铁律）

项目里有两类配置接口，**用户可见的功能开关一律走业务端点**，通用 KV 仅作 admin 入口。

### `/api/v1/settings/<name>` — 孤立配置端点（前端业务功能默认走这里）

- 路径风格：`/settings/<kebab-case-name>`（如 `/settings/hls-proxy`、`/settings/music-path`、`/settings/http-proxy`）
- 数据形态：**强类型** JSON（如 `{enabled: bool}` 或聚合对象），不是 `{value: string}`
- 默认值：handler 内部承担（配置缺失时 GET 返回业务默认，PUT 时直接写入即可，**前端无需先 POST 创建**）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [songloft-org/songloft](https://github.com/songloft-org/songloft) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
