---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

Voglander 是基于 Spring Boot 3.5.3 + Java 17 的企业级视频监控平台，支持 GB28181、GT1078、ONVIF 等协议，提供设备管理、实时监控和视频流处理能力。整体采用严格的分层架构与统一的模板方法/装配器模式。

## 开发命令

### 构建和运行
```bash
mvn clean compile                          # 编译
mvn spring-boot:run -pl voglander-web       # 运行主应用（入口 ApplicationWeb）
mvn clean install -pl voglander-common      # 构建特定模块
mvn clean package -pl voglander-web         # 打包
```

### 测试
```bash
mvn test                                              # 全部测试（集成测试自动跳过或按需执行）
mvn test -Dtest=DeviceManagerTest                     # 单个测试类
mvn test -Dtest=DeviceManagerTest#testCreateDevice    # 单个测试方法
mvn test -Dspring.profiles.active=test                # 指定 profile

# 集成测试 Profile：显式运行需要外部服务的集成测试
mvn test -Pintegration-tests                          # 激活集成测试，服务不可用时测试失败

# Redis 集成测试：运行时探测 Redis，不可用则 Assumptions.assumeTrue 自动跳过
# 先启动：brew services start redis  或  docker run -p 6379:6379 -d redis
mvn test -Dtest=MediaNodeCacheIntegrationTest

# PostgreSQL 集成测试：类似模式，需要 PostgreSQL 运行
mvn test -Dtest=PostgreSQLIntegrationTest
```

> **测试集中放置**：所有测试（manager / integration / service / controller）统一在
> `voglander-web/src/test/java/io/github/lunasaw/voglander` 下实现，其他模块不建测试目录。
>
> **测试基类选择**：
> - **BaseTest**：同步测试，使用 `@Transactional` 自动回滚，适用于 Manager/Repository 集成测试
> - **BaseAsyncTest**：异步测试（含 `@Async`、`CountDownLatch`、并发操作），无 `@Transactional`，需在 `@AfterEach` 手动清理
> - **BaseE2eTest**：端到端测试，包含 SIP/GB28181 场景
>
> **服务可用性检测**：集成测试使用 `Assumptions.assumeTrue()` 检测外部服务（Redis/PostgreSQL），不可用时自动跳过。
>
> 详细测试指南见 `doc/1.0.9/TEST_EXECUTION_GUIDE.md`

### 覆盖率（JaCoCo 聚合）
```bash
./generate-coverage-report.sh   # clean → test → verify，自动打开报告
# 输出：voglander-coverage-report/target/site/jacoco-aggregate/index.html
```

### 数据库
- 默认 **SQLite**（`app.db` 自动创建），测试用 `test-app.db` + `schema-sqlite.sql`
- 可选 **MySQL**：建库 `voglander` → 执行 `sql/voglander.sql` → 改 `application-repo.yml`
- 可选 **PostgreSQL**：建库 `voglander` → 执行 `sql/voglander-postgresql.sql` → 改 `application-repo.yml`

## 架构概述

### 多模块结构与依赖
```
voglander-web         # REST 控制器、过滤器、拦截器；应用主模块，承载所有测试
voglander-manager     # 业务编排、复杂多服务协调（含 MediaSessionManager 等）
voglander-service     # 核心业务服务，extends IService<DO>，单表单一职责
voglander-repository  # 实体(DO)、Mapper、缓存
voglander-integration # 外部系统集成（GB28181/ZLM/Excel），统一 ResultDTO
voglander-client      # 外部服务客户端与 DTO
voglander-common      # 常量、枚举、异常、工具
voglander-test        # 测试配置与工具
voglander-coverage-report  # JaCoCo 聚合报告（聚合上述模块）
```
依赖方向：`web → manager → service → repository → common`；`integration/client → common`。

根包：`io.github.lunasaw.voglander`。

### 分层职责
- **Web**：REST 控制器、参数校验、Req/VO 格式转换（**入参必须用 Web 层 `*Req`，不得直接收 DTO**）
- **Manager**：复杂业务流编排、多服务协调（**对外方法参数/返回一律用 DTO，不暴露 DO**）
- **Service**：`extends IService<DO>`，单表核心逻辑
- **Repository**：持久化与缓存
- **Integration**：外部系统包装器，统一 `ResultDTO` 响应

