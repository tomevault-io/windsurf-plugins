---
trigger: always_on
description: 本项目基于阿里巴巴 **COLA（Clean Object-Oriented and Layered Architecture）** 框架开发，支持 **DDD（领域驱动设计）** 和 **MVC（模型-视图-控制器）** 两种开发模式。
---

# 项目架构规范与设计原则

## 一、架构概述

本项目基于阿里巴巴 **COLA（Clean Object-Oriented and Layered Architecture）** 框架开发，支持 **DDD（领域驱动设计）** 和 **MVC（模型-视图-控制器）** 两种开发模式。

### 架构分层原则

```
┌─────────────────────────────────────────┐
│         Adapter Layer (适配层)          │  ← 对外接口，处理HTTP请求
├─────────────────────────────────────────┤
│       Application Layer (应用层)        │  ← 业务编排与流程控制
├─────────────────────────────────────────┤
│         Domain Layer (领域层)           │  ← 【仅DDD模式】核心业务逻辑
├─────────────────────────────────────────┤
│     Infrastructure Layer (基础设施层)   │  ← 技术实现，数据持久化
└─────────────────────────────────────────┘
```

**注意：当前项目暂不需要 Client 层（API 定义层）**

---

## 二、模式选择指南

### 2.1 何时使用 DDD 模式？

适用于**复杂的写场景**，具有以下特征：

- ✅ **复杂的业务逻辑**：包含多个业务规则、状态转换
- ✅ **需要领域模型**：业务实体包含行为和状态
- ✅ **多层次的业务编排**：需要协调多个领域对象
- ✅ **重要的业务流程**：核心业务功能，需要严格的业务规则

**DDD 模式示例场景：**
- 帖子发布（审核、权限、状态管理）
- 评论管理（审核、嵌套回复、通知）
- 用户注册（验证、角色分配、欢迎流程）
- 订单处理（状态流转、库存扣减、支付）

**DDD 模式分层结构：**
```
Adapter (Controller)
    ↓ 调用
Application (编排层)
    ↓ 调用
Domain (领域层)
    - Entity (E 后缀)：领域实体
    - ValueObject (V 后缀)：值对象
    - DomainService：领域服务
    - Gateway：领域网关接口
    ↓ 依赖接口
Infrastructure (实现层)
    - GatewayImpl：实现 Gateway 接口
    - Mapper：MyBatis-Plus 数据访问
```

### 2.2 何时使用 MVC 模式？

适用于**简单的业务逻辑或查询场景**，具有以下特征：

- ✅ **简单的 CRUD 操作**：直接的增删改查
- ✅ **查询场景**：数据展示、报表、列表查询
- ✅ **配置管理**：系统配置的读写
- ✅ **工具类服务**：辅助功能、数据转换

**MVC 模式示例场景：**
- 系统配置查询
- 简单的数据字典管理
- 用户信息查询
- 日志查询
- 统计数据获取

**MVC 模式分层结构：**
```
Adapter (Controller)
    ↓ 调用
Application (当做 Service)
    ↓ 直接调用
Infrastructure (Mapper)
    - Mapper：MyBatis-Plus 数据访问
    - PO：持久化对象
```

**注意：MVC 模式下，Application 层直接调用 Infrastructure 的 Mapper，无需 Domain 层**

---

## 三、项目结构

### 3.1 模块划分

#### 业务模块（nx-biz-*）
```
nx-biz-forum/                    # 论坛业务模块
├── nx-forum-adapter/            # 适配层（Controller）
├── nx-forum-app/                # 应用层（Application）
├── nx-forum-domain/             # 领域层（Domain Entity、Gateway）【仅DDD】
└── nx-forum-infrastructure/     # 基础设施层（Mapper、GatewayImpl）
```

