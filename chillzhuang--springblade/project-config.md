---
trigger: always_on
description: 欢迎来到 SpringBlade 开源微服务版（master 分支）。本文档汇总了微服务拓扑、API/Service 分层、Feign 调用、网关与配置中心等常见约定，帮助 Claude Code 快速上手，产出与项目风格一致的代码。
---

# CLAUDE.md

欢迎来到 SpringBlade 开源微服务版（master 分支）。本文档汇总了微服务拓扑、API/Service 分层、Feign 调用、网关与配置中心等常见约定，帮助 Claude Code 快速上手，产出与项目风格一致的代码。

作为 AI 助手参与本项目开发时，你必须：

- 动手前先看一看目标服务及其 API 模块的现有实现，模仿它的结构与风格通常是最快的路径
- 需要外部知识时，优先查阅 Spring Boot、Spring Cloud、MyBatis-Plus、Nacos 等官方文档，而不是凭记忆作答
- 需求不够清晰时，先复述已知信息、抛出关键澄清问题，再动手
- 面对跨服务或跨模块改动，拆成可验证的小步骤逐步推进比一气呵成更稳妥

> 所有开发内容必须建立在深度思考过的基础之上，禁止机械生成与错误填充。
> 如果你已了解所有规范，请在用户第一次对话时说明："我已充分了解 SpringBlade 开源微服务版开发规范。"

---

## 1. 项目概述

SpringBlade 是 BladeX 平台的**开源微服务版本**（master 分支），基于 Spring Cloud + Spring Boot 构建，由 Nacos 做服务发现与配置中心，Sentinel 做熔断限流，Spring Cloud Gateway 做统一入口，依赖 blade-tool 核心库。

---

## 2. 项目架构

```
SpringBlade/                            # Maven 父工程（pom 聚合）
├── blade-auth/                         # 认证授权服务（多 granter：password/refresh/social/captcha）
├── blade-gateway/                      # Spring Cloud Gateway 网关（WebFlux + JWT 鉴权过滤器）
├── blade-common/                       # 公共模块（常量、启动器、工具、缓存键）
├── blade-ops/                          # 运维服务父模块
│   ├── blade-admin/                    #   Spring Boot Admin 监控
│   ├── blade-develop/                  #   代码生成（Velocity 模板）
│   ├── blade-report/                   #   报表服务（UReport2）
│   ├── blade-resource/                 #   资源 / OSS 管理
│   ├── blade-seata-order/              #   Seata 分布式事务演示：订单
│   └── blade-seata-storage/            #   Seata 分布式事务演示：库存
├── blade-service/                      # 业务服务父模块
│   ├── blade-system/                   #   系统管理（用户/角色/菜单/字典/租户/部门/岗位）
│   ├── blade-desk/                     #   工作台（通知公告）
│   ├── blade-log/                      #   日志服务（API/Error/Usual 日志落库）
│   └── blade-demo/                     #   业务演示模块
├── blade-service-api/                  # 业务契约父模块（Entity / VO / DTO / Feign）
│   ├── blade-system-api/               #   ISysClient（部门、租户、客户端信息）
│   ├── blade-user-api/                 #   IUserClient（用户核心信息）
│   ├── blade-desk-api/                 #   桌面 API 契约
│   ├── blade-dict-api/                 #   IDictClient（字典翻译）
│   ├── blade-scope-api/                #   数据权限契约
│   └── blade-demo-api/                 #   演示模块契约
├── doc/
│   ├── nacos/                          # Nacos 配置脚本（blade.yaml、各环境 datasource）
│   └── sql/                            # 多数据库建表 & 升级脚本
├── script/                             # Docker / FatJar / 服务脚本
└── pom.xml                             # 父 POM（blade-tool 依赖管理）
```

> **与开源 Boot 版的关键区别**：Cloud 为多模块 Maven 工程，存在 **API/Service 分离**，跨服务调用通过 Feign，Controller 路由**不带** `AppConstant` 前缀（网关按服务名路由）。

### 2.1 分层架构

每个业务域遵循 **API 模块 + Service 模块**双层划分：

