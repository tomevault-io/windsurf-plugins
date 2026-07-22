---
trigger: always_on
description: 本文件为 Claude Code (claude.ai/code) 在本仓库中工作时提供指引。
---

# CLAUDE.md

本文件为 Claude Code (claude.ai/code) 在本仓库中工作时提供指引。

## 概述

LiteFlow（v2.16.0）是一个轻量级规则引擎框架，用于复杂的组件化业务编排。它通过 DSL 驱动工作流，支持热重载和 11 种脚本语言。项目面向 Java 8+（最高支持到 JDK 25），拥有 2000+ 测试用例。

**官方文档**：https://liteflow.cc/pages/5816c5/

## 命令

### 构建
```bash
# 构建整个项目（模块集由 JDK 版本自动选择 —— 见下方说明）
mvn clean package -DskipTests

# 带测试构建
mvn clean package

# 构建指定模块
mvn clean package -DskipTests -pl liteflow-core

# 发布生产模块（手动选择 profile，不会自动激活）
mvn clean package -DskipTests -P release-on-8     # JDK 8 发布产物
mvn clean package -DskipTests -P release-on-17     # react-agent + spring-boot4-starter
```

**JDK 驱动的模块选择（重要）：** 根 `pom.xml` 中没有顶层的 `<modules>` 块 —— 构建的模块列表完全由按 JDK 激活的 profile 提供：
- `compile-8-to-16`（在 JDK `[1.8,17)` 上激活）：core、scripts、rules、spring-boot-starter、spring、solon、testcase-el、el-builder、benchmark。
- `compile-17+`（在 JDK `[17,)` 上激活）：以上全部，**外加** `liteflow-react-agent` 和 `liteflow-spring-boot4-starter`。

因此 `liteflow-react-agent` 和 `liteflow-spring-boot4-starter` 只有在 JDK 17+ 上构建时才会被纳入。这两个模块以 `maven.compiler.target=17` 编译，但根 pom 注释指出 agentscope-java（react-agent 背后的引擎）**运行时需要 Java 21+** —— 本仓库当前的开发 JDK 为 21。`release-on-8` / `release-on-17` 这两个 profile 没有 `<activation>`，必须用 `-P` 手动选择。

### 运行测试
```bash
# 运行所有测试
mvn test

# 运行指定模块的测试（liteflow-testcase-el/ 下有 30+ 个测试模块）
mvn test -pl liteflow-testcase-el/liteflow-testcase-el-springboot

# 运行单个测试类
mvn test -pl liteflow-core -Dtest=FlowExecutorTest

# 运行指定的测试方法
mvn test -pl liteflow-core -Dtest=FlowExecutorTest#testExecute
```

### 其他命令
```bash
# 依赖树
mvn dependency:tree

# 查看模块结构
ls liteflow-*/pom.xml
```

## 高层架构

### 核心执行模型

**FlowExecutor** → **Chain** → **Condition Tree（条件树）** → **Node Components（节点组件）**

1. **FlowExecutor**：执行工作流的入口（`execute2Resp(chainId, param)`）
2. **FlowBus**：所有 chain 和 node 的中央元数据注册中心（线程安全，支持热重载）
3. **Chain**：由 EL 表达式组成的命名工作流，编译为一棵 Condition 树
4. **Slot/DataBus**：基于 slot 池的线程安全上下文管理（`slotSize` 可配置）
5. **NodeComponent**：所有业务逻辑组件的基类

### 关键架构模式

#### 1. 两阶段解析
为处理循环依赖，chain 的构建分两个阶段进行：
- **阶段 1**：注册 chain ID（创建占位 chain）
- **阶段 2**：通过 EL 解析构建完整的条件树

#### 2. EL 表达式语言
使用 QLExpress 解析声明式工作流。示例算子：
- **THEN(a, b, c)**：顺序执行
- **WHEN(a, b, c)**：并行执行（异步）
- **IF(condition, trueNode, falseNode)**：条件分支
- **SWITCH(selector).to(a, b, c)**：多路分支
- **FOR(count).DO(loop)**：固定次数循环
- **WHILE(condition).DO(loop)**：条件循环
- **ITERATOR(iterator).DO(loop)**：迭代器循环
- **RETRY(node).times(3).forException(Ex.class)**：重试机制
- **CATCH(node).DO(handler)**：异常处理
- **TIMEOUT(node).time(1000)**：执行超时（毫秒）
- **PRE(a, b)**：前置条件（始终在 chain 之前执行，出错也会执行）
- **FINALLY(a, b)**：finally 条件（始终在 chain 之后执行）
- **AND(a, b)**、**OR(a, b)**、**NOT(a)**：用于 IF 条件的布尔逻辑
- **node.tag("t")**、**.data("k","v")**、**.id("id")**：节点级修饰符

