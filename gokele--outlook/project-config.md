---
trigger: always_on
description: 面向已开通 POP3/IMAP 的 Outlook 账号池，提供按需在线取件、令牌自动轮换、分类备注与开放 API 的网页系统。
---

# Outlook 取件台

面向已开通 POP3/IMAP 的 Outlook 账号池，提供按需在线取件、令牌自动轮换、分类备注与开放 API 的网页系统。

接口契约见 [API.md](API.md)。改动任何接口的入参、响应字段或枚举取值时，必须同步更新它。

## 两条不可动摇的约束

1. **邮件不落库。** 没有 `messages` 表，没有 `fetch_cursors` 表，没有缓存层。邮件只在单次请求的内存中存在，响应写出后即释放。取件日志只记条数与结果，不记主题、发件人与正文，否则等于变相存了邮件。任何"加个邮件缓存表"的改动都违背这条。

2. **取 access_token 与轮换 refresh_token 是两件事。** 由请求 scope 是否含 `offline_access` 决定。微软只在收到该值时才返回新的 refresh_token；不含时只返回 access_token，原 refresh_token 保持有效且不变。三档策略见 `internal/tokensvc`。

## 目录

```
backend/          Go API 服务
  main.go         入口，装配依赖并启动 HTTP 与调度器
  internal/
    model/        跨层共享类型。时间一律 Unix 秒
    config/       环境变量装载
    crypto/       AES-256-GCM 令牌加密、API Key 哈希、登录密码派生
    store/        唯一的持久化层，一套 SQL 跑两种数据库
    oauth/        令牌端点客户端与 AADSTS 错误分类
    tokensvc/     三档取令牌策略与 client_id 熔断
    fetcher/      三条取件通道，无状态接口
    orchestrator/ 取件编排：通道降级、并发合并、长轮询、验证码提取
    scheduler/    常驻轮换调度器，到期时间驱动
    importer/     批量导入与三层去重
    httpapi/      后台与开放 API 的 HTTP 处理
    updater/      从 GitHub Releases 拉新版、校验散列、替换二进制与重启
  web/            前端产物的 go:embed 封装与单页应用静态服务
frontend/         React 19 + Vite + TanStack Router/Query + Ant Design 5
.github/workflows/ Release：推标签即交叉编译四平台并发布
```

进程守护与反代不进版本库：与具体发行版、反代软件和证书方案强耦合，
给模板多半还是要改。

## 偏离默认规范的地方

按全局规范需要记录原因与影响范围：

- **未使用 sqlc，改为手写 SQL 加 `database/sql`。** 原因：sqlc 需要按引擎各生成一套代码，两套实现会随时间发散。现在一套 SQL 同时跑 PostgreSQL 与 SQLite，差异只有取任务的加锁子句一处（`store.forUpdateSkipLocked`）与占位符改写（`store.rebind`）。影响范围：`internal/store` 内部，上层不感知。
- **IMAP 与 POP3 用标准库手写协议交互，未引入 emersion/go-imap。** 原因：避免第三方库的版本与行为风险，协议交互本身不复杂。影响范围：`internal/fetcher`。
- **未使用容器。** 生产直接跑一个二进制加 PostgreSQL。影响范围：部署配置，尚未提交。
- **前端嵌进后端二进制**（`web` 包的 `go:embed`），不由 Nginx 托管静态文件。原因：前后端版本天然绑定，不会出现前端已更新而后端是旧版、接口对不上的情况；部署也退化成拷一个文件。代价是二进制从 16 MB 涨到 25 MB，且改前端也要重新编译后端。影响范围：`web/`、`internal/httpapi/server.go` 的兜底路由、构建流程。
- **前端 UI 库锁定 Ant Design 5**，符合 `/enterprise-ui` 规范的默认选型（React 中后台首选）。曾用 shadcn/ui + Tailwind 重写过一版，因为需要自己做全部视觉决策（配色、圆角、密度、卡片风格逐项反复确认）而回退。**结论：不要再换库。** antd 自带成熟默认视觉，`ConfigProvider` 的 Design Token 三层结构足以承载品牌色定制；真要提升观感，优先做两件事——配 `colorPrimary` 主色、引入 ProComponents（ProTable 内置筛选栏/列设置/密度切换/批量操作）。
  - Arco Design 评估过并搭过并排 demo：纯 React 单栈用不上它的双栈优势，且在 React 19 下触发 `element.ref was removed` 警告，兼容性需另行评估。TDesign 定位多端统一，本项目无此需求。
  - React 19 兼容依赖 `@ant-design/v5-patch-for-react-19`，不可移除。

## 开发

```bash
cd backend
cp ../env.example .env      # 首次: 本地配置, 已被 gitignore
go run .                    # 默认用 ./data/app.db，首次启动会打印随机管理员密码
go test ./...

cd frontend
npm install && npm run dev  # 代理 /api 到 127.0.0.1:8080
```

