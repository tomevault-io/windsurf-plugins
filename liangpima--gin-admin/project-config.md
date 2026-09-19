---
trigger: always_on
description: Gin-Admin 是一套基于 Go + Gin + GORM + Vue3 + Element Plus 的后台管理框架。
---

# AGENTS.md — Gin-Admin 项目开发规范

## 项目概述

Gin-Admin 是一套基于 Go + Gin + GORM + Vue3 + Element Plus 的后台管理框架。
本文件定义了 AI Agent 在本项目中生成代码时必须遵守的规则和规范。

## 核心架构

采用 **Controller → Service → Repository** 三层架构。

```
Controller (接口层)
  ├── 参数接收 (ShouldBindJSON/ShouldBindQuery)
  ├── 参数校验 (binding tag)
  └── 返回统一 Response

Service (业务层)
  ├── 业务逻辑
  ├── 事务控制 (db.Transaction)
  └── 调用本模块 Repository

Repository (数据层)
  ├── 数据库操作 (GORM)
  └── 只做数据访问，不含业务逻辑
```

## 项目技术栈

- **后端**: Go 1.25, Gin v1.10, GORM v1.25, MySQL 5.7+, Redis 3.0+
- **认证**: JWT (golang-jwt/v5) + Casbin RBAC
- **前端**: Vue 3.5, Element Plus 2.9, Vite 6, TypeScript 5.7, Pinia 2.3
- **其他**: Zap 日志, Viper 配置, Swagger 文档, 1Password/robfig/cron

## 目录结构

```
go-admin/
├── cmd/server/main.go              # 唯一入口
├── config/
│   ├── config.yaml                 # 应用配置
│   ├── config.go                   # 配置加载
│   └── casbin/model.conf           # RBAC 模型
├── internal/
│   ├── cache/redis.go              # Redis 封装（可选）
│   ├── common/                     # 统一响应/错误码/业务错误语义/模型/分页/软删除唯一值释放
│   ├── database/mysql.go           # MySQL 连接
│   ├── logger/zap.go               # Zap 日志
│   ├── middleware/                 # 中间件
│   │   ├── casbin.go               # RBAC 鉴权 + 角色菜单策略同步
│   │   ├── casbin_adapter.go       # Casbin 策略的 GORM 适配器
│   │   └── permission.go           # 路由权限登记表（见规则13）
│   └── module/
│       ├── system/                 # 系统管理（用户/角色/菜单/部门/岗位/配置/字典/日志/文件/协议）
│       ├── payment/                # 支付模块
│       ├── member/                 # 会员模块（会员/等级/标签/积分）
│       ├── captcha/                # 验证码
│       └── monitor/                # 监控（占位，未实现）
├── pkg/
│   ├── auth/jwt.go                 # JWT 工具
│   ├── upload/                     # 多端文件上传（本地/OSS/COS/MinIO）
│   │   ├── upload.go               # 上传入口（自动选择存储方式 + 扩展名校验）
│   │   ├── local.go                # 本地存储
│   │   ├── aliyun_oss.go           # 阿里云 OSS
│   │   ├── tencent_cos.go          # 腾讯云 COS
│   │   └── minio.go                # MinIO
│   ├── excel/excel.go              # Excel 导入导出
│   ├── task/cron.go                # 定时任务调度
│   └── utils/                      # Hash/Snowflake/字符串工具
├── router/router.go               # 路由注册（含 /uploads 静态服务）
├── sql/                            # 数据库脚本
├── docs/                           # Swagger 文档
├── web/                            # 前端 (Vue3)
│   └── src/
│       ├── api/                    # API 接口定义（16个）
│       ├── components/             # 公共组件（10个）
│       ├── hooks/                  # useResponsive, useTheme
│       ├── layout/                 # 布局组件
│       ├── router/                 # 路由配置
│       ├── store/modules/          # app/permission/tagsView/user
│       ├── utils/                  # auth/format/request
│       └── views/                  # 页面（7个目录）
├── Makefile
├── start-all.ps1                   # 一键启动脚本
└── AGENTS.md
```

