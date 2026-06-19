---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

forge-admin 是一个基于 RBAC 的企业级后台管理系统，采用 monorepo 结构，前后端分离，已通过 GB/T 22239-2019 二级等保技术改造。

**技术栈：**
- 前端：Vue 3.4 + TypeScript + Element Plus + vxe-table + Pinia + Vite 5
- 后端：Spring Boot 3.2.0 + MyBatis Plus 3.5.7 + MySQL + Redis + Flowable 7.0.1
- 认证：JWT Token（访问令牌 2 小时，刷新令牌 7 天）
- 加密：AES-256-GCM（敏感字段）+ BCrypt（密码哈希）+ jasypt（配置文件）
- API 文档：Knife4j，地址 `/api/doc.html`

## 关键配置

| 服务 | 端口 | 路径 |
|------|------|------|
| 前端开发 | 3003 | `apps/forge-web` |
| 小程序开发 | - | `apps/forge-miniapp` |
| 后端 API | 8181 | `apps/forge-server` |
| 上下文路径 | - | `/api` |

**数据库：** MySQL `forge_admin`，localhost:3306 | **Redis：** localhost:6379（无密码） | **Java：** 21 | **Node：** 22.9.0 | **pnpm：** 8.15.4

## 开发命令

### 前端（在 `apps/forge-web` 目录下）
```bash
pnpm install    # 安装依赖
pnpm dev        # 启动开发服务器（端口 3003）
pnpm build      # 生产构建（含类型检查）
pnpm lint       # 运行 ESLint
pnpm test       # 运行 vitest 单元测试
```

### 小程序（在 `apps/forge-miniapp` 目录下）
```bash
pnpm install        # 安装依赖
pnpm dev:mp-weixin  # 微信小程序开发模式
pnpm build:mp-weixin # 微信小程序生产构建
```

### 后端（在 `apps/forge-server` 目录下）
```bash
mvn spring-boot:run -pl forge-server          # 启动开发服务器
mvn clean compile                              # 仅编译
mvn clean package -DskipTests                  # 打包 JAR（跳过测试）
mvn clean install -DskipTests                  # 安装到本地仓库
mvn test -pl forge-module-system/forge-module-system-biz  # 运行指定模块测试
mvn test -Dtest=ClassName -pl <module>         # 运行单个测试类
```

## 后端架构

### 多模块结构

后端是一个 14 模块的 Maven 项目，包名基类为 `com.forge`。

```
apps/forge-server/
├── pom.xml                          # 根聚合 POM（parent: spring-boot-starter-parent:3.2.0）
├── forge-dependencies/              # BOM 版本管理（纯 POM，无 Java 代码）
├── forge-framework/                 # 框架层
│   ├── forge-common/                # 纯公共：注解、枚举、异常、响应、JSON 序列化、工具类
│   ├── forge-spring-boot-starter-mybatis/   # MyBatis Plus 配置、数据权限框架
│   ├── forge-spring-boot-starter-redis/     # Redis 配置、缓存配置
│   ├── forge-spring-boot-starter-security/  # JWT、OAuth2、JustAuth 社交登录
│   └── forge-spring-boot-starter-web/       # Jackson、全局异常、WebSocket、Excel、限流
├── forge-module-system/             # 系统模块
│   ├── forge-module-system-api/     # 实体 + DTO（82 个文件）
│   └── forge-module-system-biz/     # Controller/Service/Mapper（含 auth、quartz）
├── forge-module-workflow/           # 工作流模块
│   ├── forge-module-workflow-api/   # 实体 + DTO（44 个文件）
│   └── forge-module-workflow-biz/   # Controller/Service/Mapper + Flowable 集成
└── forge-server/                    # Spring Boot 启动入口（ForgeAdminApplication.java）
```

**模块依赖关系：**
```
forge-server ← system-biz, workflow-biz
workflow-biz ← workflow-api, system-api, starters, flowable
system-biz ← system-api, starters, quartz, justauth
starter-security ← forge-common, system-api
starter-mybatis ← forge-common
starter-web ← forge-common
starter-redis ← forge-common
```

### 模块分层模式

每个业务模块遵循：`controller/` → `service/`（接口 + impl）→ `mapper/` → 实体在 `api/` 模块，`dto/` 在 `api/` 模块。

**包结构：**
- `com.forge.common` — 公共（annotation, enumeration, exception, json, response, utils）
- `com.forge.framework.mybatis` — MyBatis 配置、数据权限（permission/）
- `com.forge.framework.redis` — Redis 配置
- `com.forge.framework.security` — JWT、OAuth2 配置
- `com.forge.framework.web` — Web 配置、全局异常、WebSocket
- `com.forge.modules.system` — system + auth + quartz
- `com.forge.modules.workflow` — 工作流

### 横切关注点

- `@OperationLog(title, businessType)` — 审计日志（starter-web 中）
- `@DataPermission(deptAlias, userAlias)` — SQL 级数据范围过滤（starter-mybatis 中，5 种范围类型）
- `@RateLimiter(keyType, time, count)` — Redis 令牌桶限流（starter-web 中）
- `@Cacheable/@CacheEvict` — Redis 缓存（缓存名：dictData, dictType, sysConfig, userInfo, menu, dept）
- `@Valid @RequestBody` — Jakarta DTO 校验
- `@EncryptField` — 敏感字段自动加解密（starter-mybatis 中，配合 `EncryptTypeHandler`，使用 AES-256-GCM）
- `@XssIgnore` — 跳过 XSS 过滤（starter-web 中，标注在 Controller 方法或类上）

### 等保二级安全改造

系统已完成 GB/T 22239-2019 二级等保技术改造，关键能力：

- **密码策略**：`PasswordPolicyProperties` 控制复杂度（8-32位、大小写+数字+特殊字符）、90天有效期、5条历史校验、BCrypt 强度=12
- **登录安全**：`LoginAttemptService` 实现失败锁定（5次→15分钟），`CaptchaService` 图形验证码，单点登录通过 `LoginUserSessionService.kickOutUserSessions` 实现（同步清理 refreshToken）
- **敏感数据加密**：`CryptoUtils`（AES-256-GCM，`ENCv1:` 前缀），`@EncryptField` + `EncryptTypeHandler` 自动加解密
- **配置加密**：jasypt-spring-boot-starter，支持 `ENC()` 格式加密敏感配置
- **应用安全**：`XssFilter` 全局 XSS 过滤，Spring Security 安全响应头（CSP/HSTS/X-Frame-Options），`FileUploadValidator` 文件上传校验
- **审计脱敏**：`SensitiveDataMasker` 在 `OperationLogAspect` 中自动脱敏密码/手机号/邮箱/身份证

**安全配置前缀**：`forge.security.{captcha|password|login|upload}`，详见 `application.yml` / `application-prod.yml`。

**文档：**
- 合规说明：`apps/forge-server/docs/SECURITY-COMPLIANCE.md`
- 部署检查清单：`apps/forge-server/docs/DEPLOYMENT-CHECKLIST.md`
- 手动迁移 SQL：`apps/forge-server/docs/MANUAL-MIGRATION.sql`

### Spring Boot Starter 自动配置

每个 starter 通过 `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` 注册。修改 starter 类时需同步更新此文件。

### 数据库迁移

位置：`apps/forge-server/forge-server/src/main/resources/db/migration/`
命名：`V{YYYYMMDD}{seq}__{description}.sql`

## 前端架构（`apps/forge-web/src/`）

```
api/           # API 模块，带类型的请求/响应接口

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wlddhj/forge-admin](https://github.com/wlddhj/forge-admin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