开发用 SQLite，生产用 PostgreSQL。这个差异的代价是并发与锁的语义在 SQLite 上无法验证：
`FOR UPDATE SKIP LOCKED` 在 SQLite 上根本不存在，调度器抢任务的竞争在它上面永远不会真正发生。
因此**上线前必须在 PostgreSQL 上跑一遍，只跑 SQLite 的测试通过不算通过**：

```bash
TEST_DATABASE_URL=postgres://user:pass@127.0.0.1:5432/dbname go test -count=1 ./...
```

CI 会在真实 PostgreSQL 上跑一遍（见 `.github/workflows/test.yml`，
带 `postgres:17` service，SQLite 与 PostgreSQL 各跑一趟），
因此推上去就能知道结果；本地改数据库相关代码时仍建议自己先跑一遍。

改前端版式后跑响应式检查。它在 12 个视口 × 全部 12 条路由上做机器判定
（整页横向滚动、元素越界、触摸目标过小、文字裁切），再加连续缩放采样、
弹窗抽屉专项、200% 缩放、键盘走查与手机端完整流程：

```bash
cd frontend
ADMIN_PASS=xxx npm run check:empty        # 空态检查，必须在灌数据之前
ADMIN_PASS=xxx npm run seed:responsive    # 灌一批含极端长度的数据
ADMIN_PASS=xxx npm run check:responsive   # 完整跑约 5 分钟
QUICK=1 ADMIN_PASS=xxx npm run check:responsive   # 改代码时用这个，约 100 秒
```

**`check:empty` 的顺序不能调。** 一旦灌了数据，空态就再也回不来了 ——
而 `check:responsive` 要求库里有数据（空表照不出任何版式问题），
于是它永远看不到空态。空态恰恰是新用户看到的第一屏：装完、登录、落在总览，
那一刻整个产品只有空态。它除了查版式，还查总览与账号列表在空着时
**有没有一条通往别处的链接** —— 判据不能写成"有没有可点的东西"，
PageContainer 的「刷新」按钮每一页都有，那样这条检查永远通过。

它用系统已装的 Chrome（`CHROME_PATH` 可指定），不下载 playwright 自带的浏览器。
CI 里作为独立任务跑在真实产物上（前端嵌进 Go 二进制、单端口托管），
失败时截图会作为 artifact 上传。

## 关键设计点，改代码前先读

- **三档取令牌**（`internal/tokensvc/tokensvc.go`）：命中缓存零请求；access_token 过期但距上次轮换不足 60 天时只换 access_token，不带 `offline_access`，不写 accounts 行；首次验证或满 60 天才轮换。双重检查锁保证并发只轮换一次。
- **错误分类**（`internal/oauth/oauth.go`）：只有 `invalid_grant` 与需要交互授权的错误会把账号置为失效。网络类与限流类**绝不改状态**，否则微软侧一次抖动会批量误杀账号。判定读 `error_codes` 数组里的 AADSTS 数字码，不读 `error_description`。
- **client_id 熔断**（`tokensvc.checkAndSuspend`）：先于账号状态判定生效。几千个账号常共用少数 client_id，应用被封时逐个标失效会造成大规模误判。
- **调度速率由积压推导**（`internal/scheduler/scheduler.go`）：不设每日配额。配额要人工从账号数反推，账号增长后会静默失效。速率上限取单 IP、单 client_id、全局并发三者最小值。
- **速率上限本身也自动推导，但只往下调**（`internal/scheduler/autorate.go`）：单 IP、单 client_id 两条上限原来是手填的，十万账号和十亿账号需要的速率差四个数量级，没人能凭直觉估准。自适应按"账号数 ÷ 轮换阈值"算出稳态需求，留 1.5 倍余量后分摊到各出口与各应用。**需求高于安全上限时速率停在上限**（单 IP 30/分钟、单 client_id 20/分钟），并报出还缺多少出口与应用注册——照着需求把速率调上去不是提高吞吐，是送去封号，而且封的是整批账号赖以存活的应用注册。推导结果缓存 5 分钟且计数带超时：`COUNT(*)` 在十亿行上是分钟级全表扫描，每个 tick 数一次会把调度器卡死在计数上。
- **来源 IP 只在连接来自可信反代时才采信请求头**（`internal/httpapi/clientip.go`）：`X-Forwarded-For` 与 `X-Real-IP` 是请求方写的，无条件相信它们等于让人自己声明来源 IP——而 API Key 的 IP 白名单与登录限速都以来源 IP 为判据，于是两个安全控制一起失效。判可信必须看 `r.RemoteAddr`（内核填的，伪造不了）。链里取**最右**那个非反代地址：nginx 的 `proxy_add_x_forwarded_for` 是把真实客户端追加到客户端自带的值后面，取最左等于专门去读攻击者写的内容。chi 的 `middleware.RealIP` 已被官方标记 Deprecated（三个 CVE），不要用；它的替代 `ClientIPFromXFF` 也不看 `RemoteAddr`，把「只有反代能连到本服务」交给防火墙保证，那个前提在自建部署里常常不成立。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gokele/Outlook](https://github.com/gokele/Outlook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-17 -->
