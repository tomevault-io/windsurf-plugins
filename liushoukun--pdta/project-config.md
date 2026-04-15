---
trigger: always_on
description: 业务系统文档架构规范 - 定义文档目录结构和组织方式
---


# 业务系统文档架构规范

## 📁 文档根目录结构

```
docs/
├── product/          # 产品文档（产品经理维护）
├── user/             # 用户手册（面向最终用户）
├── requirement/      # 需求管理（产品+研发+测试）
├── technical/        # 技术文档（技术团队维护）
├── operations/       # 运维文档（运维团队维护）
├── training/         # 培训资料（各角色培训）
├── project/          # 项目管理（项目经理维护）
├── knowledge/        # 知识库（团队沉淀）
├── index.md          # VitePress文档首页
└── CHANGELOG.md      # 系统更新日志
```

## 📱 product/ - 产品文档

**维护者**：产品经理  
**目的**：说明产品定位、业务流程、功能规划

```
product/
├── index.md                  # 产品文档首页
├── overview.md               # 产品概述（系统定位/目标用户/核心价值）
├── business-process/         # 业务流程（含流程图）
│   ├── index.md              # 业务流程概述
│   ├── user-registration.md
│   ├── order-process.md
│   └── payment-process.md
├── features/                 # 功能列表
│   ├── index.md              # 功能列表首页
│   ├── feature-list.md       # 功能清单
│   └── [module]-feature.md   # 各模块功能说明
├── roadmap/                  # 产品规划
│   ├── index.md              # 产品规划概述
│   ├── current-version.md
│   ├── next-version.md
│   └── backlog.md
├── release-notes/            # 版本发布说明
│   ├── index.md              # 版本发布说明首页
│   └── vX.X.X.md
└── wireframes/               # 原型设计
    └── index.md              # 原型设计说明
```

**命名规范**：
- 业务流程：使用动词短语，如 `user-registration.md`
- 功能说明：使用模块名，如 `user-management.md`
- 版本说明：使用版本号，如 `v1.0.0.md`

## 👤 user/ - 用户手册

**维护者**：产品经理 + 技术文档工程师  
**目的**：帮助最终用户快速上手和使用系统

```
user/
├── index.md                  # 用户手册首页
├── quick-start.md            # 快速入门（5分钟上手）
├── login-register.md         # 登录注册
├── guides/                   # 按角色分类的操作指南
│   ├── index.md              # 操作指南概述
│   ├── admin/                # 管理员操作
│   │   ├── index.md          # 管理员操作概述
│   │   ├── user-management.md
│   │   ├── role-management.md
│   │   └── system-settings.md
│   ├── operator/             # 运营人员操作
│   │   ├── index.md          # 运营操作概述
│   │   ├── content-management.md
│   │   └── order-handling.md
│   └── customer-service/     # 客服操作
│       ├── index.md          # 客服操作概述
│       └── ticket-handling.md
├── faq.md                    # 常见问题
├── troubleshooting.md        # 问题排查
└── videos/                   # 视频教程链接
    └── index.md              # 视频教程目录
```

**编写要求**：
- ✅ 大量使用截图和步骤说明
- ✅ 使用序号列表描述操作步骤
- ✅ 突出关键按钮和字段
- ✅ 提供实际业务场景示例

## 📋 requirement/ - 需求管理（核心）

**维护者**：产品经理（PRD）、技术负责人（TDD）、测试工程师（TEST）  
**目的**：管理需求全生命周期，保证需求可追溯

```
requirement/
├── index.md                  # 需求管理说明
├── backlog/                  # 需求池
│   ├── index.md              # 需求池概述
│   ├── pending.md            # 待评估需求
│   ├── approved.md           # 已批准待排期
│   └── rejected.md           # 已拒绝需求
├── template/                 # 文档模板
│   ├── index.md              # 模板使用说明
│   ├── PRD-template.md
│   ├── TDD-template.md
│   └── TEST-template.md
├── YYYYMM-需求名称/          # 需求文件夹
│   ├── index.md              # 需求概述
│   ├── product/              # 产品需求
│   │   ├── PRD.md
│   │   └── attachments/      # 原型图、流程图
│   ├── technical/            # 技术设计
│   │   ├── TDD.md
│   │   ├── database.md       # 数据库设计
│   │   ├── api-design.md     # 接口设计
│   │   └── ui-design.md      # UI设计
│   ├── testing/              # 测试文档
│   │   ├── test-cases.md     # 测试用例
│   │   └── test-report.md    # 测试报告
│   └── review/               # 评审记录
│       ├── prd-review.md     # PRD评审记录
│       ├── tdd-review.md     # TDD评审记录
│       ├── code-review.md    # 代码审查报告
│       └── acceptance.md     # 验收记录
└── archives/                 # 历史需求归档
    ├── index.md              # 归档说明
    └── YYYY/
        └── index.md          # 年度归档概述
```

**命名规范**：
- 需求文件夹：`YYYYMM-需求简短名称`，如 `202602-用户登录注册`
- 需求简短名称：中文，不超过10个字，使用连字符分隔
- 评审记录：包含评审日期和结论

**文档版本管理**：
```markdown
---
version: 1.2.0
created: 2026-02-09
updated: 2026-02-15
author: 张三
status: draft | review | approved | development | testing | published
---
```

## 🔧 technical/ - 技术文档

**维护者**：技术团队  
**目的**：说明系统架构、接口设计、开发规范

```
technical/
├── index.md                  # 技术文档首页
├── architecture/             # 系统架构
│   ├── index.md              # 架构概览（含架构图）
│   ├── overview.md           # 架构详细说明
│   ├── system-design.md      # 系统设计
│   ├── tech-stack.md         # 技术栈
│   ├── frontend-architecture.md
│   ├── backend-architecture.md
│   └── security.md           # 安全设计
├── database/                 # 数据库文档
│   ├── index.md              # 数据库概览
│   ├── overview.md           # 数据库详细说明
│   ├── er-diagram.md         # ER图（使用Mermaid）
│   ├── tables/               # 表结构文档
│   │   ├── index.md          # 表结构概述
│   │   ├── user-tables.md    # 用户相关表
│   │   └── [module]-tables.md
│   ├── migrations.md         # 数据迁移记录
│   └── performance.md        # 性能优化
├── api/                      # 接口文档
│   ├── index.md              # 接口文档首页
│   ├── overview.md           # 接口概览
│   ├── authentication.md     # 认证鉴权
│   ├── modules/              # 按模块分类
│   │   ├── index.md          # 接口模块概述
│   │   ├── user-api.md
│   │   └── [module]-api.md
│   ├── error-codes.md        # 错误码说明
│   └── changelog.md          # 接口变更记录
├── development/              # 开发文档
│   ├── index.md              # 开发文档首页
│   ├── setup.md              # 开发环境搭建
│   ├── code-style.md         # 代码规范
│   ├── git-workflow.md       # Git工作流
│   ├── testing.md            # 测试规范
│   └── build-deploy.md       # 构建部署
├── third-party/              # 第三方集成
│   ├── index.md              # 第三方集成概述
│   ├── payment.md
│   ├── sms.md

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/liushoukun) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
