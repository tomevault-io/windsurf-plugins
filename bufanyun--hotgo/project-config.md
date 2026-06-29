---
trigger: always_on
description: **生成或修改任何代码前，必须完整阅读本文档。**
---

# HotGo 项目开发规范

**生成或修改任何代码前，必须完整阅读本文档。**  
本规范是项目唯一事实来源，所有新增代码必须严格对齐现有模块风格，不得自创新的代码模式。

---

## 技术栈

| 层 | 技术                                                             |
|----|----------------------------------------------------------------|
| 后端语言 | Go，module 名 `hotgo`                                            |
| 后端框架 | GoFrame v2（gdb、ghttp、gcron、gerror、gctx、gerror、gjson、gvar） |
| 前端框架 | Vue 3 + Vite + TypeScript                                      |
| UI 组件库 | **Naive UI**（优先使用，未经允许不得引入其他 UI 库）                             |
| 状态管理 | Pinia                                                          |
| HTTP 客户端 | 项目封装 `@/utils/http/axios`（`http.request`），禁止直接使用 axios         |

---

## 后端目录结构

```
server/
├── addons/                       # 插件模块（每个插件独立微架构，见"插件开发"章节）
│   ├── modules/                  # 隐式注册文件（每个插件一个 .go，仅含 import）
│   └── <addonName>/              # 插件目录，与主模块结构镜像
├── api/admin/<module>/           # 主模块 HTTP 契约：Req/Res + g.Meta 路由声明
├── internal/
│   ├── consts/                   # 全局常量与字典选项（枚举在此定义并注册）
│   ├── global/                   # 全局公用变量（谨慎添加，尽量少用）
│   ├── controller/admin/sys/     # 薄控制器，只做参数透传，不含业务逻辑
│   ├── crons/                    # 主模块定时任务注册与处理
│   ├── queues/                   # 主模块消息队列消费者注册与处理
│   ├── logic/sys/                # 业务实现（sSysXxx），init() 注册到 service
│   ├── library/                  # 内部基础库（addons、dict、hgorm、cache、queue、cron 等）
│   ├── service/                  # 接口声明（ISysXxx）+ 注册函数
│   ├── dao/                      # 对外 DAO；internal/dao/internal/ 为 CLI 生成，禁止手改
│   ├── model/
│   │   ├── entity/               # 表结构体（CLI 生成，禁止手改）
│   │   ├── do/                   # ORM 写入结构（CLI 生成，禁止手改）
│   │   └── input/sysin/          # 业务入参、出参、过滤结构
│   └── router/
│       ├── admin.go              # 主路由，末尾调用 genrouter.Register()
│       └── genrouter/            # 每个生成模块一个文件，init() 追加到 LoginRequiredRouter
├── resource/
│   ├── generate/default/curd/    # 主模块 CRUD 代码生成模板（参考，禁止直接修改）
│   └── generate/default/addon/   # 插件脚手架代码生成模板（参考，禁止直接修改）
└── utility/                      # 通用工具：convert、excel、validate 等
```

---

## 后端分层约定

### 分层职责（禁止跨层调用）

```
api（HTTP契约） → controller（参数透传） → service（接口） → logic（业务实现） → dao（数据访问）
```

- **controller 层**：不写任何业务逻辑，只转调 service
- **logic 层**：不出现 HTTP 相关代码，所有校验通过 `Filter()` 或 `g.Validator()` 完成
- **dao 层**：`internal/dao/internal/` 由 CLI 自动生成，禁止手改

---

### 1. API 层 `api/admin/<module>/<module>.go`

- 每个接口一对 `XxxReq` / `XxxRes`，`g.Meta` 声明路由
- GET 接口用 `method:"get"`，写操作用 `method:"post"`
- 请求体嵌入 `sysin.XxxInp`，响应体嵌入对应 Model 或空 `struct{}`

```go
type ListReq struct {
    g.Meta `path:"/xxxModule/list" method:"get" tags:"XXX模块" summary:"获取XXX列表"`
    sysin.XxxListInp
}
type ListRes struct {
    form.PageRes
    List []*sysin.XxxListModel `json:"list" dc:"数据列表"`
}
```

