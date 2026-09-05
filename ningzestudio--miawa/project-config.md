---
trigger: always_on
description: 面向 Minecraft 启动器分发场景的 GitHub Release 镜像服务（Go 单体 + 内嵌前端）。入口 `cmd/mirror/main.go`，后端全部在 `internal/`。
---

# lemwood-mirror 项目记忆

面向 Minecraft 启动器分发场景的 GitHub Release 镜像服务（Go 单体 + 内嵌前端）。入口 `cmd/mirror/main.go`，后端全部在 `internal/`。

## 构建与测试

- Go 1.25。测试/构建：`go test -count=1 ./...`、`go vet ./...`。CI 用 `go test ./... -count=1 -timeout 120s`。
- Termux 环境 go 不在默认 PATH，位于 `/data/data/com.termux/files/usr/lib/go/bin`（apt 的 golang 包），用前先 `export PATH=$PREFIX/lib/go/bin:$PATH`。
- 构建：`go build -o mirror ./cmd/mirror`；开发运行：`go run ./cmd/mirror`。
- 纯 Go 构建，`CGO_ENABLED=0`：SQLite 驱动是 `modernc.org/sqlite`（pure-Go），**不需要 cgo**，不要为 SQLite 切换 `mattn/go-sqlite3`。
- 版本号靠 `-ldflags "-s -w -X main.Version=..."` 注入：CI tag 构建注入 `github.ref_name`；Docker 需 `--build-arg VERSION=x.y.z`（Dockerfile ARG VERSION）；dev 构建为 `"dev"`，自更新检查时视为"有更新"。
- **构建产物不入库（2026-08-15 起）**：`mirror-linux-amd64` 曾误入库（66MB）已 `git rm --cached` 移除，.gitignore 含 `mirror-linux-amd64`。本地 `go build` 的二进制只用于本地验证/部署，**不要 `git add` 任何二进制**；正式产物由 CI 按 tag 构建发布。
- 前端构建：`cd frontend && pnpm build`，产物输出到 `web/default/`（git 跟踪的内嵌资源，改完 frontend/src 必须重新构建否则线上不生效）。
- 管理端构建：`cd admin-app && pnpm build`，产物输出到 `web/admin/`（同样被 git 跟踪，会被构建重写 hash 文件名）。
- CI（`.github/workflows/build.yml`）先单独执行 Go test/vet 和前端检查/构建，再矩阵构建 windows/linux × amd64/arm64/x86；**仅 tag 推送时由单独 job 发 Release**（softprops/action-gh-release）。pnpm 版本固定 10.12.4。

## 后端架构（internal/ 包职责）

- `cmd/mirror/main.go` — 唯一入口。启动顺序：释放内嵌前端 → 加载配置 → InitDB → 流量 tracker → stats 写池（4 worker + 1000 缓冲）→ server.State → scanner（cron）→ selfupdate manager → cron 调度。`Scanner.scanLauncher` 是同步主循环，`ScanAll` 用 `scanMu.TryLock` 防重入。
- `internal/server/` — HTTP 路由 + SPA 托管 + 下载处理器。`server.go`（41KB，单体）是改动热点；`v2.go` 是公开 API v2 handler；`spafallback.go` SPA 回退；`http.go`/`utils.go` 小工具。**下载处理器是流量计数的唯一计数点**（见下"流量统计双口径"）。
- `internal/db/` — 数据库抽象（SQLite/MySQL/PostgreSQL），见下"数据库"。
- `internal/config/` — 配置加载/保存/迁移，见下"配置"。
- `internal/stats/` — 访问/下载统计，异步写池 + 快照预热。
- `internal/traffic/` — 单 IP 每日流量上限（防刷墙），预检 + 实际传输记账。
- `internal/github/` — go-github v50 封装，见下"GitHub 客户端"。
- `internal/selfupdate/` — GitHub Release 自更新，见下"自更新"。
- `internal/geoip/` — ip2region v4+v6 xdb 内嵌，本地查属地。
- `internal/version/` — 自研 SemVer-like 比较，被 launcher 索引和 selfupdate 共用。
- `internal/downloader/` — 版本索引生成与资产下载。
- `internal/download_authz/` — DB 授权状态表（`download_authorizations`）：43 字符 base64url token，DB 存 sha256 hash；Issue/Peek/Consume（atomic 单次消费）。见下"PoW 下载验证"。
- `internal/auth/` — 管理员认证 + TOTP，`CleanupTokens()` 后台清理。
- `internal/assets/` — 启动时把内嵌前端释放到项目根目录。
- `internal/firewall/` — 站点级防火墙（见下「防火墙」）。
- `internal/blacklist/` / `internal/netutil/`（客户端 IP 解析 + `ParseEntry`/`ValidEntry` IP/CIDR 解析）/ `internal/storage/` — 各自独立小包。

