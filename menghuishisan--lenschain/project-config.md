---
trigger: always_on
description: > **文档驱动开发（Documentation-Driven Development）**
---

# 链镜平台（LensChain）— 开发规范与项目指南

> **文档驱动开发（Documentation-Driven Development）**
> 一切开发以文档和规范为引擎。开发任何功能前，必须先阅读对应模块的设计文档。
> 文档是唯一真相源（Single Source of Truth），代码必须与文档保持一致。
> 如发现文档与实际需求有冲突，**先更新文档，再改代码，绝不允许反过来**。

> 本文件是 AI 辅助开发的核心指引，所有代码生成和修改必须遵守以下规范。
> 详细业务需求请查阅 `docs/modules/` 下各模块文档。

---

## 一、项目概述

链镜是一个区块链综合教学平台，集教学、实验实践、CTF竞赛三位一体，支持多链生态、多层次学生、多学校混合部署。

- **8个业务模块：** 用户与认证、学校与租户管理、课程与教学、实验环境、CTF竞赛、评测与成绩、通知与消息、系统管理与监控
- **4种用户角色：** 超级管理员、学校管理员（教师兼任）、教师、学生
- **多租户隔离：** 以 `school_id` 为租户标识，业务数据严格隔离

---

## 二、技术栈与版本

| 层级 | 技术 | 版本要求 |
|------|------|----------|
| 后端 | Go + Gin | Go 1.22+ |
| 前端 | React + Next.js (App Router) | Next.js 14+, React 18+ |
| 数据库 | PostgreSQL | 15+ |
| 缓存 | Redis | 7+ |
| 消息队列 | NATS | 2.10+ |
| 对象存储 | MinIO (S3兼容) | — |
| 容器编排 | Docker + Kubernetes | — |
| 仿真引擎 | SimEngine Core (Go微服务) + 场景算法容器 + 前端渲染器 (TypeScript) | — |
| 前端样式 | Tailwind CSS | 3.4+ |
| UI 组件 | shadcn/ui | — |
| 图标库 | Lucide React | — |
| 状态管理 | Zustand | 4+ |
| 数据请求 | TanStack Query (React Query) | 5+ |

---

## 三、仓库结构（Monorepo）

```
LensChain/
├── backend/                # Go 后端服务
├── frontend/               # Next.js 前端应用
├── sim-engine/             # 可视化仿真引擎（SimEngine）
├── deploy/                 # 部署与运维配置
├── docs/                   # 项目文档（已完成）
├── scripts/                # 工具脚本（数据库迁移、种子数据等）
├── CLAUDE.md               # 本文件
└── README.md
```

---

### 3.1 backend/ — Go 后端（标准分层架构）

