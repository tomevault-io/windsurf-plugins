---
trigger: always_on
description: > 基于项目最新架构文档（`docs/ARCHITECTURE.md`）和实际技术栈
---

# ZhiWei（知微）项目开发规范

> 基于项目最新架构文档（`docs/ARCHITECTURE.md`）和实际技术栈
> **最后更新**：2026-03

---

## 一、项目技术栈总览

### 后端（Spring Boot）
- **语言**: Java 22（优先使用 Record、Sealed Interface、Pattern Matching、Virtual Thread）
- **框架**: Spring Boot 3.5.3
- **AI 集成**: Spring AI 1.1.2（Advisor 模式、MCP 支持、结构化输出）
- **构建**: Maven 3.9.x
- **数据库**: SQLite 3.49+（WAL 模式、FTS5 全文索引）+ sqlite-vec 0.1.x（向量存储）
- **迁移**: Flyway 10.x

### 前端（Vue 3 SPA）
- **框架**: Vue 3.5+（Composition API + `<script setup lang="ts">`）
- **UI 组件库**: **Reka UI 2.x**（不是 shadcn-vue）
- **样式系统**: Tailwind CSS 4.x + CSS Variables
- **状态管理**: Pinia 3.x
- **路由**: Vue Router 4.x
- **图标库**: lucide-vue-next
- **构建工具**: Vite 6.x

---

## 二、后端开发规范

### 2.1 Java 代码风格

**必须使用 Java 22 特性**：
- **Record**：用于不可变数据传输对象（DTO）、事件封装
- **Sealed Interface/Class**：用于有限状态机、策略模式
- **Pattern Matching**：`instanceof` + switch 模式匹配
- **Virtual Thread**：IO 密集型操作优先使用

```java
// ✅ 正确：使用 Record 作为 DTO
public record ChatRequest(String sessionId, List<Message> messages) {}

// ✅ 正确：Sealed Interface 限制实现
public sealed interface AgentState permits IdleState, RunningState, WaitingState {
    AgentState transition(Event event);
}

// ✅ 正确：Pattern Matching
public String describe(Object obj) {
    return switch (obj) {
        case String s -> "字符串: " + s;
        case Number n -> "数字: " + n;
        case null -> "空值";
        default -> "未知类型";
    };
}
```

**禁止事项**：
- 禁止使用 Lombok `@Data`（改用 Record 或手动定义 getter/setter）
- 禁止使用 `public` 字段暴露数据（使用封装）
- 禁止魔法数字/字符串（使用常量枚举或配置类）

### 2.2 包结构规范

```
com.lifepilot/
├── llm/              # 多模型路由、熔断器、故障转移
├── agent/            # Agent 控制循环、状态机、上下文组装
├── tool/             # 工具契约、动态注册、执行管道
├── guardrail/        # 安全护栏策略
├── mcp/              # Model Context Protocol 客户端/服务器
├── memory/           # 四层记忆（工作/情景/语义/程序）
├── knowledge/        # 知识库、文档解析、检索
├── skill/            # Skill 注册、热加载、自扩展
├── interaction/      # CLI/Web/Channel 适配器
├── conversation/     # 对话历史、会话管理
├── datastore/        # 通用数据存储（Schema-Free JSON）
├── workflow/         # YAML 声明式工作流
├── sandbox/          # 代码执行沙箱
├── media/            # 多模态处理
├── sync/             # 外部数据源同步
├── eval/             # Agentic Evals 评估框架
├── multiagent/       # 多 Agent 协作
├── a2a/              # A2A 协议实现
├── marketplace/      # 插件市场
├── meta/             # 元能力
├── prompt/           # Prompt 模板管理
├── notification/     # 统一通知系统
├── observability/    # 轨迹记录、可观测性
└── config/           # 全局配置
```

### 2.3 命名与注释规范

**命名规则**：
- 类名/接口名：PascalCase（`AgentLoop`、`MemoryService`）
- 方法名/变量名：camelCase（`getSession`、`chatHistory`）
- 常量：SCREAMING_SNAKE_CASE（`MAX_RETRY_COUNT`）
- 包名：全小写（`com.lifepilot.agent`）

