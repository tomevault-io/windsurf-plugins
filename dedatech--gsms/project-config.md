---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Language Preference
**IMPORTANT: Always respond in Chinese (中文) for this project.** Use Chinese for all explanations, comments, and communications unless specifically requested otherwise.

## Git Commit 约定
**重要：当用户请求 Git 提交时，必须使用 git-commit-assistant agent**

当用户提到以下任一关键词时，**不要直接执行 git 命令**，而是使用 Task tool 调用 git-commit-assistant agent：
- `git commit`
- `commit` + 代码/文件
- `提交代码`
- `committer`（及其拼写变体）

**工作流程：**
1. 识别用户的提交请求
2. 调用：`使用 Task tool 调用 git-commit-assistant agent`
3. Agent 会分析变更并生成提交信息
4. 等待用户确认后再执行提交
5. **不要自动执行 git push**，除非用户明确要求

## 开发协作约定

### 服务启动分工
- **Claude 负责：** 前端服务（`npm run dev`，端口 3000）
- **用户负责：** 后端服务（`mvn spring-boot:run`，端口 8080）
- **特殊情况：** 只有当用户遇到无法解决的后端问题时，才会将后端服务交给 Claude 启动

### 端口约定（固定，不可更改）
- 前端：**3000**
- 后端：**8080**
- ⚠️ 严禁更换其他端口（因 CORS 跨域配置限制）

### 后端热部署最佳实践

**正确做法：使用 Spring Boot DevTools**
```yaml
# application.yml 已配置
spring:
  devtools:
    restart:
      enabled: true  # 启用热部署
      additional-paths: src/main/java
      exclude: static/**,public/**
    livereload:
      enabled: true
```

**优势：**
- ✅ 修改 Java 代码后自动重启应用
- ✅ 无需手动启停服务
- ✅ 高效便捷，开发体验好

**错误做法（严禁）：**
- ❌ 查端口 → 杀进程（效率低）
- ❌ 手动停止 → 重新启动（浪费时间）

**使用方式：**
1. 启动后端：`cd backend && mvn spring-boot:run`
2. **修改 Java 代码后，必须先编译：`mvn compile`**
3. DevTools 检测到 class 文件变化，自动触发重启
4. 查看日志确认重启成功

**⚠️ 关键点：**
- DevTools 监控的是 **编译后的 class 文件**，不是源代码
- 修改代码后**必须执行 `mvn compile`** 才能触发热部署
- 不可跳过编译步骤，否则不会重启

**IDE 支持：**
- IntelliJ IDEA：修改文件后自动编译，无需手动操作
- VS Code：安装 Spring Boot Extension Pack，自动编译
- 命令行（Claude 使用）：手动执行 `mvn compile` 触发热部署

## 快速开始

**环境要求：**
- JDK 8+, Maven 3.6+, Node.js 18+
- MySQL 8.0+ (创建数据库: `CREATE DATABASE gsms CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;`)

**启动后端（端口 8080）：**
```bash
cd backend
mvn spring-boot:run
# 或使用环境变量:
DB_USERNAME=root DB_PASSWORD=your_password mvn spring-boot:run
```

**启动前端（端口 3000）：**
```bash
cd frontend
npm install
npm run dev
```

**Docker 快速启动：**
```bash
# 启动所有服务（MySQL + 后端 + 前端）
docker-compose up -d
```

**测试账号：**
- `admin` / `Admin123` - 管理员
- `zhangsan03` / `Admin123` - 普通用户

**API 文档：** http://localhost:8080/swagger-ui.html

---

## 项目概述

TeamMaster（统领工时管理平台）是一个面向研发团队的轻量级工时管理系统。这是一个基于 Spring Boot 的应用，采用标准三层架构结合DTO模式，具有清晰的分层结构。

**核心功能模块：**

1. **工时管理** - 项目、迭代、任务、工时记录的完整管理
2. **用户权限管理（RBAC）** - 基于角色的访问控制系统 ✨
   - 用户管理（CRUD + 角色分配 + 启用/禁用）
   - 角色管理（CRUD + 权限分配）
   - 权限管理（CRUD + 权限查询）
   - 三级权限控制（路由级 + 按钮级 + 数据级）
   - 用户注册流程（默认禁用，需管理员审核）
3. **统计报表** - 首页看板、项目统计、用户统计、部门统计