#### 平台模块（nx-platform）
```
nx-platform/                     # 平台通用能力
├── nx-common/                   # 公共组件和工具类
└── nx-module-*/                 # 按业务域划分子模块
    ├── *-adapter/               # 适配层
    ├── *-app/                   # 应用层
    ├── *-domain/                # 领域层【可选】
    └── *-infrastructure/        # 基础设施层
```

### 3.2 包结构规范

```
com.leyuz.{module}.{domain}/
├── {domain}Application.java           # 应用服务
├── {domain}Controller.java            # 【adapter包】控制器
├── domain/                             # 【仅DDD】领域包
│   ├── {Domain}E.java                 # 领域实体
│   ├── {Domain}V.java                 # 值对象
│   ├── service/{Domain}DomainService  # 领域服务
│   └── gateway/{Domain}Gateway        # 领域网关接口
├── dto/                                # 数据传输对象
│   ├── {Domain}Cmd.java               # 命令对象
│   ├── {Domain}Query.java             # 查询对象
│   └── {Domain}VO.java                # 视图对象
└── mybatis/                            # 【infrastructure包】数据访问
    ├── I{Domain}Service.java          # Mapper 接口
    └── {Domain}PO.java                # 持久化对象
```

---

## 四、分层设计规范

### 4.1 Adapter 层（适配层）

**职责：** 处理 HTTP 请求，参数校验，响应封装

**规范：**
- 使用 `@RestController` 注解
- 类名以 `Controller` 结尾
- 方法命名规范：
  - 查询：`query*` / `list*` / `get*`
  - 创建：`create*` / `add*`
  - 更新：`update*` / `modify*`
  - 删除：`delete*` / `remove*`
- 统一使用 `SingleResponse` / `MultiResponse` 包装响应
- 入参使用 `*Cmd` / `*Query` 命名
- 使用 `@Operation` 注解文档化 API
- 使用 `@Schema` 注解提供参数说明

**示例：**
```java
@RestController
@RequiredArgsConstructor
@Tag(name = "Thread", description = "帖子管理")
public class ThreadController {
    private final ThreadApplication threadApplication;
    
    @PostMapping("/thread")
    @Operation(summary = "创建帖子")
    public SingleResponse<Void> createThread(
        @RequestParam Integer forumId,
        @RequestBody @Valid ThreadCmd threadCmd) {
        threadApplication.createThread(forumId, threadCmd);
        return SingleResponse.buildSuccess();
    }
    
    @GetMapping("/thread/{threadId}")
    @Operation(summary = "查询帖子详情")
    public SingleResponse<ThreadDetailVO> getThread(@PathVariable Long threadId) {
        return SingleResponse.of(threadApplication.getThreadForView(threadId));
    }
}
```

### 4.2 Application 层（应用层）

#### 4.2.1 DDD 模式下的 Application

**职责：** 业务编排、流程控制、权限校验、事务管理

**规范：**
- 使用 `@Service` 注解
- 类名以 `Application` 结尾
- 使用构造器注入（`@RequiredArgsConstructor`）
- 负责组织和调度领域对象
- 处理事务（`@Transactional`）
- DTO 与领域对象的转换
- 权限校验
- 不包含核心业务逻辑（应在 Domain 层）

**示例：**
```java
@Service
@RequiredArgsConstructor
@Slf4j
public class ThreadApplication {
    private final ThreadDomainService threadDomainService;
    private final ThreadGateway threadGateway;
    private final ForumPermissionResolver forumPermissionResolver;
    
    public void createThread(Integer forumId, ThreadCmd threadCmd) {
        // 1. 权限校验
        forumPermissionResolver.checkPermission(forumId, "thread:new");
        
        // 2. 构建领域对象
        ThreadE threadE = ThreadE.builder()
            .forumId(forumId)
            .subject(threadCmd.getSubject())
            .content(threadCmd.getContent())
            .build();
        
        // 3. 调用领域服务
        threadDomainService.save(threadE);
    }
    

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [walker8/nx-forum](https://github.com/walker8/nx-forum) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
