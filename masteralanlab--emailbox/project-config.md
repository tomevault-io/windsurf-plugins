---
trigger: always_on
description: 本文件是给在这个仓库里工作的人与 AI 代理看的约定。它只写**这个项目**的规则，
---

# AGENTS.md

本文件是给在这个仓库里工作的人与 AI 代理看的约定。它只写**这个项目**的规则，
通用的 Go / React 教程内容一律不收——那些查官方文档更准。

## 1. 这是什么项目

Emailbox：面向公众注册的多租户 SaaS，批量托管第三方邮箱账号（Outlook OAuth / Gmail /
QQ / 163 等），提供统一收信、令牌刷新、代理配置与批量运维。

两件事决定了这个项目的绝大多数取舍：

1. **它保管的是别人的邮箱凭据。** 一次越权、一次不留痕的导出、一次误发到别的租户，
   后果都是直接泄露用户资产。因此隔离测试、审计、加密不是可选项。
2. **它是多租户的。** 任何一条 SQL 都必须带 `tenant_id`，任何一个端点都要能回答
   「A 租户的用户拿 B 租户的 ID 来请求会怎样」——答案必须是 404。

完整设计见 [`docs/plan/`](docs/plan/README.md)，进度与踩过的坑见
[`docs/plan/PROGRESS.md`](docs/plan/PROGRESS.md)。**动手前先看 PROGRESS 的「过程中发现的坑」**，
那里记的每一条都是已经付出过代价的。

## 2. 核心原则

- **只做必要的事**：不引入当前功能不需要的依赖和抽象。依赖每多一个，供应链面就大一圈。
- **代码自解释，注释解释「为什么」**：命名和结构说明「做了什么」，注释留给
  「为什么这么做」「不这么做会怎样」。本仓库的注释密度偏高，且几乎都是后者，请保持这个风格。
- **失败要响亮**：解密失败、配置非法、通道构造失败，一律报错而不是回落到某个"安全"默认值。
  静默降级在这个项目里等于「用户以为走了代理，其实从服务器公网 IP 直连」。
- **一致性优于个性**：跟着周围代码的写法走。

## 3. 技术栈

**后端**：Go 1.25+、Echo v5、`database/sql` + sqlc（无 ORM）、`log/slog`、
SQLite（modernc 纯 Go 驱动）/ PostgreSQL 双引擎、bcrypt、AES-256-GCM、
go-imap/v2（锁定 beta 版本，封装在 `pkg/mailer/imapx` 内不外泄类型）。

