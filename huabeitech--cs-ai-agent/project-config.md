---
trigger: always_on
description: 本文件定义本项目内 AI Agent 的强制开发规则。除非用户明确要求偏离，否则必须遵循。
---

# AGENTS.md

本文件定义本项目内 AI Agent 的强制开发规则。除非用户明确要求偏离，否则必须遵循。

## 1. 基本原则

- 适用范围：仓库根目录及所有子目录
- 优先级：用户明确指令 > 本文件 > 默认实现习惯
- 若与用户要求冲突：先执行用户要求，并在变更说明中标注偏离点

## 2. 固定技术栈

- 后端：`Golang` + `Iris` + `GORM` + `github.com/mlogclub/simple`
- 数据库：同时兼容 `SQLite` 和 `MySQL`
- 前端：`Next.js(App Router)` + `React` + `shadcn/ui` + `Tailwind CSS`
- 前端包管理器：`pnpm`

## 3. 目录约定

```text
.
├── cmd/
│   ├── server/
│   ├── migration/
│   └── generator/
├── internal/
│   ├── bootstrap/
│   ├── config/
│   ├── models/
│   ├── repositories/
│   ├── services/
│   ├── controllers/
│   ├── migration/
│   ├── dto/
│   ├── convert/
│   ├── errorsx/
│   └── logx/
├── web/
└── docs/
```

## 4. 后端分层

必须遵循单向依赖：`models -> repositories -> services -> controllers`

- `models`：只定义实体和表映射
- `repositories`：只封装数据访问
- `services`：负责业务规则、事务编排、聚合逻辑
- `controllers`：只做参数解析、权限校验、service 调用、响应封装

禁止：

- controller 直接调用 repository
- 直接将 GORM model 返回前端
- 在 models/repositories 中写业务编排

## 4.1 分层全链路（models -> repositories -> services -> controllers -> builders）

本章是对“分层”规则的**可执行细化**：每一层只做该层应该做的事情，数据以 DTO 为中心流转，GORM 细节集中在 repository，事务边界集中在 service，返回组装集中在 builders/controller。

### 4.1.1 依赖方向（必须）

只允许如下依赖（单向）：

- `models` → 不依赖任何业务层
- `repositories` → 依赖 `models`、基础库（`gorm`/`simple/sqls`）
- `services` → 依赖 `repositories`、`models`、`enums/errorsx/utils`，负责事务与业务编排
- `builders` → 依赖 `models`、`dto/response`（必要时可依赖少量 `services` 用于补充展示字段，但优先在 service 里聚合好）
- `controllers` → 依赖 `services`、`builders`、`dto/request`、`web/params`、`web` 响应封装

禁止反向依赖：

- `repositories` 不能依赖 `services/controllers/builders`
- `models` 不能依赖 `repositories/services/controllers/builders`
- `controllers` 不能依赖 `repositories`（必须通过 service）

### 4.1.2 数据形态与流转（推荐统一）

一条典型 CRUD/业务动作的数据流：

1. **controller** 读取参数（query/body/form/path），做权限校验，调用 **service**
2. **service** 执行业务规则（校验、幂等、状态机、聚合），必要时开启事务，调用 **repository**
3. **repository** 只做数据读写（CRUD + 查询），返回 `models` 或必要的聚合结构
4. **builders** 将 `models`/聚合结果映射为 `response DTO`
5. **controller** 返回 `web.JsonData(...)` 或 `web.JsonPageData(...)`

强约束：

- **controller 入参使用 request DTO**
- **controller 出参使用 response DTO**
- **禁止直接返回 models 到前端**

### 4.1.3 各层“允许/禁止”清单

#### models（实体层）

- **允许**
  - 字段定义、表名、索引/约束标签、关联关系（GORM tags）
  - 轻量常量/枚举字段类型（更推荐放 `internal/pkg/enums`）
- **禁止**
  - 业务方法（例如 `CanDispatch()` 这类规则判断应在 service）
  - DB 访问、事务、复杂计算

#### repositories（数据访问层）

- **允许**
  - CRUD：`Get/Take/Find/FindOne/FindPageBy.../Create/Update/Updates/UpdateColumn/Delete`
  - 与查询相关的“可复用”方法：`FindByUserID`、`CountByStatus`、`FindActiveBy...`
  - 只要是“数据访问细节”，都应在这里（SQL 条件、排序、分页、锁）
- **禁止**
  - 业务编排（跨表流程、状态流转、事件发布等）
  - 权限判断、登录态判断
  - 直接拼接返回 DTO（DTO 映射属于 builders/controller）