**注释规范**：
- **必须使用中文注释**
- 类和方法使用 Javadoc 风格注释，说明业务意图
- 复杂逻辑添加行内注释说明
- 日志使用中文，清晰说明操作上下文

```java
/**
 * Agent 控制循环，负责协调工具调用和 LLM 响应
 * 
 * 工作流程：
 * 1. 接收用户消息
 * 2. 组装上下文（记忆检索 + 对话压缩）
 * 3. 调用 LLM 生成响应
 * 4. 处理工具调用请求
 * 5. 返回结果或继续循环
 */
public class AgentLoop {
    /**
     * 执行单轮对话
     *
     * @param sessionId 会话 ID
     * @param userMessage 用户消息
     * @return Agent 响应
     */
    public AgentResponse execute(String sessionId, String userMessage) {
        log.info("开始处理会话 {} 的用户消息", sessionId);
        // ... 实现逻辑
    }
}
```

### 2.4 日志规范

- 使用 SLF4J + Logback
- 日志级别使用：
  - `ERROR`：业务异常需要干预
  - `WARN`：可恢复的异常或需要关注
  - `INFO`：关键业务流程（用户交互、工具调用、LLM 请求）
  - `DEBUG`：开发调试信息
- 禁止在日志中打印敏感信息（密码、Token、个人隐私）

```java
// ✅ 正确：结构化日志
log.info("用户 {} 开始新会话，会话ID: {}", userId, sessionId);
log.warn("LLM 调用失败，尝试第 {} 次重试", retryCount);
log.error("工具 {} 执行异常: {}", toolName, e.getMessage(), e);

// ❌ 错误：日志级别不当
log.info("收到请求");  // 太泛
log.error("连接数据库", e);  // 非关键错误用 WARN
```

### 2.5 数据库规范

**Flyway 迁移脚本**：
- 位置：`src/main/resources/db/migration/`
- 命名格式：`V{版本号}__{描述}.sql`
- 示例：`V57__create_notification_history.sql`

```sql
-- ✅ 正确：包含注释说明业务意图
-- 通知历史表：存储所有发送过的通知记录
-- 用于通知重发、未读统计、用户通知偏好设置
CREATE TABLE notification_history (
    id TEXT PRIMARY KEY,
    session_id TEXT NOT NULL,
    urgency TEXT NOT NULL CHECK (urgency IN ('HIGH', 'MEDIUM', 'LOW')),
    content TEXT NOT NULL,
    sent_at TEXT NOT NULL,
    read_at TEXT,
    created_at TEXT DEFAULT (datetime('now'))
);

-- 创建索引加速查询
CREATE INDEX idx_notification_session ON notification_history(session_id);
CREATE INDEX idx_notification_urgency ON notification_history(urgency);
```

**SQL 编码规范**：
- 关键字大写：`SELECT`、`FROM`、`WHERE`
- 表名/列名：snake_case
- 避免 `SELECT *`，明确列出需要的列
- 使用参数化查询防止 SQL 注入

### 2.6 API 设计规范

参考 `docs/API_STANDARD.md`，核心原则：

- RESTful 风格，资源命名使用名词复数
- 响应格式统一：`{ code, message, data }`
- 分页响应：`{ items: [], total, page, pageSize }`
- 错误响应：包含错误码和用户友好消息
- 列表接口支持排序和过滤参数

```java
// ✅ 正确：统一的 API 响应格式
public record ApiResponse<T>(int code, String message, T data) {
    public static <T> ApiResponse<T> success(T data) {
        return new ApiResponse<>(0, "success", data);
    }
    
    public static <T> ApiResponse<T> error(int code, String message) {
        return new ApiResponse<>(code, message, null);
    }
}
```

### 2.7 测试规范

- 单元测试：JUnit 5 + Mockito
- 集成测试：Spring Boot Test
- 测试方法名使用中文，描述测试场景
- 测试类放在 `src/test/java` 对应包下

```java
/**
 * AgentLoop 单元测试
 */
class AgentLoopTest {
    
    @Test
    void 应该正确处理用户消息并返回响应() {
        // given
        String sessionId = "test-session";
        String userMessage = "帮我创建一个待办";
        
        // when

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ntygod/ZhiWei](https://github.com/ntygod/ZhiWei) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