| 层次 | 包路径 | 所在模块 |
| --- | --- | --- |
| Controller | `controller/` | Service 模块（`blade-service/*` 或 `blade-ops/*`） |
| Service / ServiceImpl | `service/` + `service/impl/` | Service 模块 |
| Mapper（接口 + XML 同包） | `mapper/` | Service 模块 |
| Wrapper | `wrapper/`（Entity → VO 转换） | Service 模块 |
| Excel 导入导出 | `excel/` | Service 模块 |
| Feign 实现类 | `feign/` | Service 模块（实现对应 API 模块的 Feign 接口） |
| Entity | `entity/` | **API 模块（`blade-*-api`）** |
| VO / DTO | `vo/` `dto/` | **API 模块** |
| Feign Client + Fallback | `feign/` | **API 模块** |

### 2.2 API / Service 分离原则

- **API 模块**（`blade-*-api`）：定义 Entity、VO、DTO、Feign 接口 + Fallback，供其他服务依赖；**不含**具体业务逻辑
- **Service 模块**（`blade-service/*`、`blade-ops/*`）：实现 Controller、ServiceImpl、Mapper、Wrapper 与 Feign 接口的服务端实现
- 跨服务调用**必须**通过 Feign Client，禁止直接依赖其他 Service 模块
- 公共常量与工具放在 `blade-common`

---

## 3. 技术栈

| 类别 | 技术 |
| --- | --- |
| JDK | Java 21 |
| 基础框架 | Spring Boot / Spring Cloud / Spring Cloud Alibaba / Maven |
| 核心库 | blade-core-bom、blade-core-cloud、blade-core-boot |
| 注册中心 / 配置中心 | Nacos |
| 熔断限流 | Sentinel（`spring-cloud-starter-alibaba-sentinel`） |
| 分布式事务 | Seata（`blade-starter-transaction`，见 `blade-seata-order`、`blade-seata-storage`） |
| RPC | OpenFeign（与 Sentinel 集成） |
| 网关 | Spring Cloud Gateway（WebFlux） |
| ORM | MyBatis-Plus + dynamic-datasource-spring-boot4-starter |
| 安全 | blade-core-secure（JWT + 可选 AES 加密 + SM2 国密登录） |
| 缓存 | BladeRedis（Protostuff 序列化，响应式 Gateway 侧用 Reactive Redis） |
| 数据权限 | blade-starter-datascope（DataScopeHandler 行级过滤） |
| 监控 | Spring Boot Admin（`blade-admin`）+ Actuator |
| 文档 | springdoc（各服务 OpenAPI 3 原生 UI） |
| 报表 | UReport2（`blade-report`） |
| 代码生成 | blade-starter-develop + Apache Velocity（`blade-develop`） |
| 验证码 | easy-captcha |
| JWT | jjwt-impl + jjwt-jackson |
| 数据库 | MySQL / PostgreSQL / Oracle / SQL Server / 达梦 / 人大金仓 / 崖山 |

---

## 4. 开发规范

### 4.1 编写新功能前

1. 先阅读目标服务中已有的类，理解其结构、命名和风格
2. 标准参考模块：`blade-service/blade-system`（Service 最完整）、`blade-service-api/blade-system-api`（API 契约最完整）、`blade-service/blade-desk` + `blade-desk-api`（最简样板）
3. 主动模仿现有代码风格，包括缩进（Java 用 Tab，YAML/JSON 用 2 空格）、注解顺序、Javadoc 格式
4. 新建 Java 源文件必须包含 Apache 2.0 许可证头部和 Javadoc 类注释（`@author Chill`）

### 4.2 编写完成后

1. 使用 `mvn clean compile -DskipTests` 编译验证（父工程根目录执行会编译所有子模块），编译不通过必须修复
2. 引入模块依赖前先检查循环依赖；**跨服务依赖必须经由 `blade-*-api` + Feign**，禁止 `blade-service/A` 直接依赖 `blade-service/B`
3. 编译通过后将测试交由用户执行，**不得自行执行任何测试**（包括 `mvn test`、启动 Nacos/服务、访问接口）
4. 除非用户明确要求，不应撰写示例或额外文档

### 4.3 代码生成

当需要生成 CRUD 全套代码（Entity、VO、Service、Controller、Wrapper、Mapper、Feign、建表语句等）时，优先使用 **`/blade-design`** skill。该 skill 可根据模块名、实体名和字段列表，自动生成符合 BladeX 框架规范的后端代码和多数据库建表语句，确保生成结果与项目风格完全一致。


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chillzhuang/SpringBlade](https://github.com/chillzhuang/SpringBlade) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
