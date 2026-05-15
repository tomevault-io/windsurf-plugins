---
trigger: always_on
description: 多租户 AI API 网关 SaaS 平台。租户注册后获取 API Key，通过统一接口调用 OpenAI/Claude/Gemini 等大模型，平台负责计费、限流、监控。
---

# Team-API — 项目开发规范

## 项目概述

多租户 AI API 网关 SaaS 平台。租户注册后获取 API Key，通过统一接口调用 OpenAI/Claude/Gemini 等大模型，平台负责计费、限流、监控。

双控制台架构：管理后台（平台运营）+ 租户控制台（组织使用），两套完全独立的用户体系。

## 技术栈

- **后端**：Go + GoFrame (v2) + goose（数据库迁移）
- **数据库**：PostgreSQL（唯一，不用 MySQL/SQLite）
- **缓存**：Redis + 内存缓存（双层缓存）
- **前端**：Vue 3 + Vite + TailwindCSS + Naive UI（管理后台）
- **包管理**：bun
- **对象存储**：S3 / OSS / COS / MinIO（禁止纯本地磁盘作为生产存储）

## 项目结构

项目基于 GoFrame v2 标准项目结构（`gf init` 生成），在此基础上扩展业务模块。

```
team-api/
├── api/                        # API 定义（请求/响应结构体 + 路由注解）
│   ├── admin/v1/               #   管理后台 API（gf gen ctrl 源）
│   ├── tenant/v1/              #   租户控制台 API（gf gen ctrl 源）
│   ├── captcha/v1/             #   验证码 API（管理后台 + 租户控制台共用）
│   ├── docs/v1/                #   OpenAPI 文档 API
│   ├── open/v1/                #   开放平台 API
│   ├── payment/v1/             #   支付回调 API
├── hack/
│   └── config.yaml             # gf CLI 脚手架配置（gen dao 等工具的数据库连接和生成规则）
├── internal/
│   ├── cmd/                    # 入口命令、服务启动、路由注册
│   │   ├── cmd.go              #   main 包注册，路由分组定义
│   │   └── cmd_reset_pwd.go    #   密码重置命令
│   ├── consts/                 # 常量定义（业务状态码、枚举等）
│   ├── controller/             # 控制器（由 gf gen ctrl 从 api/ 自动生成，禁止手动修改）
│   │   ├── admin/              #   管理后台控制器
│   │   ├── tenant/             #   租户控制台控制器
│   │   ├── captcha/            #   验证码控制器
│   │   ├── docs/               #   文档控制器
│   │   ├── open/               #   开放平台控制器
│   ├── handler/                # 特殊端点处理器（不走 Controller 链路，见下方说明）
│   │   ├── relay/              #   AI 代理端点处理器
│   │   ├── public/             #   支付回调端点处理器
│   │   └── setup/              #   系统初始化向导处理器
│   ├── logic/                  # 业务逻辑实现（核心代码所在）
│   │   ├── admin/              #   管理后台业务逻辑
│   │   ├── tenant/             #   租户控制台业务逻辑
│   │   ├── billing/            #   计费引擎
│   │   ├── common/             #   公共逻辑（缓存、配置、安全、邮件、验证码、JWT 等）
│   │   ├── docs/               #   OpenAPI 文档生成
│   │   ├── monitor/            #   监控告警引擎
│   │   ├── notification/       #   通知服务
│   │   ├── open/               #   开放平台业务逻辑
│   │   ├── payment/            #   支付逻辑（回调处理、订单履约）
│   │   ├── relay/              #   Relay 业务逻辑（亲和性、健康检查、缓存）
│   │   └── task/               #   异步任务框架
│   ├── model/                  # 数据模型（由 gf gen dao 自动生成，禁止手动修改）
│   │   ├── do/                 #   Domain Objects（查询条件、输入结构体）
│   │   └── entity/             #   Entity（与数据库表 1:1 映射的数据实体）
│   ├── packed/                 # 打包资源（内嵌静态文件、编译时生成）
│   ├── dao/                    # 数据访问对象（由 gf gen dao 自动生成，禁止手动修改）
│   ├── response/               # 统一响应工具包（Success/Error 封装）
│   ├── service/                # 业务接口定义（由 gf gen service 从 logic/ 自动生成，禁止手动修改）
│   ├── middleware/             # 中间件
│   └── utility/                # 工具函数
│       ├── crypto/             #   加密工具
│       ├── export/             #   数据导出
│       ├── totp/               #   TOTP 两步验证
│       └── turnstile/          #   Cloudflare Turnstile 验证
├── relay/                      # AI 模型代理层（顶级模块，独立于 GoFrame 脚手架生成目录）
│   ├── channel/                #   供应商适配器（registry.go + 24 个子目录，每个供应商一个）
│   │   └── openai/ claude/ gemini/ ...
│   ├── common/                 #   Relay 层共享类型和工具
│   ├── constant/               #   Relay 层常量（供应商类型、模式、错误码）
│   ├── dto/                    #   数据传输对象（OpenAI/Claude/Gemini/Realtime/Task 等格式定义）
│   ├── handler/                #   Relay 请求处理器（chat/claude/gemini/audio/rerank/task/realtime 等）
│   ├── helper/                 #   Relay 辅助函数（流式处理、状态码映射、系统提示词、thinking 处理）
│   ├── override/               #   请求/响应覆盖（Header 改写、参数覆盖）
│   ├── scheduler/              #   渠道调度器（调度、亲和性、重试）
│   └── taskchannel/            #   异步任务渠道适配器
│       ├── registry.go         #     任务渠道注册表
│       ├── kling/              #     可灵视频生成
│       ├── midjourney/         #     Midjourney 图像生成
│       ├── sora/               #     Sora 视频生成
│       └── suno/               #     Suno 音乐生成
├── manifest/
│   ├── config/                 # 应用运行时配置（GoFrame 标准路径）
│   ├── deploy/                 # 部署配置（K8s/Compose 等）
│   ├── docker/                 # Docker 构建配置
│   ├── i18n/                   # 国际化资源文件
│   └── protobuf/               # Protobuf 定义文件
├── migrations/                 # goose 数据库迁移脚本（六位序号编号：000001_xxx.sql）
├── web/                        # 前端
│   ├── admin/                  # 管理后台
│   └── tenant/                 # 租户控制台
├── docs/                       # 项目文档
│   ├── 开发计划-v2/            #   分周期开发计划（周期一～六）
│   └── 协议文档/               #   协议相关文档
├── new-api/                    # 参考项目（只读，不修改）
├── sub2api/                    # 参考项目（只读，不修改）
├── main.go                     # 应用入口
├── go.mod
└── Makefile
```

### Relay 层设计说明

Relay 层作为顶级 `relay/` 模块独立于 GoFrame 脚手架生成目录（`internal/` 下的 `controller/service/dao/model`），原因：
- **文件量大**：24 个供应商适配器 + 4 个异步任务适配器 + 共享模块，放在 `internal/logic/` 下层级过深
- **职责独立**：Relay 是纯粹的代理转发层，不依赖 GoFrame ORM/DAO，与 `internal/` 下的业务逻辑解耦
- **参考对齐**：目录结构与 new-api 的 `relay/` 保持一致，降低移植和理解成本

`internal/logic/` 中的业务逻辑（如计费、渠道调度）通过 Go import 调用 `relay/` 包，而非将 relay 嵌入 logic 内部。

### GoFrame 代码生成与手动编写分界

| 目录 | 生成方式 | 能否手动修改 |
|------|---------|------------|
| `api/` | 手动编写 | 是（结构体定义和路由注解） |
| `hack/config.yaml` | 手动编写 | 是（CLI 工具配置） |
| `internal/controller/` | `gf gen ctrl` | 否（自动生成） |
| `internal/service/` | `gf gen service` | 否（自动生成接口） |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [qianfree/team-api](https://github.com/qianfree/team-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
