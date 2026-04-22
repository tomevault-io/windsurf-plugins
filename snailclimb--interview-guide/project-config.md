---
trigger: always_on
description: Spring Boot 4.0 + Java 21 + Spring AI + React 面试平台。写代码时必须遵守以下规则。
---

# AI Interview Platform 编码规范

Spring Boot 4.0 + Java 21 + Spring AI + React 面试平台。写代码时必须遵守以下规则。

---

## 一、项目结构

单模块 Gradle 项目，按功能分包：

```
interview.guide/
├── App.java                          # @SpringBootApplication + @EnableScheduling
│
├── common/                           # 通用基础能力
│   ├── annotation/                   #   @RateLimit（可重复注解，滑动窗口限流）
│   ├── aspect/                       #   RateLimitAspect（AOP + Redis Lua 限流）
│   ├── ai/                           #   StructuredOutputInvoker（结构化输出重试）
│   │                                 #   LlmProviderRegistry（多 LLM Provider 注册与缓存）
│   ├── async/                        #   AbstractStreamConsumer/Producer（Redis Stream 模板）
│   ├── config/                       #   配置类（CORS、S3、ObjectMapper、OpenAPI、LlmProvider）
│   ├── constant/                     #   CommonConstants、AsyncTaskStreamConstants
│   ├── exception/                    #   ErrorCode（10 个错误域 1xxx-10xxx）
│   │                                 #   BusinessException、RateLimitExceededException
│   ├── model/                        #   AsyncTaskStatus
│   └── result/                       #   Result<T>（统一响应包装）
│
├── infrastructure/                   # 技术基础设施
│   ├── export/                       #   PdfExportService（iText 8）
│   ├── file/                         #   文件解析（Tika）、存储（S3/RustFS）、校验、清洗
│   ├── mapper/                       #   MapStruct 映射器（Interview、Resume、KB、RagChat）
│   └── redis/                        #   RedisService、InterviewSessionCache
│
└── modules/                          # 业务模块（每个模块自包含 MVC 分层）
    ├── resume/                       #   简历管理：上传、解析、AI 评分、去重
    ├── interview/                    #   模拟面试：会话、AI 出题、答题评估、报告导出
    ├── knowledgebase/                #   知识库：文档上传、向量化（pgvector）、RAG 查询、聊天会话
    ├── interviewschedule/            #   面试日程：日历管理、AI 解析面试邀请
    └── voiceinterview/               #   语音面试：WebSocket 实时通话、ASR/TTS、多轮评估
```

**技术栈**：Spring Boot 4.0 / Java 21（虚拟线程）/ Spring AI 2.0 / JPA + PostgreSQL + pgvector / Redisson / Redis Stream / MapStruct / iText 8 / Apache Tika

**前端**：React 18 + TypeScript + Vite + TailwindCSS 4（`frontend/` 目录）

---

## 二、分层架构

```
Controller → Service → Repository
                ↕
          Infrastructure（RedisService、FileStorageService、PdfExportService）
```

### Controller 层

- 仅路由和委托，禁止业务逻辑
- RESTful 风格：`/api/{module}/{action}`
- 使用 `@RateLimit` 注解做限流（`@Repeatable`，每维度独立 count）
- 通过 `@Valid` + `@RequestBody` 校验请求

### Service 层

- 业务逻辑编排，合理拆分大 Service（如 `ResumeUploadService`、`ResumeParseService`、`ResumeGradingService`）
- 使用 `LlmProviderRegistry.getChatClientOrDefault(provider)` 获取 ChatClient（支持多 Provider）
- 异步任务通过 Redis Stream（`AbstractStreamProducer/Consumer` 模板）
- 所有业务异常使用 `BusinessException(ErrorCode.XXX, message)`，禁止 `RuntimeException`

### Repository 层

- Spring Data JPA，继承 `JpaRepository`
- 自定义查询用 `@Query` 或方法命名约定

---

## 三、JavaBean 后缀规则

