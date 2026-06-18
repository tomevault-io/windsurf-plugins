---
trigger: always_on
description: 本文件用于说明在本仓库中工作的代理应如何理解项目结构、定位代码以及验证修改。内容以当前源码为准，不以 README 的宣传性描述为准。
---

# AGENTS.md

本文件用于说明在本仓库中工作的代理应如何理解项目结构、定位代码以及验证修改。内容以当前源码为准，不以 README 的宣传性描述为准。

## 构建与验证

- 使用仓库根目录下的系统 Maven；仓库内没有 `mvnw`
- 全量构建：`mvn clean install`
- 全量打包：`mvn clean package`
- 仅编译：`mvn clean compile`
- 运行全部测试：`mvn test`
- 运行单模块测试：`mvn -pl koding-common -am test`
- 运行单个测试类：`mvn -pl koding-common -Dtest=ResponseTest test`
- 运行单个测试方法：`mvn -pl koding-common -Dtest=ResponseTest#success test`

当前 Maven 构建中没有单独的 lint 或 format 任务。修改后通常用 `compile`、`test`、`package` 或 `install` 进行验证。

## 提交规范

- 提交代码时，Git commit message 必须统一采用 Conventional Commits 规范，并使用全英文编写
- 统一格式：`<type>(<scope>): <subject>`
- 采用该规范的目的包括：提高 Commit 可读性、区分功能开发和 Bug 修复等类型、支持自动生成 Changelog、支持版本发布自动化、提高多人协作效率

## 仓库定位

- 这是一个 Maven 多模块框架仓库，根 `pom.xml` 的 `packaging` 为 `pom`
- 仓库不是一个可直接启动的业务服务，源码里没有 `main` 方法，也没有 `@SpringBootApplication`
- 根模块的职责主要是：
  - 聚合四个子模块
  - 统一管理依赖版本
  - 通过 `dependencyManagement` 暴露各子模块版本
- 当前源码中固定的基础版本：
  - Java 25
  - Spring Boot 4.0.4
  - Spring Cloud 2025.1.1
  - Spring Cloud Alibaba 2025.1.0.0

如果用户说“运行项目”，通常应理解为编译、打包、跑测试，或者在外部 Spring Boot 应用中集成这些 starter 进行验证，而不是直接启动本仓库。

## 主模块与职责

### `koding-common`

基础公共模块，也是其他 starter 的共享依赖。

主要职责：
- 统一响应模型：`Response<T>`、`PageResponse`
- 统一异常模型：`BaseExceptionInterface`、`BizException`、`ResponseCodeEnum`
- 全局异常处理：`GlobalExceptionHandler`
- 公共常量、枚举、校验器、日期与数字工具、JSON 工具

关键源码位置：
- `koding-common/src/main/java/com/koding/framework/common/response/Response.java`
- `koding-common/src/main/java/com/koding/framework/common/exception/GlobalExceptionHandler.java`
- `koding-common/src/main/java/com/koding/framework/common/util/JsonUtils.java`

需要注意：
- `Response<T>` 采用统一返回结构：`success`、`message`、`errorCode`、`data`
- `GlobalExceptionHandler` 已处理：
  - `BizException`
  - `MethodArgumentNotValidException`
  - `IllegalArgumentException`
  - 其他兜底 `Exception`
- `JsonUtils` 持有静态 `JsonMapper`，并允许被外部自动配置通过 `init(...)` 覆盖

涉及接口返回、异常编码、校验报错格式、JSON 工具行为的变更，通常应先从这个模块入手。

### `koding-spring-boot-starter-jackson`

负责提供框架默认的 Jackson 配置，并把 Spring 管理的 `JsonMapper` 接入 `koding-common` 中的静态 JSON 工具。

主要职责：
- 提供 `JsonMapper` Bean
- 关闭未知字段反序列化失败
- 关闭空 Bean 序列化失败
- 设定默认时区为 `Asia/Shanghai`
- 调用 `JsonUtils.init(objectMapper)`，让公共 JSON 工具与 Spring 配置保持一致

关键源码位置：
- `koding-spring-boot-starter-jackson/src/main/java/com/koding/framework/jackson/config/JacksonAutoConfiguration.java`
- `koding-spring-boot-starter-jackson/src/main/resources/META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`