```
backend/
├── cmd/
│   └── server/
│       └── main.go              # 程序入口
├── internal/
│   ├── router/                  # 路由注册层（路径与中间件绑定，不含处理逻辑）
│   │   ├── router.go            # 总路由入口，初始化 Gin Engine，挂载各模块路由
│   │   ├── auth.go              # 模块01 路由组
│   │   ├── school.go            # 模块02 路由组
│   │   ├── course.go            # 模块03 路由组
│   │   ├── experiment.go        # 模块04 路由组
│   │   ├── ctf.go               # 模块05 路由组
│   │   ├── grade.go             # 模块06 路由组
│   │   ├── notification.go      # 模块07 路由组
│   │   └── system.go            # 模块08 路由组
│   ├── handler/                 # HTTP 处理层（按模块分子目录，按功能域分文件）
│   │   ├── auth/                # 模块01
│   │   │   ├── login.go         # 登录、登出、Token刷新
│   │   │   ├── user.go          # 用户管理、导入、个人中心
│   │   │   └── security.go      # SSO、安全策略
│   │   ├── school/              # 模块02（单文件）
│   │   │   └── school.go        # 入驻申请、学校管理、SSO配置
│   │   ├── course/              # 模块03
│   │   │   ├── course.go        # 课程CRUD、章节、课时、选课
│   │   │   ├── assignment.go    # 作业管理、提交、批改
│   │   │   └── discussion.go    # 讨论区、评价、公告、统计
│   │   ├── experiment/          # 模块04
│   │   │   ├── template.go      # 模板、镜像、场景、标签
│   │   │   ├── instance.go      # 实例生命周期、检查点、快照
│   │   │   └── group.go         # 分组、组内通信、教师监控
│   │   ├── ctf/                 # 模块05
│   │   │   ├── competition.go   # 竞赛管理、题目、报名
│   │   │   ├── battle.go        # 攻防赛回合、攻击、防守
│   │   │   └── environment.go   # 题目环境、队伍链、监控
│   │   ├── grade/               # 模块06（单文件）
│   │   │   └── grade.go         # 学期、成绩审核、申诉、预警
│   │   ├── notification/        # 模块07（单文件）
│   │   │   └── notification.go  # 通知、公告、模板、偏好
│   │   └── system/              # 模块08（单文件）
│   │       └── system.go        # 审计、配置、告警、备份
│   ├── service/                 # 业务逻辑层（同样按模块分子目录）
│   │   ├── auth/
│   │   ├── school/
│   │   ├── course/
│   │   ├── experiment/
│   │   ├── ctf/
│   │   ├── grade/
│   │   ├── notification/
│   │   └── system/
│   ├── repository/              # 数据访问层（同样按模块分子目录）
│   │   ├── auth/
│   │   ├── school/
│   │   ├── course/
│   │   ├── experiment/
│   │   ├── ctf/
│   │   ├── grade/
│   │   ├── notification/
│   │   └── system/
│   ├── model/                   # 数据模型
│   │   ├── entity/              # 数据库表映射结构体（按模块分文件）
│   │   │   ├── user.go          # users, user_profiles 等
│   │   │   ├── school.go        # schools, school_applications 等
│   │   │   ├── course.go        # courses, chapters, lessons 等
│   │   │   ├── experiment.go    # experiment_templates, instances 等
│   │   │   ├── ctf.go           # competitions, challenges 等
│   │   │   ├── grade.go         # semesters, grade_reviews 等
│   │   │   ├── notification.go  # notifications, templates 等
│   │   │   └── system.go        # system_configs, alert_rules 等
│   │   ├── dto/                 # 请求/响应 DTO（按模块分文件）
│   │   └── enum/                # 枚举常量定义（按模块分文件）
│   ├── middleware/              # 中间件（JWT鉴权、RBAC权限、多租户注入、日志、限流）
│   └── pkg/                    # 内部公共包（不对外导出）
│       ├── snowflake/           # 雪花ID生成器
│       ├── response/            # 统一响应封装
│       ├── errcode/             # 业务错误码定义
│       └── validator/           # 自定义校验器
├── pkg/                         # 可导出公共包（供 sim-engine 等其他服务使用）
├── configs/                     # 配置文件（yaml）
├── migrations/                  # 数据库迁移文件（SQL）
└── go.mod
```

**分层调用规则（严格单向依赖）：**
```
main.go → router → handler → service → repository → model
              ↓        ↓         ↓          ↓
          middleware   pkg       pkg        pkg
```
- **router** 只负责路径注册和中间件绑定，不含任何处理逻辑
- **handler** 不得直接调用 repository，不得包含业务逻辑
- **service** 不得引用 `*gin.Context` 或任何 HTTP 相关类型
- **repository** 不得包含业务逻辑，不得调用 service
- **model** 不依赖任何其他层
- **跨模块 service 调用** 必须通过接口（interface）解耦，不直接引用具体实现

**文件拆分原则：**
- 单文件应尽量保持精简，**建议控制在 500-800 行以内**
- 超过 800 行时必须评估是否按功能域拆分；如果拆分会明显破坏内聚性，可保留但需确保职责单一、结构清晰、便于维护
- handler/service/repository 三层统一按模块建子目录

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [menghuishisan/LensChain](https://github.com/menghuishisan/LensChain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