**前端**：TypeScript、React 19、Vite、Tailwind CSS v4、
[Cloudflare Kumo](https://kumo.cloudflare.com)（`@cloudflare/kumo`）、
`@phosphor-icons/react`、Zustand v5、React Router v7、Axios、Vitest、Bun。

不用的东西也写清楚，免得反复提议：没有 ORM、没有 GraphQL、没有表单库、没有 i18n
（文案直接写中文）、没有 shadcn（已随 Kumo 接入删除）、没有 toast 库
（错误以内联红字展示）。

## 4. 目录与分层

```text
api/            路由装配（mountMailRoutes 同时给用户侧与管理员侧用）+ 接口级测试
cmd/mailprobe/  协议层诊断 CLI
configs/        环境变量
db/migrations/  两套同版本号的迁移（sqlite / postgres）
db/query/       sqlc 命名 SQL，两个方言各一份
db/generated/   sqlc 生成代码，不手改
pkg/crypto/     AES-256-GCM 凭据加解密
pkg/quota/      配额计算与消耗
pkg/mailer/     协议层：graph/、imapx/、回退链、代理、导入导出格式
pkg/job/        任务系统：Manager、worker pool、事件广播
pkg/handler/    HTTP 处理器、审计中间件、SSE writer
pkg/service/    业务逻辑
pkg/repo/       数据访问，在此吸收两种方言的差异
pkg/middleware/ 认证（会话 Cookie / API Key）、租户成员、平台管理员
pkg/model/      数据结构、DTO、RBAC 权限矩阵
web/src/        React 前端
```

| 层 | 职责 | 不该出现在这里的东西 |
|---|---|---|
| handler | 解析参数、调 service、映射错误到 HTTP | 业务规则、SQL |
| service | 业务规则、事务、加解密、配额、编排协议层 | `echo.Context`、HTTP 状态码 |
| repo | 调 sqlc 生成的查询，吸收方言差异 | 业务判断 |
| model | 结构体、常量、权限矩阵、`Normalize()` 这类纯函数 | 数据库与网络访问 |

文件超过 ~200 行就考虑拆分（`AccountService` 的导入、导出、批量各自成文件就是这么来的）。

## 5. 后端规约

### 5.1 统一响应格式

```jsonc
{ "code": 0, "data": {}, "message": "操作成功" }        // 成功
{ "code": 1, "data": null, "message": "具体错误信息" }  // 失败
{ "code": 0, "data": { "items": [], "pagination": { "page": 1, "limit": 50, "total": 0, "pages": 0 } } }
```

需要前端分支处理的业务码（定义在 `pkg/handler/response.go`）：

| code | 含义 |
|---|---|
| 1001 | 超出配额，`data` 为 null，上限与已用量写在 `message` 里 |
| 1003 | 用户已被管理员禁用 |
| 1004 | 邮箱账号已存在 |
| 1005 | 上游邮件服务失败，`data` 带 `{error_kind, channel}` |

`error_kind` 必须回传：前端据此区分「重新授权」「联系服务商」「检查代理」三种完全不同的处置，
只给一段文案用户不知道该做什么。

**401 只表示「本次请求的调用方没通过认证」，不表示别的。**
托管邮箱的凭据失效（`auth_failed` / `consent_required`）属于上游问题，走 502 + 1005，
具体原因由 `error_kind` 承载——`upstreamFailure` 曾经把它映射成 401，
结果是用户导入的一批账号里有一个 token 过期、点开就把**用户本人**踢回登录页。由
`api/mail_messages_test.go:TestUpstreamAuthFailureIsNotUnauthorized` 钉住。

5xx 的原始错误只记日志不回传（`failure()` 已处理），日志里带 `request_id`。

### 5.2 双引擎 SQL

**两个引擎各写各的 SQL**，不追求写法一致；repo 层方法签名统一。三条硬约束：

1. **`db/query/` 下的 SQL 必须全部 ASCII。** sqlc 遇到多字节字符会静默截断生成的 SQL 常量，
   运行时报 `incomplete input`，且爆炸点常在另一条查询上。`db/query/query_test.go` 拦这个。
2. **`ORDER BY` 无法参数化。** `sqlc.arg()` 会被原样留在 SQL 文本里，裸 `?` 被静默丢弃。
   每个「排序字段 × 方向」各写一条查询，由 service 按白名单分派；两个方言的 `.sql`
   由同一个脚本生成以防漂移。
3. **变长 IN 用 `sqlc.slice()`（SQLite）/ `= ANY($n::text[])`（PostgreSQL）。**
   `json_each()` 走不通——sqlc 的 SQLite 解析器不认识它。

改查询的流程：改 `db/query/` → `make sqlc-generate` → 补 `pkg/repo/parity_test.go` 的用例。
CI 会跑 `sqlc-verify` 和 PostgreSQL service container。

`TestGeneratedSQLHasNoLeftoverDirectives` 扫描生成产物里是否残留 `sqlc.arg` / `sqlc.slice`
——上面几个坑都属于「sqlc 静默留下非法 SQL」，必须在 CI 拦住。

### 5.3 多租户与权限

**一个租户空间只属于一个用户。**数据模型保留了完整的多租户结构（以后要做团队协作时
不用改表），但前端不展示这个概念：没有工作区切换、没有成员管理，后台也只有一份用户清单
（`AdminTenantsPage` 与 `GET /admin/tenants` 都已删除，配额与「进入其邮箱」并进了用户行）。
新增涉及租户的界面时，先想清楚它对用户意味着什么——多半应该表述成「用户」而不是「工作空间」。


- 每条业务 SQL 都带 `WHERE tenant_id = ?`，`tenantID` 来自 URL 且已由中间件校验成员身份
- 权限用 `middleware.Require(model.PermissionXxx)`，矩阵在 `pkg/model/permission.go`
- **平台管理员的跨租户放行只能有一处**：`middleware.Require` 里的那个口子，
  其前提是 `platform_user` 只由 `RequirePlatformAdmin` 设置。不要给管理员合成一个
  假的 `tenant_member`——那个假成员会流进审计和业务判断，事后分不清是真成员还是管理员
- 用户侧与管理员侧挂**同一份路由表**（`api.mountMailRoutes` 挂两次），不要抄第二遍

**第二条认证入口是 API Key**（`Authorization: Bearer`，2026-08-27 起）。它不是第二套接口：
Key 认证通过后被塞成一个只读的虚拟租户角色 `model.TenantRoleAPI`，走同一份 `/mail/**` 路由，
权限照常由 `middleware.Require` 收敛到 `group:read` / `account:read` / `message:read`。
三条不能松的约束：

- Key 只在 `/api/v1/tenants/<id>/**` 下有效（`middleware.tenantScopePrefix`）。
  它不属于任何用户，放它进以 `user_id` 为主语的端点，handler 会拿着空 `user_id` 往下走
- Key 拿不到 `tenant:update`，因此**读不到也重置不了自己**；拿不到 `account:secret`，导出照样 403
- 新增只读端点时想清楚 Key 是否也该能调——它跟着权限走，不跟着路由走

### 5.4 凭据与审计

- 邮箱密码、`refresh_token`、含口令的代理地址进出库经 `pkg/crypto`，密文永不出接口
  （`MailAccountResponse` 只回 `has_password` 这类布尔值）
- 代理地址要**先解密再打码**，解密失败回显「(无法解密)」而不是密文
- **协议层的写回必须是窄 UPDATE**（`auth_channel` / `refresh_token` / `last_refresh_*` 各一条），
  用整行改写会把用户此刻正在编辑的分组、备注、代理一起覆盖掉
- 写操作挂 `handler.AuditWrite`，管理员的读挂 `handler.AuditAdminRead`
  （只记管理员：普通用户翻十页邮件就是十条，会把真正要看的淹掉）
- **导出是全平台风险最高的接口**，三件必须同时在场：`account:secret` 权限、
  强制审计、按用户限流。少任何一件都等于开了一个不设防或不留痕的凭据出口

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MasterAlanLab/emailbox](https://github.com/MasterAlanLab/emailbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