需要注意：
- 这个模块当前实现的是“基础 JSON mapper 装配”，并没有源码体现 README 中提到的日期格式增强、枚举处理、空值过滤等更丰富能力
- 如果序列化行为异常，要同时检查 `JacksonAutoConfiguration` 和 `JsonUtils`

### `koding-spring-boot-starter-biz-context`

负责在 Web 请求与 Feign 调用之间透传用户上下文。当前实现范围很窄，核心只有 `userId`。

主要职责：
- Servlet 入口读取请求头中的 `userId`
- 将其写入 `LoginUserContextHolder`
- 在 Feign 出站请求中回写同名请求头
- 借助 `TransmittableThreadLocal` 支持线程池场景下的上下文传播

关键源码位置：
- `koding-spring-boot-starter-biz-context/src/main/java/com/koding/framework/biz/context/filter/HeaderUserId2ContextFilter.java`
- `koding-spring-boot-starter-biz-context/src/main/java/com/koding/framework/biz/context/holder/LoginUserContextHolder.java`
- `koding-spring-boot-starter-biz-context/src/main/java/com/koding/framework/biz/context/interceptor/FeignRequestInterceptor.java`
- `koding-spring-boot-starter-biz-context/src/main/java/com/koding/framework/biz/context/config/ContextAutoConfiguration.java`
- `koding-spring-boot-starter-biz-context/src/main/java/com/koding/framework/biz/context/config/FeignContextAutoConfiguration.java`

实际链路：
1. `HeaderUserId2ContextFilter` 从请求头读取 `GlobalConstants.USER_ID`
2. 若存在，则写入 `LoginUserContextHolder`
3. 请求结束后在 `finally` 中清理 ThreadLocal
4. `FeignRequestInterceptor` 从 holder 读取 `userId`，写回 Feign 请求头

需要注意：
- 当前上下文模型只有 `userId`，没有租户、traceId、登录态对象等更复杂能力
- `LoginUserContextHolder` 底层是 `Map<String, Object>` + `TransmittableThreadLocal`
- `getUserId()` 最终返回 `Long`

### `koding-spring-boot-starter-biz-operationlog`

负责基于 AOP 的接口/方法操作日志记录。

主要职责：
- 提供 `@ApiOperationLog` 注解
- 通过 `ApiOperationLogAspect` 对标注方法进行环绕增强
- 记录方法描述、入参、出参、耗时

关键源码位置：
- `koding-spring-boot-starter-biz-operationlog/src/main/java/com/koding/framework/biz/operationlog/aspect/ApiOperationLog.java`
- `koding-spring-boot-starter-biz-operationlog/src/main/java/com/koding/framework/biz/operationlog/aspect/ApiOperationLogAspect.java`
- `koding-spring-boot-starter-biz-operationlog/src/main/java/com/koding/framework/biz/operationlog/config/ApiOperationLogAutoConfiguration.java`
- `koding-spring-boot-starter-biz-operationlog/src/main/resources/META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`

实际行为：
- 切点匹配 `@ApiOperationLog`
- 记录入参 JSON、描述信息、执行耗时、出参 JSON
- 依赖 `JsonUtils` 做参数和结果序列化

需要注意：
- 当前实现只有日志输出，没有异步投递、落库、消息队列、审计持久化
- 如果日志内容异常或序列化失败，需要同时检查本模块和 `koding-common` 的 `JsonUtils`

## 自动配置机制

三个 starter 都通过 `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` 注册自动配置，而不是通过组件扫描生效。

排查 starter “为什么没生效”时，优先检查：
1. 对应 starter 依赖是否被引入
2. `AutoConfiguration.imports` 是否包含目标自动配置类
3. 自动配置类是否真的声明了需要的 Bean

## 模块依赖关系

源码中的依赖关系很简单：

- `koding-common`：基础模块
- `koding-spring-boot-starter-jackson` -> 依赖 `koding-common`
- `koding-spring-boot-starter-biz-context` -> 依赖 `koding-common`
- `koding-spring-boot-starter-biz-operationlog` -> 依赖 `koding-common`

这意味着：

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [minmobugu/koding-framework](https://github.com/minmobugu/koding-framework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