规则在 XML/JSON/YML 中定义：
```xml
<chain name="myChain">
    THEN(a, WHEN(b, c).maxWaitSeconds(5), IF(e, f, g));
</chain>
```

#### 3. 组件类型
均继承 `NodeComponent`，但具有专门的行为：
- **NodeComponent**：标准同步组件（`process()` 方法）
- **NodeBooleanComponent**：为 IF 条件返回 boolean（`processBoolean()`）
- **NodeSwitchComponent**：为 SWITCH 路由返回 string（`processSwitch()`）
- **NodeIteratorComponent**：为 ITERATOR 结构提供迭代逻辑
- **NodeForComponent**：控制 FOR 循环行为
- **ScriptComponent**：基于脚本的组件（Groovy、JS、Python 等）

**声明式组件模式**：任何 Spring bean 都可以在不继承基类的情况下成为组件，方式是使用 `@LiteflowCmpDefine`（类级别，指定 `NodeTypeEnum`）和 `@LiteflowMethod`（方法级别，映射到 `LiteFlowMethodEnum`）。这样可以摆脱类继承层级的约束。

**组件生命周期钩子**（在 NodeComponent 子类中覆写，或通过 `@LiteflowMethod` 实现）：
- `isAccess()` – 执行前的准入检查；返回 `false` 则跳过
- `beforeProcess()` / `afterProcess()` – 每个组件的前/后置钩子
- `onSuccess()` / `onError()` – 结果回调
- `isContinueOnError()` – 当该节点失败时 WHEN 是否继续
- `isEnd()` – 标记该节点之后应停止 chain
- `rollback()` – 失败时按逆序调用

**`@FallbackCmp`**：标注一个兜底组件，当主组件不存在或抛异常时启用。

#### 4. 基于 Slot 的上下文
线程安全的执行上下文：
- `DataBus.offerSlot(chainId)` 从池中获取一个 slot
- Slot 包含执行元数据、上下文 bean 和步骤跟踪
- `DataBus.releaseSlot(slotIndex)` 将 slot 归还到池
- 池大小通过 `slotSize` 属性配置

#### 5. 解析模式策略
三种模式（`ParseModeEnum`）：
- **PARSE_ALL_ON_START**：启动时解析所有 chain（默认）
- **PARSE_ONE_ON_FIRST_EXEC**：每个 chain 在首次使用时惰性解析（启动更快）
- **PARSE_ALL_ON_FIRST_EXEC**：在任意 chain 首次执行时解析全部 chain

### 模块结构

#### 核心模块
- **liteflow-core**：核心引擎（FlowExecutor、FlowBus、DataBus、Slot、Condition 系统、组件模型）
- **liteflow-el-builder**：基于 QLExpress 的编程式 chain 构建器 API

#### 规则源插件（`liteflow-rule-plugin/` 下 6 种实现）
- **liteflow-rule-zk**：ZooKeeper 配置源
- **liteflow-rule-sql**：SQL 数据库配置源
- **liteflow-rule-nacos**：Nacos 配置中心
- **liteflow-rule-etcd**：etcd 配置源
- **liteflow-rule-apollo**：Apollo 配置中心
- **liteflow-rule-redis**：Redis 配置源

#### 脚本插件（`liteflow-script-plugin/` 下 11 种语言）
- **liteflow-script-groovy**：Groovy 脚本
- **liteflow-script-javascript**：Rhino JavaScript（JSR223）
- **liteflow-script-graaljs**：GraalVM JavaScript
- **liteflow-script-qlexpress**：阿里 QLExpress
- **liteflow-script-python**：Jython（JVM 上的 Python）
- **liteflow-script-lua**：LuaJ
- **liteflow-script-aviator**：Aviator 表达式语言
- **liteflow-script-java**：Janino（Java 编译器）
- **liteflow-script-javax**：JSR223 标准 Java 编译器
- **liteflow-script-javax-pro**：基于 Liquor 的 Java 编译器（增强版）
- **liteflow-script-kotlin**：Kotlin 脚本

#### 框架集成
- **liteflow-spring**：Spring 框架集成（组件扫描、bean 生命周期、AOP）
- **liteflow-spring-boot-starter**：Spring Boot 2/3 自动配置，使用 `@ConfigurationProperties`
- **liteflow-spring-boot4-starter**：Spring Boot 4 starter（仅 JDK 17+ —— 在 `compile-17+` profile 下构建）
- **liteflow-solon-plugin**：Solon 框架集成（Spring 的轻量替代方案）


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dromara/liteflow](https://github.com/dromara/liteflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
