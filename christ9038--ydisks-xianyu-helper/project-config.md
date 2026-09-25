---
trigger: always_on
description: The active application contains:
---

# AGENTS.md

## Repository shape

The active application contains:

- `cmd/server/` — server entrypoint, administrator bootstrap (`/initialize` and `-init-admin`), and HTTP server.
- `cmd/init-admin/` — interactive administrator initialization CLI.
- `cmd/dbverify/` — migration + core CRUD verification tool across SQLite/MySQL/Postgres.
- `internal/server/` — chi HTTP API and SPA serving.
- `internal/adapter/` — wiring layer that implements `engine.Handler` and `automation.OrderDetailFetcher` (system events → automation center, order-detail fetch → browser, protocol-level credential renewal, account alerts → notifier).
- `internal/account/` — enabled-account supervisor.
- `internal/engine/` — per-account runtime, replies, and delivery behavior.
- `internal/automation/` — unified automation center (paid delivery, review gifts, review requests) + scheduler.
- `internal/xianyu/` — MTOP, WebSocket, QR login, and protocol code.
- `internal/browser/` — in-process Chromium automation through playwright-go.
- `internal/db/` — multi-database access (SQLite/MySQL/Postgres) with embedded Goose migrations per dialect.
- `frontend/` — active React/Vite source.
- `internal/webui/static/` — embedded frontend build output.

## Common commands

```bash
cd /Users/christ/Workspace/git/xianyu/Ydisks-Xianyu-Helper

make build      # go build ./cmd/server
make test       # go test ./...
make test-server       # go test ./internal/server
make test-server-race  # server 生命周期与凭证并发 smoke race
make vet        # go vet ./...
make lint       # golangci-lint run ./... (0 issues baseline)
make check      # fmt + architecture + API contract + vet + lint + test + comments
make cover      # Go 全量覆盖率（默认不启动 Chromium）
make cover-browser # 本地 Chromium 页面与 CDP 覆盖率
make cover-frontend # 前端 V8 覆盖率（文本/JSON/HTML）
make frontend   # build frontend into internal/webui/static
```

Run the server (SQLite by default; MySQL/Postgres via `-db-url` or `DATABASE_URL`):

```bash
go run ./cmd/server -db data/xianyu_data.db -addr :59188
DATABASE_URL="mysql://user:pass@tcp(host:3306)/db" go run ./cmd/server -addr :59188
```

On a new database, open the management page after starting the server. The first-run page accepts
and confirms an administrator password, creates the `admin` user, and signs the user in automatically.
The CLI bootstrap remains available for headless or operational environments.

Disable browser automation only when the user explicitly requests it or explicitly confirms that Chromium is unavailable. Unless the user gives that direction, agents MUST NOT add `-no-browser` when starting the server:

```bash
go run ./cmd/server -db data/xianyu_data.db -addr :59188 -no-browser
```

Initialize or reset the administrator:

```bash
go run ./cmd/server -init-admin -db data/xianyu_data.db -admin-password '...'
```

Verify a database (migration + CRUD across dialects):

```bash
go run ./cmd/dbverify "mysql://user:pass@tcp(host:3306)/db"
```

Run a focused test:

```bash
go test ./internal/server -run TestName -v
go test ./internal/db -run TestMigrate -v
```

Cross-database regression (requires Docker containers or external DBs):

```bash
TEST_MYSQL_URL="mysql://root:pass@tcp(host:3306)/db" \
TEST_POSTGRES_URL="postgres://user:pass@host:5432/db" \
go test ./internal/db -run TestMultiDB -v
```

Build the frontend:

```bash
cd /Users/christ/Workspace/git/xianyu/Ydisks-Xianyu-Helper/frontend
npm install
npm run build
```

Run the frontend development server:

```bash
npm run dev
```

Vite proxies backend routes to `localhost:59188`. Production builds are written to `internal/webui/static/` and embedded by the Go server.

## 闲鱼商品列表协议约束

闲鱼商品列表接口在账号没有在售商品时，成功响应可能省略 `cardList` 字段；**缺少 `cardList` 不是异常，也不是不完整响应**，必须按空商品列表处理并允许全量同步正常完成。本协议事实不得在代码审查、测试或后续重构中被改写为“缺少 `cardList` 即报错”。只有平台明确返回 `cardList` 且其值类型不符合接口协议，或其他字段明确表明响应失败时，才按异常处理。涉及该字段的测试必须覆盖成功且省略 `cardList` 的空列表场景。

## 砍价免拼与自动发货顺序（不可变业务规则）

所有自动化交易流程以 WebSocket 系统消息驱动；按订单状态扫描的调度器只可作为丢失消息后的兜底，绝不能抢跑或推断 WebSocket 阶段。砍价订单必须严格按以下顺序执行：

1. 收到“我已小刀，待刀成”时，只读取账号管理中的独立“自动免拼”开关；开关开启才调用免拼接口。此阶段严禁发卡、发送发货模板或确认发货。
2. 收到“我已成功小刀，待发货”时，才进入付款后自动发货规则并执行发卡或发货模板。自动免拼开关关闭不阻止已经到达该阶段的订单发卡。
3. 发卡或发货模板成功后，才可按账号“自动确认发货”开关调用普通确认发货接口。发卡是自动确认发货前的最后一步；免拼不是确认发货的替代分支，确认发货不得因为订单是砍价订单而改调免拼接口。

兜底任务只能根据订单已持久化的阶段事实恢复尚未完成的动作：普通待发货订单可补触发付款后发货；砍价订单只有在已记录免拼成功或已收到最终“待发货”阶段后才可补发卡及确认发货。兜底任务不得调用免拼接口，也不得抢在 WebSocket 阶段确认之前发卡、发货或触发其他自动化行为。

自动发货必须按以下四条可测试分支理解，不能把“商品类型”和“免拼阶段”混成同一个判断：

1. 常规发货：付款 WebSocket → 获取订单金额、数量和规格 → 匹配常规发卡/模板动作 → 发卡成功 → 按“自动确认发货”开关确认发货。
2. 多规格发货：付款 WebSocket → 获取订单完整 SKU 组合 → 只执行完整匹配该组合的发卡/模板动作 → 发卡成功 → 按“自动确认发货”开关确认发货；不得按部分规格或待获取金额猜测规则。
3. 二人小刀普通发货：未开启“自动免拼”时，不处理“待刀成”阶段；收到“我已成功小刀，待发货”后执行与常规发货相同的发卡和确认顺序。
4. 二人小刀免拼发货：开启“自动免拼”时，“待刀成” WebSocket 只执行一次免拼；必须等待“我已成功小刀，待发货” WebSocket 后才发卡，发卡成功后才允许确认发货。

历史失败运行的恢复也属于兜底，执行任何发卡、模板或确认发货前必须重新核对：运行最初来自 WebSocket 或合法待发货兜底、订单当前仍是 `pending_ship`、订单归属账号未变化、账号仍开启自动发货。订单已取消、完成或已发货时取消旧运行；订单事实缺失、快照身份不一致或外部结果不确定时停止自动重放并发送“需要人工处理”通知。所有进入 `needs_review` 或等价人工处理状态的自动化路径都必须发送该独立通知类别；原自动化类别订阅保持兼容接收。

## Mandatory refactoring governance — DO NOT SKIP

The repository is following the authoritative long-term plan in
`docs/architecture/refactoring-master-plan.md`. Before changing Go package boundaries, HTTP APIs, database
access, account credentials, application wiring, React page structure, tests, CI or compatibility behavior,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Christ9038/Ydisks-Xianyu-Helper](https://github.com/Christ9038/Ydisks-Xianyu-Helper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