#### 分层调用选择
- **复杂业务**（多表/复杂逻辑）：Controller → 专用 Service → 多个 Manager；该 Service 也可直接调 `IService` 基础方法
- **简单单表**：Controller 可直接调 `IService`，无需经 Manager
- 避免过度设计与不必要的层级嵌套

## Web 层 Controller 模板方法规范

Controller 对应 Manager 标准 CRUD，按优先级实现：
- **必须（核心模板）**：`add` / `update` / `get` / `deleteOne` / `deleteBatch` / `getPage`
- **可选（增强业务）**：含操作日志的 `createXxx` / `updateXxx` / `deleteXxx`
- **避免（专用细粒度）**：状态更新、Key 更新等通常不在 Controller 暴露

关键约定：
- 入参：`*CreateReq` / `*UpdateReq`（含 `@NotNull` 的 ID）/ `*QueryReq`，经 `WebAssembler.xxxReqToDto()` 转 DTO
- 出参：Manager 返回 DTO → `WebAssembler.dtoToVo()` → 前端
- 更新：必带主键 ID，走 `manager.updateById(id, updateDTO)`
- 查询：实现一个**全量分页条件查询**接口，由前端灵活组装条件
- 统一返回 `AjaxResult.success()` / `AjaxResult.error()`；分页包装为 `*ListResp`（含 `total` + `items`）
- 用 `@Valid` 校验；类加 `@Tag`，方法加 `@Operation`，参数加 `@Parameter`

```java
@PostMapping("/add")
public AjaxResult<Long> add(@Valid @RequestBody StreamProxyCreateReq req) {
    return AjaxResult.success(streamProxyManager.add(webAssembler.createReqToDto(req)));
}

@PutMapping("/update")
public AjaxResult<Long> update(@Valid @RequestBody StreamProxyUpdateReq req) {
    return AjaxResult.success(streamProxyManager.updateById(req.getId(), webAssembler.updateReqToDto(req)));
}
```

## 关键设计模式

- **Assembler**：层间数据转换（DTO ↔ DO ↔ VO）
- **Manager 模板方法**：统一 CRUD + 缓存/日志
- **Wrapper**：外部系统集成，统一异常处理与 `ResultDTO`
- **Template**：统一内部方法承载缓存/日志等横切关注点

### Manager 层设计与模板方法

每个 Manager 实现统一模板方法签名：
```java
public Long add(XxxDTO dto)
public Long update(XxxDTO queryDTO, XxxDTO updateDTO)   // 查询条件与更新内容分离
public Long updateById(Long id, XxxDTO updateDTO)        // 最常用
public XxxDTO get(XxxDTO dto)
public Boolean deleteOne(XxxDTO dto)
public Boolean deleteBatch(XxxDTO dto)
public Page<XxxDTO> getPage(XxxDTO dto, int page, int size)  // 默认按 createTime 降序
private void clearCache(Long id, String oldKey, String newKey)  // 主键 + 业务键双清理
```

规范：
- **统一入口**：所有增/改/删走内部统一方法，集中做参数校验、缓存清理、日志，禁止散落直接操作 DB
- **对外用 DTO**：公开方法参数/返回均为 DTO；内部可用 DO 但须 private；查询方法命名 `getXxxDTOByYyy`
- **默认值**：依赖数据库字段默认值，不在代码硬编码
- **UNIQUE 约束**：`saveOrUpdate` 前先按业务主键（如 `app+stream`）查现有记录，避免唯一约束冲突
- 历史暴露 DO 的方法标 `@Deprecated` 并提供 DTO 替代

#### LambdaQueryWrapper 查询构建标准（强制）
必须用带 `condition` 参数的链式方法，禁止手动 null 判断，也**禁止** `new LambdaQueryWrapper<>(entityObject)`（会带入 null 字段污染条件）：
```java

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lunasaw/voglander](https://github.com/lunasaw/voglander) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