Repository 最佳实践：

- **按主键读写优先提供统一方法**：`Get/Updates/Delete`，避免 service 层重复写 `id = ?`
- **查询条件优先使用 `sqls.Cnd` / `sqls.NewCnd()`**
- **repository 方法签名统一接收 `db *gorm.DB`**（支持 `sqls.DB()` 与 `ctx.Tx`）

#### services（业务层）

- **允许**
  - 业务规则：参数规范化、跨实体校验、状态机、幂等、并发语义
  - 聚合：需要组合多个 repository 结果
  - 事务编排：`sqls.WithTransaction(func(ctx *sqls.TxContext) error { ... })`
  - 调用 builders 前的领域对象整理（如果 builders 只做映射更干净）
- **禁止**
  - controller 才该做的事情：参数解析/HTTP 细节/响应封装
  - repository 才该做的事情：散落 GORM 查询（除非一次性复杂 SQL 且不值得抽）

Service 最佳实践：

- **事务只在“需要原子性”的地方开**，并确保事务内所有 DB 操作都走 `ctx.Tx`
- **service 内调用 repository**，不要“既有 repo 又直接 GORM”混搭造成风格分裂

#### builders（输出构建层）

定位：将 `models`（或 service 聚合结果）转换为 `response DTO`，避免 controller 写一堆映射样板代码。

- **允许**
  - `Model -> ResponseDTO` 的纯映射
  - 时间格式化、枚举 label 填充（必要时）
  - 批量构建：`BuildXxxList([]models.Xxx) []response.Xxx`
- **禁止**
  - DB 访问（builders 不应查询数据库）
  - 权限判断、事务、复杂业务流程

builders 推荐形式：

- 位置：`internal/builders/*_builder.go`
- 方法：`BuildXxx(item *models.Xxx) *response.Xxx` / `BuildXxxList(list []models.Xxx) []response.Xxx`

#### controllers（接口层）

- **允许**
  - 参数解析：`params.ReadJSON/ReadForm/NewPagedSqlCnd/GetInt64...`
  - 权限：`AuthService.GetAuthPrincipal/RequirePermission/HasPermission`
  - 调 service，调 builders，包装 `web.JsonData/JsonPageData/JsonError`
- **禁止**
  - 直接调用 repository
  - 直接返回 models
  - 在 controller 内写业务编排（例如“先写 A 再写 B”）

### 4.1.4 “事务”最佳实践（替代模糊口号）

事务边界应由 service 决定，原则如下：

- **必须开事务**（`sqls.WithTransaction`）
  - 多条写 SQL（例如更新主表 + 写日志表/事件表/关系表）
  - “读-改-写”且要求一致性（并发下不能错）
  - 跨多个 repository 的写操作需要原子性
- **不需要开事务**
  - 只有一次写 SQL（单条 `Create/Updates/UpdateColumn/Delete`）
  - 只有一次写 SQL + 纯计算/参数清洗

事务内规则：

- 在事务内，所有 DB 调用必须使用 `ctx.Tx`（repository 方法的 `db` 参数传 `ctx.Tx`）
- 禁止事务内混用 `sqls.DB()`（会脱离事务）

### 4.1.5 一个“标准接口”的代码骨架（示例）

```go
// Controller: 参数/权限/响应
func (c *XxxController) PostUpdate() web.JsonResult {
  operator := services.AuthService.GetAuthPrincipal(c.Ctx)
  if operator == nil {
    return web.JsonErrorMsg("未登录或登录已过期")
  }
  var req request.UpdateXxxRequest
  if err := params.ReadJSON(c.Ctx, &req); err != nil {
    return web.JsonError(err)
  }
  if err := services.XxxService.UpdateXxx(req, operator); err != nil {
    return web.JsonError(err)
  }
  return web.JsonSuccess()
}
```

```go
// Service: 业务规则 + 事务编排 + 调 repository
func (s *xxxService) UpdateXxx(req request.UpdateXxxRequest, operator *dto.AuthPrincipal) error {
  current := repositories.XxxRepository.Get(sqls.DB(), req.ID)
  if current == nil {
    return errorsx.InvalidParam("对象不存在")
  }
  // 只有一次写 SQL：不需要事务
  return repositories.XxxRepository.Updates(sqls.DB(), req.ID, map[string]any{
    "name": strings.TrimSpace(req.Name),

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [huabeitech/cs-ai-agent](https://github.com/huabeitech/cs-ai-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