**项目结构（Monorepo）：**

```
gsms/
├── backend/          # Spring Boot 后端（端口 8080）
├── frontend/         # Vue 3 前端（端口 3000）
├── docs/            # 项目文档
├── deployment/      # 部署配置
└── TODO.md          # 待办事项
```

**技术栈：**

**后端：**

- Java 8 + Spring Boot 2.7.0
- MyBatis-Plus 3.5.3.1 + MySQL 8.0
- JWT 认证 (jjwt 0.9.1)
- Maven 构建管理
- SpringDoc OpenAPI 1.7.0 API文档
- Flyway 数据库版本管理
- PageHelper 分页插件
- Java 8 Time API (LocalDateTime, LocalDate)

**前端：**

- Vue 3.4 + TypeScript 5.3 + Vite 5.0
- Element Plus 2.5 UI 组件库
- Vue Router 4 + Pinia 2.1
- Axios 1.6 HTTP 客户端

## 常用命令

### 后端构建和运行

```bash
cd backend

# 构建项目
mvn clean install

# 运行应用（开发环境）
mvn spring-boot:run

# 使用指定配置文件运行
mvn spring-boot:run -Dspring-boot.run.profiles=dev

# 使用环境变量覆盖数据库配置
DB_USERNAME=root DB_PASSWORD=your_password mvn spring-boot:run

# 打包
mvn clean package
```

### 前端构建和运行

```bash
cd frontend

# 安装依赖
npm install

# 运行开发服务器
npm run dev

# 构建生产版本
npm run build

# 预览生产构建
npm run preview

# 代码检查
npm run lint

# 代码格式化
npm run format
```

### Docker 部署

```bash
# 启动所有服务（MySQL、后端、前端）
docker-compose up -d

# 查看日志
docker-compose logs -f backend

# 停止服务
docker-compose down

# 重新构建并启动
docker-compose up -d --build
```

### 测试

```bash
# 运行所有测试
mvn test

# 运行指定测试类
mvn test -Dtest=UserControllerTest

# 运行指定测试方法
mvn test -Dtest=UserControllerTest#testGetUserById

# 运行所有Controller测试
mvn test -Dtest=*ControllerTest
```

### 数据库

```bash
# Flyway数据库迁移（启动时自动执行）
# 手动执行迁移：
mvn flyway:migrate

# 查看Flyway状态
mvn flyway:info

# 修复失败的迁移
mvn flyway:repair
```

## 架构设计

### 分层结构（标准三层架构）

```
controller/         # REST API层 - 处理HTTP请求、参数校验
├── dto/           # 数据传输对象（独立包）
├── service/       # 业务逻辑层
│   └── impl/      # 服务实现类
├── model/         # 领域层（注意：不是 domain/）
│   ├── entity/    # JPA实体（数据库模型）
│   ├── enums/     # 业务枚举（包括错误码）
│   └── errorcode/ # 错误码枚举
├── repository/    # 数据访问层（MyBatis Mapper接口）
└── infra/         # 基础设施层
    ├── common/    # 通用组件（Result、PageResult）
    ├── config/    # Spring配置类
    ├── exception/ # 自定义异常
    └── utils/     # 工具类
```

**⚠️ 重要：目录命名差异**

本项目使用 `model/` 而非 `domain/`：
- 实体位于：`model/entity/` （不是 `domain/entity/`）
- 枚举位于：`model/enums/` （不是 `domain/enums/`）
- 错误码位于：`model/errorcode/` （不是 `domain/errorcode/`）

**数据流：**

1. **Controller** 接收HTTP请求，使用 `@Valid` 校验，调用Service
2. **DTO** 对象在Controller和Service层之间传输数据
3. **Service** 实现业务逻辑，使用 `UserContext.getCurrentUserId()` 获取当前用户
4. **Repository**（MyBatis Mapper）处理数据库操作
5. **Model** 实体表示数据库表结构（位于 `model/entity/` 目录）
6. **Result<T>** 包装所有API响应为统一格式
7. **GlobalExceptionHandler** 统一异常处理，返回标准错误响应

### 架构模式

**标准三层架构 + DTO模式：**

- **Controller层**：处理HTTP请求和响应，参数校验
- **Service层**：实现业务逻辑和事务管理
- **Repository/Mapper层**：数据持久化操作

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dedatech/gsms](https://github.com/dedatech/gsms) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
