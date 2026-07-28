---
trigger: always_on
description: - `mss-boot` 简称 `mss`
---

# mss-boot-admin 项目开发指南

## 项目简称
- `mss-boot` 简称 `mss`
- `mss-boot-admin` 简称 `admin`
- `mss-boot-admin-antd` 简称 `antd`
- `mss-boot-docs` 简称 `docs`

## 启动项目复用约束

当用户明确表达“启动项目”（含同义表达，如“启动前后端”“把项目跑起来”）时，默认执行以下流程，不再重复询问：

1. 启动 `admin` 后端：在 `mss-boot-admin` 目录执行 `go run . server`（后台运行）。
2. 启动 `antd` 前端：在 `mss-boot-admin-antd` 目录执行 `pnpm dev`（后台运行）。
3. 默认以当前仓库配置为准：后端参考 `config/application.yml` 的 `server.addr`（当前基线通常为 `0.0.0.0:8080`），前端开发端口通常为 `8000`。
4. 启动后输出可验证状态（如端口可访问、终端无致命报错），便于继续联调。
5. 若用户未显式要求停止服务，保持服务运行以支持后续调试。

## AIGC 提示词/文档落盘约束（与 mss-boot 保持一致）

在本仓库中，生成任何提示词或文档时必须遵守：

1. 仅允许在 `aigc/prompts/`（或其子目录）创建/更新提示词与文档。
2. 禁止在仓库根目录创建提示词/文档文件。
3. 若用户给出的目标路径不在 `aigc/prompts/` 下，需自动重定向到 `aigc/prompts/`。
4. 文件名使用小写 kebab-case；中文文档使用 `.zh-CN.md` 后缀。
5. 输出结果时必须返回实际写入路径。

## 开源项目协作约束（新增）

本仓库为开源项目，生成注解/文档/提示词时必须遵守：

1. 使用开源协作语气：中立、可讨论、可验证，避免绝对化表述。
2. 结论应基于仓库当前可见代码与文档，必要时明确“适用范围与假设”。
3. 不写入敏感信息（密钥、凭据、私有地址、个人隐私、内部系统细节）。
4. 优先输出可复现步骤与验证方式，便于社区贡献者复核。
5. 注释和建议应面向外部贡献者可理解，避免依赖内部上下文。

## 项目概述
mss-boot-admin 是一个基于 Gin + React + Ant Design v5 + Umi v4 + mss-boot 的前后端分离权限管理系统。当前产品主线是治理优先、Agent 可读、可持续维护的后台管理能力：RBAC 权限、系统配置、通知、国际化、可观测与发布治理。

历史代码中仍保留租户、虚拟模型和代码生成相关实现。除非用户明确要求维护这些历史能力，否则不要把它们作为新功能设计的默认方向；新开发应优先围绕单租户可维护后台、清晰契约、可审计发布和 AI 协作上下文展开。

## 核心技术栈

### 后端技术
- **Web框架**: Gin - 高性能的Go HTTP框架
- **ORM**: GORM - 数据库操作
- **权限管理**: Casbin - RBAC权限控制
- **认证**: gin-jwt + OAuth2 - 支持JWT和第三方登录
- **API文档**: Swag + gin-swagger - 自动生成Swagger文档
- **数据库**: 支持 MySQL 8.0+, PostgreSQL, SQLite, SQL Server, DM(达梦)
- **缓存**: Redis + redislock
- **消息队列**: NSQ, Kafka (支持AWS MSK)
- **配置中心**: Consul, 支持多配置源(本地文件、embed、S3、数据库、MongoDB)
- **监控**: Pyroscope (性能分析), Prometheus (指标采集)
- **云服务**: AWS SDK (EKS, S3, Secrets Manager)

### 核心依赖库
- **mss-boot**: 内部核心框架库
- **协议**: gRPC, WebSocket (gorilla/websocket)
- **调度**: robfig/cron - 定时任务
- **工具**: spf13/cobra (命令行), go-git (Git操作)

## 项目架构

### 目录结构规范

```
mss-boot-admin/
├── apis/          # API控制器层 - 定义RESTful接口和路由
├── cmd/           # 命令行入口 - Cobra命令定义
│   ├── migrate/   # 数据库迁移命令
│   └── server/    # 服务启动命令
├── config/        # 配置管理 - 支持多配置源
├── dto/           # 数据传输对象 - 请求/响应结构体
├── models/        # 数据模型层 - GORM实体定义
├── service/       # 业务逻辑层 - 核心业务处理
├── middleware/    # 中间件 - 认证、权限、租户等
├── router/        # 路由配置 - 注册控制器
├── center/        # 中心化服务 - 租户、缓存、队列管理
├── pkg/           # 工具包 - 通用工具函数
├── notice/        # 通知模板 - 邮件等通知
└── compose/       # Docker Compose配置 - 本地开发环境
```

