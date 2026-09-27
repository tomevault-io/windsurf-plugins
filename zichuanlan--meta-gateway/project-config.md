---
trigger: always_on
description: 本文件供 AI 编码代理（Codex / Claude / Cursor / WorkBuddy 等）阅读。目标是让你在**不读完全部源码**的情况下，也能安全、正确地改动这个仓库。
---

# AGENTS.md — Meta Gateway 仓库协作指南

本文件供 AI 编码代理（Codex / Claude / Cursor / WorkBuddy 等）阅读。目标是让你在**不读完全部源码**的情况下，也能安全、正确地改动这个仓库。

> 项目一句话：**Meta Gateway 是一个自托管的 AI 网关 + 管理控制台**。对下游暴露 OpenAI / Anthropic 兼容协议，对上游聚合多个渠道（站点）的账号与模型，做路由、故障转移、用量计费与审计。架构上坚持**透明 pass-through**：不改写请求语义、不注入提示词、不伪造响应。

---

## 1. 命令速查

### 后端（Go 1.26+）

```bash
go build ./...                 # 全量编译
go vet ./...                   # 静态检查
go test ./internal/...         # 后端测试（默认短模式）
go test ./internal/proxy/ -run TestXxx -v   # 单测定位
gofmt -l .                     # 格式化检查（CI 会卡）
go build -o bin/meta-gateway ./cmd/server   # 构建二进制
```

本地开发启动（最小必需环境变量）：

```bash
ADMIN_TOKEN=test MASTER_KEY=test-key-32-chars-long!!!!!!! METRICS_TOKEN=test \
  ./bin/meta-gateway
```

### 前端（Node 24+，位于 `web/`）

```bash
cd web
npm ci
npm run typecheck      # tsc -b --pretty false
npm test               # vitest（CI 下等价于 vitest run）
npm run build          # tsc -b && vite build
```

**`npm run build` 的产物写入 `internal/webui/dist`，由 `go:embed` 编译进二进制。**
改了 `web/src` 却没重建 dist，Go 侧跑的仍是旧 UI —— 这是本项目最高频的"改了没生效"原因。

### 完整交付前的质量门（全绿才算完成，少跑一项就可能被 CI 挡下）

```bash
cd web && npm run lint && npx tsc -b && npx vitest run && npx vite build && cd ..
gofmt -l . && go vet ./... && go build ./... && go test ./...
```

> **`npm run lint` 别省。** CI 的 Verify 步骤是 `npm run lint && npm run typecheck && npm test -- --run
> && npm run build`，一条 eslint **error**（例如测试文件里没被用到的 `within` 导入）就能让 CI 全红；
> 而 `release.yml` 的 `wait-for-ci` 会因此**拒绝发布**（tag 推上去了，镜像不会发）。2026-09-24 的
> v3.5.0 就是在推送前补跑 lint 时才拦下这条 —— 更早的清单里没有它，而 tsc / vitest 都不会报未使用的导入。

> Windows / 沙箱环境注意：`vite build` 默认 `emptyOutDir: true`，会先删掉旧的 `dist/`（含数十个
> 带 hash 的 chunk）。在带批量删除护栏的沙箱里，这一步有两种表现，**都是同一个根因、都要提权重跑**：
>
> 1. **报错中断**：`error during build:` 后面**没有任何错误详情**，`dist/assets` 被清空但新产物没写出来。
> 2. **静默挂死（更隐蔽，2026-09-24 实测）**：进程**永不退出、也不报任何错**，日志停在
>    `✓ 1834 modules transformed.` 之后再无下文，`rendering chunks` 永远不出现。护栏拦掉删除、
>    进程就在那里等。此时 `dist/` 里**仍是上一次构建的旧 chunk**（拿 `stat().st_mtime` 一看还是几十分钟前），
>    容易误判成"已经构建好了"。
>
> **判断口径**：健康构建只要 **~4s**（transform 阶段也就几秒）。如果 `transformed` 之后超过十几秒没动静，
> 就是被挂住了 —— 直接 kill 掉，改用 `dangerouslyDisableSandbox: true` 重跑。提权后实测 **4.03s** 完成。
> 别去动 `vite.config`（关 `emptyOutDir` 会留下陈旧 chunk，反而制造新的"改了没生效"）。
>
> 连带坑：挂死期间 `internal/webui/embed.go` 会锁住 `dist/assets`，此时跑 **任何** 依赖
> `internal/webui` 的 Go 测试（如 `go test ./internal/httpapi/`）都会报
> `pattern dist: open ...\dist\assets: The process cannot access the file because it is being used by
> another process.` —— 这是文件锁不是代码错，等构建结束后重跑即可。