### 2. Controller 层 `internal/controller/admin/sys/<module>.go`

- 变量：`var XxxModule = cXxxModule{}`，类型：`type cXxxModule struct{}`
- 只做参数转发和结果组装，禁止直接操作 dao 或写业务逻辑

```go
func (c *cXxxModule) List(ctx context.Context, req *xxxmodule.ListReq) (res *xxxmodule.ListRes, err error) {
    list, totalCount, err := service.SysXxxModule().List(ctx, &req.XxxListInp)
    if err != nil {
        return
    }
    if list == nil {
        list = []*sysin.XxxListModel{}
    }
    res = new(xxxmodule.ListRes)
    res.List = list
    res.PageRes.Pack(req, totalCount)
    return
}
```

### 3. Service 层 `internal/service/sys.go`

- 在已有文件中**追加**接口声明和注册函数，禁止新建文件
- 声明 `ISysXxxModule` 接口，提供 `SysXxxModule()` 获取函数和 `RegisterSysXxxModule()` 注册函数

### 4. Logic 层 `internal/logic/sys/<module>.go`

- 结构体：`type sSysXxxModule struct{}`
- `init()` 里调用 `service.RegisterSysXxxModule(NewSysXxxModule())`
- `Model()` 方法：`return handler.Model(dao.XxxTable.Ctx(ctx), option...)`
- 列名引用必须用 `dao.XxxTable.Columns().FieldName`，禁止硬编码字符串字段名
- `Edit` 方法以 `in.Id > 0` 区分更新/新增，更新用 `XxxUpdateFields`，新增用 `XxxInsertFields`
- 字段校验在 `sysin` 的 `Filter(ctx)` 方法完成，logic 层不重复校验

### 5. Model 层 `internal/model/input/sysin/<module>.go`

| 类型 | 用途 |
|------|------|
| `XxxUpdateFields` | 更新时字段白名单 |
| `XxxInsertFields` | 新增时字段白名单 |
| `XxxEditInp` | 嵌入 `entity.XxxTable`，含 `Filter()` 校验方法 |
| `XxxListInp` | 列表查询入参，嵌入 `form.PageInp` |
| `XxxListModel` | 列表返回字段 |
| `XxxViewInp` / `XxxViewModel` | 详情入参/出参 |
| `XxxDeleteInp` | 删除入参（通常含 `Ids []int64`） |
| `XxxStatusInp`、`XxxMaxSortInp` 等 | 其他操作 |

### 6. 路由注册 `internal/router/genrouter/<module>.go`

```go
package genrouter

import "hotgo/internal/controller/admin/sys"

func init() {
    LoginRequiredRouter = append(LoginRequiredRouter, sys.XxxModule) // XXX模块
}
```

新建此文件即可，`genrouter/init.go` 通过 `init()` 链自动加载，无需修改 `admin.go`。

---

## 字典与枚举规范

**字典由后端统一维护**，前端通过固定接口拉取，禁止在前端硬编码枚举值。

### 后端：定义与注册（`internal/consts/` 下对应文件）

```go
// 1. 定义常量
const (
    XxxStatusEnabled = 1 // 启用
    XxxStatusDisable = 2 // 禁用
)

// 2. 定义选项（选择合适的样式函数）
var XxxStatusOptions = []*model.Option{
    dict.GenSuccessOption(XxxStatusEnabled, "启用"),
    dict.GenWarningOption(XxxStatusDisable, "禁用"),
}

// 3. 在 init() 中注册（同文件）
func init() {
    dict.RegisterEnums("XxxStatusOptions", "XXX状态选项", XxxStatusOptions)
}
```

**样式函数选择规则：**

| 函数 | 适用场景 |
|------|---------|
| `dict.GenSuccessOption` | 正常、启用、成功 |
| `dict.GenWarningOption` | 待处理、禁用、警告 |
| `dict.GenErrorOption` | 失败、封禁、危险 |
| `dict.GenInfoOption` | 信息类、中性状态 |
| `dict.GenPrimaryOption` | 主要操作、强调 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bufanyun/hotgo](https://github.com/bufanyun/hotgo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
