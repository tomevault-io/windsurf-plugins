---
trigger: always_on
description: > 轻量级 Java AI Agent 平台
---

# LightBot - AI Coding Guide

> 轻量级 Java AI Agent 平台
>
> Tech Stack: SpringBoot + SpringAI + Vue3

---

## 项目原则

| 原则 | 说明 |
|------|------|
| **模块化** | 每个功能域独立模块，边界清晰，可独立演进 |
| **AI Native** | 架构围绕 AI 能力设计，而非传统 CRUD 套壳 |
| **Java First** | 后端统一 Java，不混入 Python / Node |
| **渐进式演进** | 先单体后拆分，先简单后复杂，不提前设计 |
| **低耦合** | 模块间通过接口通信，禁止直接依赖实现类 |

---

## 架构原则

### 禁止事项

```text
❌ 禁止跨模块直接调用 Service
❌ 禁止 Controller 写业务逻辑
❌ 禁止循环依赖
❌ 禁止硬编码 Prompt
❌ 禁止直接依赖 OpenAI SDK / DashScope SDK
❌ 禁止在业务代码中直接 new RestTemplate 调用模型
❌ 禁止使用拼接SQL语句操作数据库（必须使用 MyBatis-Plus）
❌ 禁止在 Service 中直接操作中间件客户端（必须通过 Util 类）
❌ 禁止 lightbot-server 新增业务 Service、DTO/VO、tool/builtin、service/chat、workflow/*Executor*
```

### 必须遵守

```text
✅ 所有模型调用必须通过统一 AI Framework（SpringAI）
✅ 跨模块调用通过 Service 接口；跨域聚合使用 Port 模式（如 Dashboard）；禁止 import ...service.impl.*
✅ 所有 Prompt 必须模板化，存储于配置或数据库
✅ 所有外部依赖通过依赖注入，禁止静态方法调用
✅ 所有数据库操作必须使用 MyBatis-Plus（LambdaQueryWrapper / ServiceImpl）
✅ 所有中间件操作（MinIO/Redis/MQ等）必须封装为 Util 类
✅ 业务核心逻辑必须有注释说明意图
```

### 模块结构

> 架构边界详见 `docs/architecture/module-boundaries.md`

```
lightbot/
├── lightbot-common/           # Result、枚举、公共类型
├── lightbot-framework/        # Spring 配置、Redis/MinIO 等中间件 Util
├── lightbot-platform/         # 用户、任务、系统配置、API Key、Dashboard、日志
├── lightbot-ai/               # 模型工厂、Prompt、LLM Trace
├── lightbot-knowledge/        # RAG、文档、图谱、评测
├── lightbot-tool/             # Tool / MCP / Skill / SubAgent 定义与注册
├── lightbot-workflow/         # Workflow DSL、节点处理器、图校验
├── lightbot-agent/            # Agent 运行时（Chat、Workflow 执行、SubAgent）
├── lightbot-server/           # HTTP 入口（Controller、Config、拦截器）
└── lightbot-ui/               # Vue3 前端（独立目录，非 Maven 模块）
```

### 依赖规则

```text
common ← framework ← platform ← ai ← knowledge ← tool ← workflow ← agent ← server
```

```text
lightbot-server    → platform, ai, knowledge, tool, workflow, agent
lightbot-agent     → platform, workflow, knowledge
lightbot-workflow  → tool
lightbot-tool      → knowledge
lightbot-knowledge → ai
lightbot-ai        → framework, platform
lightbot-platform  → framework
lightbot-framework → common
```

**下层禁止依赖上层，同层禁止循环依赖。**

---

## 代码规范

### 包结构

```
com.lightbot.{module}/
├── controller/        # 接口层，只做参数校验和返回
├── service/           # 业务接口定义
│   └── impl/          # 业务实现类
├── entity/            # 数据库实体
├── dto/               # 数据传输对象
├── mapper/            # MyBatis-Plus Mapper 接口
├── util/              # 工具类（中间件封装：MinIO/Redis/MQ等）
├── config/            # 配置类
├── constant/          # 常量
├── enums/             # 枚举
└── exception/         # 异常定义
```