## 开发规则（13条铁律）

### 规则1: Controller 只负责参数接收与返回

- **允许**: 参数接收 (`ShouldBindJSON`/`ShouldBindQuery`)、参数校验 (`binding` tag)、返回结果 (`common.Success`/`common.Error`)
- **禁止**: 在 Controller 中编写业务逻辑、直接操作数据库、调用 Repository

### 规则2: Service 负责业务逻辑与事务控制

- **允许**: 业务判断、数据组装、事务管理 (`db.Transaction`)、调用本模块 Repository
- **禁止**: 直接返回 HTTP 响应、操作 `gin.Context`、引入 `net/http` 相关依赖

### 规则3: Repository 只负责数据库操作

- **允许**: GORM 查询、CRUD 操作、SQL 构建
- **禁止**: 业务逻辑判断、跨表关联查询（应通过 Service 组合）、返回 HTTP 响应

### 规则4: 禁止跨模块访问 Repository

每个 Service 只能访问自己模块的 Repository。

```
✅ 允许:
UserService  → UserRepository
RoleService  → RoleRepository

❌ 禁止:
PaymentService → UserRepository   (跨模块)
OrderService   → UserRepository   (跨模块)
```

如需跨模块数据，通过调用对应 Service 实现。

### 规则5: 所有接口必须返回统一 Response 结构

```go
// 成功
common.Success(c, data)
common.SuccessWithPage(c, list, total, page, pageSize)

// 失败 —— Service 返回的 error 一律用 FailWith，不要手写业务码
common.FailWith(c, err)

// 参数绑定失败等 Controller 自己产生的错误，直接用 Error
common.Error(c, common.CodeBadRequest, err.Error())
common.Unauthorized(c, "未登录")
common.Forbidden(c, "无权限")
```

禁止直接使用 `c.JSON()` 返回业务数据。

#### 错误语义：业务错误 vs 系统错误（必须遵守）

**`common.FailWith(c, err)` 是 Service 错误的唯一出口**，它按错误类型自动选择业务码：

| 错误来源 | 构造方式 | 对外业务码 | 文案 |
|----------|----------|-----------|------|
| 业务校验失败（重名、状态冲突、密码太弱…） | `common.NewBizError("...")` | 400 | 原文透出 |
| 操作目标不存在 | `common.NewNotFoundError("...")` | 404 | 原文透出 |
| DB / IO / 第三方失败 | 原样 `return err` | 500 | **通用文案**，真实错误只进日志 |

```go
// ✅ Service：业务错误显式标记，系统错误原样返回
if s.repo.CountByCode(tenantID, req.Code, 0) > 0 {
    return common.NewBizError("角色编码已存在")
}
user, err := s.repo.FindByID(tenantID, id)
if err != nil {
    return common.NotFoundOrErr(err, "用户不存在")   // gorm.ErrRecordNotFound → 404
}
return err                                        // 其余 → 500

// ✅ Controller：一行收口
if err := ctl.userService.Create(tenantID, &req, operatorID); err != nil {
    common.FailWith(c, err)
    return
}

// ❌ 禁止：把所有错误都写成同一个码
common.Error(c, common.CodeInternalError, err.Error())   // DB 故障与重名混为一谈
common.Error(c, common.CodeBadRequest, err.Error())      // DB 故障被说成「参数错误」
```

要点：

- **单条查询必须用 `common.NotFoundOrErr(err, "XX不存在")`**。GORM 查不到记录返回的是
  `gorm.ErrRecordNotFound`，不转换就会变成 500，用户完全不知道是自己传的 ID 不对
- Service 内部**复用已包装的方法**，不要绕过它直连 repository。例如 `CloseOrder` 应调
  `s.GetOrder()` 而非 `s.orderRepo.FindByOrderNo()`，否则 404 语义会丢
- 系统错误对外统一返回「服务器内部错误」，**不要把原始 error 回给调用方** ——

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [liangpima/Gin-Admin](https://github.com/liangpima/Gin-Admin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-19 -->