### 核心设计模式

#### 1. **租户相关历史能力 (Compatibility)**
- `ModelGormTenant`、`models.Tenant`、`center.TenantImp` 等租户相关代码仍存在，主要作为兼容和维护对象。
- 新功能不要默认引入租户隔离，除非需求明确说明需要恢复或维护租户能力。
- 修改这些代码时必须补充兼容性、数据隔离和回滚说明。

```go
// 历史能力示例：租户模型定义
type YourModel struct {
    models.ModelGormTenant  // 仅在明确需要租户兼容时使用
    // 其他字段...
}
```

#### 2. **权限管理 (RBAC with Casbin)**
- 基于 Casbin 的 RBAC 权限控制
- 支持角色-菜单-API三级权限映射
- 数据权限支持7种范围: 全部数据、本部门、本部门及子部门、自定义部门、仅本人、本人及下属、本人及所有下属
- 核心类型: `models.Role`, `models.Menu`, `models.API`, `pkg.AccessType`

```go
// 权限类型枚举
const (
    MenuAccessType      AccessType = "menu"      // 菜单权限
    APIAccessType       AccessType = "api"       // API权限
    ComponentAccessType AccessType = "component" // 组件权限
)
```

#### 3. **认证系统 (Authentication)**
- JWT Token认证机制
- 支持多种登录方式:
  - 用户名密码登录 (`pkg.UsernameLoginProvider`)
  - GitHub OAuth2 登录 (`pkg.GithubLoginProvider`)
  - 飞书/Lark OAuth2 登录 (`pkg.LarkLoginProvider`)
  - 邮箱验证码登录 (`pkg.EmailLoginProvider`)
  - 邮箱注册登录 (`pkg.EmailRegisterProvider`)
- 支持Personal Access Token (PAT)
- 核心类型: `middleware.Auth`, `models.UserLogin`, `models.UserAuthToken`

#### 4. **虚拟模型历史能力 (Virtual Model Compatibility)**
- `models.Model`、`models.Field` 等动态模型代码仍存在，主要作为兼容和维护对象。
- 新功能不要默认走动态模型或配置生成 CRUD，除非用户明确指定。
- 需要新增后台能力时，优先使用清晰的模型、DTO、API、权限和文档契约。

#### 5. **统计分析 (Statistics)**
- 内置统计接口，自动跟踪数据变化
- 支持实时统计和定时校准
- 核心类型: `models.Statistics`, `center.StatisticsImp`

```go
// 实现统计接口
func (*YourModel) StatisticsName() string { return "your-model-total" }
func (*YourModel) StatisticsType() string { return "your-model" }
func (*YourModel) StatisticsTime() string { return pkg.NowFormatDay() }
func (*YourModel) AfterCreate(tx *gorm.DB) error {
    _ = center.Default.NowIncrease(ctx, model)
    return nil
}
```

### 数据库设计规范

#### 基础模型类型

1. **ModelGorm** - 基础模型（无租户）
   - ID (varchar 64)
   - CreatedAt, UpdatedAt, DeletedAt (软删除)

2. **ModelGormTenant** - 历史租户兼容模型
   - 继承 ModelGorm
   - TenantID (varchar 64) - 历史租户隔离字段
   - CreatorID (varchar 64) - 创建人（支持数据权限）
   - Remark (text) - 备注

3. **命名规范**
   - 表名前缀: `mss_boot_`
   - 外键命名: `{model}_id`
   - 索引: 租户ID、创建人ID、状态字段需建索引

### API开发规范

#### 控制器开发模式

使用 `response.Controller` 框架自动生成RESTful API:

```go
// 示例：标准控制器定义
func init() {
    e := &YourController{
        Simple: controller.NewSimple(
            controller.WithAuth(true),                      // 启用认证
            controller.WithModel(new(models.YourModel)),    // 关联模型
            controller.WithSearch(new(dto.YourModelSearch)), // 搜索DTO
            controller.WithModelProvider(actions.ModelProviderGorm), // 数据提供者
            controller.WithScope(center.Default.Scope),     // 作用域
        ),
    }
    response.AppendController(e) // 注册控制器
}

type YourController struct {
    *controller.Simple
}
```

#### 自动生成的路由
- `GET /api/your-models` - 列表查询
- `GET /api/your-models/:id` - 详情查询
- `POST /api/your-models` - 创建
- `PUT /api/your-models/:id` - 更新

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mss-boot-io/mss-boot-admin](https://github.com/mss-boot-io/mss-boot-admin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
