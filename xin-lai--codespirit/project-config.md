---
trigger: always_on
description: CodeSpirit 项目结构详细说明 - 完整的项目目录树和组件分类
---


# CodeSpirit 项目结构

## 完整目录树

```
Src/
├── ApiServices/                                        # API服务层
│   ├── CodeSpirit.AiCardsApi/                         # AI卡片API服务
│   ├── CodeSpirit.ApprovalApi/                        # 审批工作流API服务
│   ├── CodeSpirit.ConfigCenter/                       # 配置中心API
│   ├── CodeSpirit.ContentApi/                         # 内容管理API
│   ├── CodeSpirit.DigitalPartner.Plugins.ExamAnalyst/ # 考试分析插件（AI伙伴工具）
│   ├── CodeSpirit.ExamApi/                            # 考试系统API
│   ├── CodeSpirit.FileStorageApi/                     # 文件存储API
│   ├── CodeSpirit.IdentityApi/                        # 身份认证API
│   ├── CodeSpirit.MallApi/                            # 商城API
│   ├── CodeSpirit.MessagingApi/                       # 消息服务API
│   ├── CodeSpirit.PartnerApi/                         # 伙伴API（AI助手前端）
│   ├── CodeSpirit.PathfinderAgent/                    # Pathfinder代理（AI目标管理代理）
│   ├── CodeSpirit.PathfinderApi/                      # Pathfinder API（AI目标管理）
│   └── CodeSpirit.SurveyApi/                          # 问卷调查API
├── Components/                                         # 独立组件库
│   ├── CodeSpirit.Aggregator/                         # 数据聚合器组件
│   ├── CodeSpirit.AiFormFill/                         # AI表单智能填充组件
│   ├── CodeSpirit.Amis/                               # AMIS界面生成引擎
│   ├── CodeSpirit.Audit/                              # 审计追踪组件（含LLM审计）
│   ├── CodeSpirit.Authorization/                      # 权限管理组件（RBAC+ABAC）
│   ├── CodeSpirit.Caching/                            # 分布式缓存组件（多级缓存+分布式锁）
│   ├── CodeSpirit.Charts/                             # 智能图表组件
│   ├── CodeSpirit.ConfigCenter.Client/                # 配置中心客户端
│   ├── CodeSpirit.LLM/                                # 大语言模型集成组件
│   ├── CodeSpirit.Localization/                       # 多语言本地化组件
│   ├── CodeSpirit.Messaging/                          # 消息队列组件
│   ├── CodeSpirit.MultiTenant/                        # 多租户组件
│   ├── CodeSpirit.Navigation/                         # 导航组件
│   ├── CodeSpirit.OData/                              # OData查询组件
│   ├── CodeSpirit.PartnerSdk/                         # 伙伴SDK（AI助手工具SDK）
│   ├── CodeSpirit.PathfinderTools/                    # Pathfinder工具库
│   ├── CodeSpirit.PdfGeneration/                      # PDF生成组件
│   ├── CodeSpirit.ScheduledTasks/                     # 定时任务组件（分布式调度）
│   ├── CodeSpirit.Settings/                           # 设置管理组件
│   ├── CodeSpirit.Shared/                             # 组件共享库
│   ├── CodeSpirit.UdlCards/                           # UDL卡片组件
│   └── CodeSpirit.VectorSearch/                       # 向量搜索组件（AI语义搜索）
├── CodeSpirit.AppHost/                                 # Aspire应用宿主（启动项目）
├── CodeSpirit.Core/                                    # 核心框架定义
├── CodeSpirit.ServiceDefaults/                         # 服务默认配置
├── CodeSpirit.Shared/                                  # 全局共享库
└── CodeSpirit.Web/                                     # Web前端项目
```

## 项目分类说明

### API服务层 (14个服务)

**注意**: 项目数量可能随开发进度变化，此处列出当前主要服务。

#### 核心业务系统
- **IdentityApi**: JWT认证、用户管理、角色权限、部门组织
- **ExamApi**: 题库管理、考试系统、阅卷、统计分析
- **SurveyApi**: 问卷设计、数据收集、统计分析
- **ApprovalApi**: 审批流程、表单流转、工作流引擎

#### AI 增强系统
- **AiCardsApi**: AI卡片、智能生成
- **PartnerApi**: AI伙伴对话前端
- **DigitalPartner.Plugins.ExamAnalyst**: 考试分析AI插件
- **PathfinderApi**: AI驱动的目标管理系统
- **PathfinderAgent**: Pathfinder智能代理

#### 基础设施服务
- **ConfigCenter**: 配置中心、动态配置管理
- **FileStorageApi**: 文件存储、引用计数、生命周期管理
- **MessagingApi**: 消息服务、通知推送
- **ContentApi**: 内容管理
- **MallApi**: 商城系统

### 核心组件库 (多个组件)

**注意**: 组件数量可能随开发进度变化，此处列出主要组件。

#### 界面生成
- **Amis**: 零前端代码CRUD界面生成引擎
- **UdlCards**: UDL卡片组件库
- **Navigation**: 智能导航组件

#### AI 集成
- **LLM**: 大语言模型统一接口（OpenAI、通义千问、DeepSeek）
- **AiFormFill**: AI表单智能填充组件
- **VectorSearch**: 向量搜索、AI语义搜索
- **PartnerSdk**: AI助手工具开发SDK
- **PathfinderTools**: Pathfinder工具库

#### 权限审计
- **Authorization**: RBAC+ABAC混合权限模型
- **Audit**: 审计追踪组件（含LLM审计）

#### 多租户
- **MultiTenant**: 多租户数据隔离
- **Localization**: 多语言本地化

#### 性能优化
- **Caching**: 多级缓存（L1内存+L2Redis）、分布式锁
- **ScheduledTasks**: 分布式定时任务调度

#### 数据处理
- **Aggregator**: 数据聚合器、字段替换
- **Charts**: 智能图表、可视化
- **OData**: OData查询支持

#### 基础设施
- **Settings**: 设置管理组件
- **PdfGeneration**: PDF生成服务
- **ConfigCenter.Client**: 配置中心客户端
- **Messaging**: 消息队列、事件总线
- **Shared**: 组件共享库

## 典型API项目结构

```
CodeSpirit.ExamApi/
├── Configuration/              # API配置类
│   └── ExamApiConfiguration.cs
├── Controllers/                # API控制器
│   ├── QuestionsController.cs
│   └── ExamsController.cs
├── Data/                       # 数据访问层
│   ├── ExamDbContext.cs
│   ├── MySqlExamDbContext.cs
│   ├── SqlServerExamDbContext.cs
│   └── Configurations/         # 实体配置
├── Dtos/                       # 数据传输对象
│   ├── Question/
│   │   ├── CreateQuestionDto.cs
│   │   ├── UpdateQuestionDto.cs
│   │   └── QuestionDto.cs
│   └── Exam/
├── Services/                   # 业务服务
│   ├── IQuestionService.cs
│   └── QuestionService.cs
├── MappingProfiles/            # AutoMapper配置
├── Resources/                  # 多语言资源
│   ├── ExamDisplayResources.cs
│   ├── ExamDisplay.resx
│   └── ExamDisplay.en.resx
├── Migrations/                 # 数据库迁移
│   ├── MySql/
│   └── SqlServer/
└── Program.cs                  # 启动文件（仅2行代码）
```

## 开发流程

### 新建 API 服务
1. 创建项目并引用 `CodeSpirit.Shared`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xin-lai/CodeSpirit](https://github.com/xin-lai/CodeSpirit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