### race 预算与测试后台资源（2026-09-20）

CI 的 race 步骤是 `go test -race -timeout 20m ./...`（**per-package** 20 分钟），不要调回默认的
10 分钟：`-race` 会给纯 Go 版 SQLite（`modernc.org/sqlite`）插桩，而每个开新库的测试都要重放全部
101 个迁移 —— 实测 **0.14s → 3.4s（25×）**。所以 `internal/store` 单包约 500s、`internal/httpapi`
约 700s，默认 600s 上限正好压在悬崖上（09-17 那次 httpapi 504.9s 惊险通过，之后 store 599.8s
只差 0.2s 就红）。race 步骤失败时先分清是 `DATA RACE` 还是 `test timed out`：后者是预算问题，
不是代码问题。想真正砍掉这笔开销，方向是「每包迁移一次模板库、各测试拷贝」，可省掉每测试的固定成本。

后台调度器（alert / balance / health sweep、alert rules、daily summary、model catalog、DB GC、
probe、update check、discovery recovery loop）都由 `NewWithDependencies` 启动，各自往
`RegisterStopper` 注册停止回调。**测试里建 router 必须用 `NewTestRouter(t, cfg, db, enc)`**
（`httpapi_test` 里写 `httpapi.NewTestRouter`），它在该测试结束时 `StopBackground`；直接用 `New`
会让调度器活到进程退出（实测 44 个泄漏 router ≈ 458 个常驻 goroutine）。
`internal/httpapi/background_test.go` 里有一个不变量测试 + TestMain 兜底守着这条约定。

---

## 2. 目录地图

```
cmd/server          # 进程入口：装配配置、store、各 service、HTTP 路由
cmd/e2e-runner      # 端到端跑测（配合 cmd/e2e-mock 上游桩）
internal/
  domain/           # 纯数据结构与归一化（models.go），无 IO
  store/            # SQLite 持久化；NNN_*.sql 为按序迁移脚本
  httpapi/          # 管理面 REST 路由与鉴权（/admin/*、/v1/* 入口）
  proxy/            # 转发核心：鉴权、路由选型、重试、计费、健康度
  routing/          # 路由 ↔ 成员关系、会话粘性
  adapters/         # 上游协议适配器（openai / anthropic / gemini / responses …）
  outbound/         # 出网策略（代理、超时、TLS）
  relay/            # 裸转发通道（尽量零加工）
  discovery/        # 上游模型发现与采纳（model_sync_mode: auto|manual）
  probe/            # 渠道健康探测与候选评估
  healthsweep/      # 健康度清扫
  usage/            # 用量与账单聚合
  financesweep/     # 余额/成本扫描
  ratelimit/        # 限流（注意 TRUSTED_PROXY_CIDRS 为空时 CDN IP 会糊在一起）
  account/          # 渠道账号同步桥
  checkin/          # 站点签到自动化
  auth/ totp/ crypto/   # 管理面鉴权、二次验证、MASTER_KEY 加解密
  backup/ exchange/ webdavsync/   # 备份与站点间迁移（AAH 兼容封套）
  alerts/ webhook/  # 告警与外发
  livetrace/        # 实时请求追踪
  observability/    # 指标与日志
  plugins/          # 插件市场与进程托管
  maintenance/      # GC 与数据清扫
  selfupdate/ updatecheck/   # 容器自更新
  config/ runtimeconfig/     # 配置读取与运行时设置
  webui/            # go:embed 前端产物（dist），不要手改
web/                # 前端源码（React 19 + Vite + TS）
docs/               # 设计文档
```

前端结构要点：
- `web/src/features/**` 按业务域分片；`web/src/lib/**` 通用工具。
- i18n 文案集中在 `zh.ts` / `en.ts`，有 `parity.test.ts` **强制双语键一一对应**，缺一个即红。

---

## 3. 铁律（改动前必读）

以下每一条都对应过真实事故，违反会静默出 bug（测试不一定能抓到）。

### 3.1 管理后台表单回填源

编辑类抽屉（如 `EditChannelDialog`）的回填数据来自 `GET /admin/channels/overview`，即
`store.ChannelStore.ListOverviews`。**凡该 SELECT 投影缺失的列，保存时都会以零值回写。**

曾导致 `model_sync_mode` / `max_reasoning_effort` / `payload_rules` / `max_concurrent` /
`proxy_url` 五列"设置了、重开就没了"。

> **给 channel 表加新列时，务必同时补 `ListOverviews` 的 SELECT + Scan**，否则前端的勾选、开关会静默失效。


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ZiChuanLan/meta-gateway](https://github.com/ZiChuanLan/meta-gateway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