### DTO 规范

```java
/**
 * 命名：{业务名}DTO
 * 用途：服务间数据传输
 */
@Data
public class AgentCreateDTO {
    @NotBlank(message = "Agent名称不能为空")
    private String name;
    
    private String systemPrompt;
    
    private String modelId;
}
```

- DTO 只用于服务间传输，不暴露给前端
- DTO/VO 归属各业务模块，**server 不持有业务 DTO/VO**（详见 `docs/architecture/module-boundaries.md`）
- 使用 Jakarta Validation 注解做校验
- 字段类型使用包装类（Long / Integer / Boolean），不用基本类型

### VO 规范

```java
/**
 * 命名：{业务名}VO / {业务名}Response
 * 用途：返回给前端的数据
 */
@Data
public class AgentVO {
    private Long id;
    private String name;
    private String modelId;
    private LocalDateTime createTime;
}
```

- VO 不包含业务逻辑
- 时间字段统一使用 `LocalDateTime`
- 列表返回使用 `PageVO<T>` 包装

### Entity 规范

```java
/**
 * 命名：数据库表名驼峰化，不加 t_ 前缀
 * 表名：agent
 * 注意：PG保留字（如 user）需换名（如 users）
 */
@Data
@TableName("agent")
@Schema(description = "Agent表")
public class Agent {

    @TableId(type = IdType.ASSIGN_ID)
    @Schema(description = "主键ID")
    @JsonSerialize(using = ToStringSerializer.class)
    private Long id;

    @TableField("user_id")
    @Schema(description = "创建者ID")
    @JsonSerialize(using = ToStringSerializer.class)
    private Long userId;

    @TableField("name")
    @Schema(description = "Agent名称")
    private String name;

    @TableField("system_prompt")
    @Schema(description = "系统提示词")
    private String systemPrompt;

    @TableField(value = "config", typeHandler = JsonNodeTypeHandler.class)
    @Schema(description = "扩展配置")
    private String config;

    @TableField(value = "create_time", fill = FieldFill.INSERT)
    @Schema(description = "创建时间")
    private LocalDateTime createTime;

    @TableField(value = "update_time", fill = FieldFill.INSERT_UPDATE)
    @Schema(description = "更新时间")
    private LocalDateTime updateTime;

    @TableField("deleted")
    @TableLogic
    @Schema(description = "逻辑删除标记")
    private Integer deleted;
}
```

- 主键统一使用雪花算法 `IdType.ASSIGN_ID`
- **表名不加 `t_` 前缀**，直接使用业务名（如 `user`、`agent`、`knowledge`）
- **每个字段必须加 `@TableField`**：明确指定数据库列名
- **每个字段必须加 `@Schema`**：使用 OpenAPI 3 注解，替代 Javadoc 注释
- **JSONB 字段**：使用 `String` 类型 + `@TableField(value = "xxx", typeHandler = JsonNodeTypeHandler.class)`
- **type/status 字段必须使用 Java 枚举**（配合 `@EnumValue` + `@JsonValue`），不使用数据库枚举
- 必须包含 `createTime`、`updateTime`、`deleted` 字段
- 使用 `@TableLogic` 实现逻辑删除
- 如果遇到SQL更新，要放到整个文件夹根目录下的sql文件夹下，并且以日期-001.sql文件命名，如果存在则编号递增,再次注意！不是docs文件夹下！
- **Long ID 字段必须加 `@JsonSerialize(using = ToStringSerializer.class)`**：包括主键和所有外键字段（如 `userId`、`agentId`、`knowledgeId` 等），防止前端 JavaScript 精度丢失

### Service 规范

**必须遵循 Interface + ServiceImpl 模式：**

```java
/**
 * 接口定义（放在 service/ 包）
 * 继承 IService<Entity> 可获得 MyBatis-Plus 内置 CRUD 方法
 */
public interface AgentService extends IService<Agent> {
    Agent create(AgentCreateDTO dto);

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [finch04/LightBot](https://github.com/finch04/LightBot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