| 后缀 | 用途 | 示例 |
|------|------|------|
| `XxxEntity` | JPA 持久化 | `ResumeEntity`、`InterviewSessionEntity` |
| `XxxDTO` | 跨层数据传输 | `ResumeListItemDTO`、`SessionResponseDTO` |
| `XxxRequest` | 前端请求体 | `CreateInterviewRequest`、`QueryRequest` |
| `XxxResponse` | 前端响应体 | `QueryResponse`、`SubmitAnswerResponse` |

- 不可变数据载体优先用 `record`（如 `CreateInterviewRequest`、`QueryRequest`）
- Entity 映射用 MapStruct（`@Mapper(componentModel = "spring")`）
- 简单场景可用 `BeanUtils.copyProperties`
- **禁止直接返回 Entity 给前端**

---

## 四、异常与错误码

### ErrorCode 分域规则

| 域 | 范围 | 示例 |
|----|------|------|
| 通用 | 1xxx | BAD_REQUEST(400)、NOT_FOUND(404) |
| 简历 | 2xxx | RESUME_NOT_FOUND(2001) |
| 面试 | 3xxx | INTERVIEW_SESSION_NOT_FOUND(3001) |
| 存储 | 4xxx | STORAGE_UPLOAD_FAILED(4001) |
| 导出 | 5xxx | EXPORT_PDF_FAILED(5001) |
| 知识库 | 6xxx | KNOWLEDGE_BASE_NOT_FOUND(6001) |
| AI 服务 | 7xxx | AI_SERVICE_TIMEOUT(7002) |
| 限流 | 8xxx | RATE_LIMIT_EXCEEDED(8001) |
| 面试日程 | 9xxx | INTERVIEW_SCHEDULE_NOT_FOUND(9001) |
| 语音面试 | 10xxx | VOICE_SESSION_NOT_FOUND(10001) |

### 异常处理规则

- 抛出：`throw new BusinessException(ErrorCode.XXX, "描述信息")`
- **禁止** `throw new RuntimeException(...)` —— 必须用 `BusinessException`
- 全局异常处理器 `GlobalExceptionHandler` 统一返回 HTTP 200 + `Result.error(code, message)`
- `catch (BusinessException e) { throw e; }` 保留业务异常原样抛出

---

## 五、限流组件

```java
// 每个 @RateLimit 对应一个维度，各自独立的 count/interval/timeUnit
@RateLimit(dimension = RateLimit.Dimension.GLOBAL, count = 10)
@RateLimit(dimension = RateLimit.Dimension.IP, count = 10)
public Result<QueryResponse> queryKnowledgeBase(...) { ... }
```

- 注解：`@Repeatable`，AOP 切面 `RateLimitAspect` 逐条执行单 key Lua 脚本
- Lua 脚本：`resources/scripts/rate_limit_single.lua`，滑动时间窗口
- Redis Key 设计：`ratelimit:{ClassName:MethodName}:dimension`（Hash Tag 分组）
- 维度：`GLOBAL`（全局限流）、`IP`（按 IP）、`USER`（按用户）

---

## 六、异步任务（Redis Stream）

使用 `AbstractStreamProducer` / `AbstractStreamConsumer` 模板：

```java
// 生产者
public class VectorizeStreamProducer extends AbstractStreamProducer<KnowledgeBaseTask> { ... }

// 消费者
public class VectorizeStreamConsumer extends AbstractStreamConsumer<KnowledgeBaseTask> { ... }
```

- 三个管道：知识库向量化、简历分析、面试评估
- 常量定义在 `AsyncTaskStreamConstants`
- 消费者实现 `processMessage()` 方法
- **失败重试**：最大 3 次，超过后标记 FAILED
- **实体删除**：异步处理前校验实体是否存在，不存在直接 ACK 丢弃

---

## 七、AI 服务调用

### LLM Provider

```java
// 统一通过 Registry 获取，支持多 Provider 路由

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Snailclimb/interview-guide](https://github.com/Snailclimb/interview-guide) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