## 数据库

- 全局 `db.DB *sql.DB` + `db.isMySQL` / `db.isPostgres` 标志。**SQLite 默认**（`<storage>/stats.db`），MySQL 可选（`mysql_host` 非空即启用），PostgreSQL 可选（`database_mode=pgsql`，或 `auto` 且 `postgres_host` 非空）。
- SQLite：`SetMaxOpenConns(1)`（单写连接，避免 SQLite 锁冲突），开 WAL + `busy_timeout=10000`。MySQL：连接池 25/10，`ConnMaxLifetime=1h`。PG：pgx v5（`rebind()` 把 `?` 转 `$n`），连接池 2/1。**注意**：单连接使 SQLite 的 WAL"读不阻塞写"失效，热路径 DB 操作全部串行——已知取舍。
- **迁移系统**在 `internal/db/migrations.go`：`CurrentSchemaVersion` 常量（当前 = 6；v5 加 visit_count/event_count 聚合列，v6 加 aggregate_key 聚合键 + 唯一索引）。新增迁移 = 往 `migrations` 切片末尾追加一项 + 递增该常量。每个 `Up` **必须幂等且三方言兼容**（MySQL 用 INSERT IGNORE、PG 用 ON CONFLICT DO NOTHING、SQLite 用 INSERT OR IGNORE）。`system_info` 表的 `schema_version` 是提交点，每个迁移成功后立即写入。MySQL 用 `ON DUPLICATE KEY UPDATE`，PG 用 `ON CONFLICT ("key") DO UPDATE`，SQLite 用 `INSERT OR REPLACE` 写 schema_version。v6 的回填合并+建唯一索引必须在同一事务内（崩溃留下重复 key 会让 CREATE UNIQUE INDEX 永久失败无法自愈）。
- `mysql_migration: true` 且存在 `stats.db` → 一次性 SQLite→MySQL 迁移，旧库改名为 `.bak`。
- 表清单：`visits, downloads, ip_blacklist, ip_daily_traffic, daily_traffic, daily_completed_traffic, download_authorizations, download_events, system_info`。建表在 `createTables()`，按 `isMySQL` 分支用各自方言。
- 注：repo 镜像功能已移除，`repo_*` 表不再迁移；`launcher.mode` 的 `clone`/`all` 已废弃（Git 镜像功能已移除），仅为兼容旧配置保留，`ShouldSyncRelease` 对 `release`/`all` 返回 true、对 `clone` 返回 false。

## 配置

- `config.yaml` 和旧 `config.json` **都在 .gitignore（含 secrets，绝不提交）**。仓库里 `internal/config/default.yaml` 是提交的嵌入式默认模板（经 `embedded.go` 内嵌）。
- `LoadConfig` 行为：无 yaml 但存在旧 `config.json` → 自动迁移到 yaml 并删除旧 json；两者都没有 → 释放内嵌 `default.yaml` 写盘。
- **后台保存会从 `defaultConfigTemplate`（text/template）整体重写 config.yaml**，不要指望手写在 yaml 里加的自定义注释能在后台保存后保留。
- `GITHUB_TOKEN` 环境变量覆盖 yaml 里的 `github_token`。
- `NormalizeConfig` 不变量：`traffic_limit_gb < 0` → 5；`max_versions ≤ 0` → 3；`admin_enabled` 但 user/password 空 → 自动禁用后台；`check_cron` 空 → `*/10 * * * *`。
- 密码用 bcrypt 哈希：`htpasswd -bnBC 14 "" <password> | tr -d ':\n'`。

## 本地数据库环境（2026-09-01 搭建，供迁移集成测试）

- 已安装 PostgreSQL 18.6 与 MySQL 8.4.11（Ubuntu apt）。**端口按仓库集成测试约定**：PG = 127.0.0.1:55432，MySQL = 127.0.0.1:33306（PG 改 `port` 于 `/etc/postgresql/18/main/postgresql.conf`；MySQL 改 `/etc/mysql/mysql.conf.d/mysqld.cnf` 追加 `port = 33306` 与 `mysql_native_password=ON`，8.4 默认禁用该插件而 go-sql-driver DSN 未带 allowPublicKeyRetrieval，必须开）。
- 凭据：PG 用户 `lemwood`（superuser，pg_hba 对 127.0.0.1/::1 为 trust，免密）；MySQL 用户 `lemwood`/`testpass`（mysql_native_password，GRANT ALL ON *.*，仅回环）。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NingZeStudio/miawa](https://github.com/NingZeStudio/miawa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-05 -->
