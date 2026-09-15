---
trigger: always_on
description: 本文件是 `backend/unispeaking-server` 的强制开发约束，也是后续新增场景时的落位指南。
---

# UniSpeaking 后端架构与场景扩展规范

本文件是 `backend/unispeaking-server` 的强制开发约束，也是后续新增场景时的落位指南。
它描述当前代码的真实结构；若历史文档、示例代码与当前实现冲突，以本文件和编译通过的
稳定接口为准。

## 1. 技术基线

- Java 21，Spring Boot 4.0.7。
- Spring Web MVC、Spring WebSocket、Spring Security、JWT Resource Server。
- PostgreSQL、Flyway、MyBatis-Plus 3.5.17。
- Maven Wrapper：所有后端命令统一使用 `./mvnw`。
- JUnit 5、Mockito、Testcontainers。
- PostgreSQL 是业务真相来源；进程内状态、缓存、对象存储都不能替代数据库。

禁止在业务代码中引入第二套 ORM、第二套运行时 DDL 或重复的 HTTP/JSON 技术栈。

## 2. 场景运行时契约与职责

场景运行时保留的稳定面为：**两个具体公共父类（`SceneFlowService` / `EvaluationService`）+ 一个 AI 能力族（`AiProvider` 根 + 5 能力接口 + `AiProviderRegistry`）**，外加两个**文档化职责**（场景准备、会话生命周期）。只有返回类型稳定且确有具体逻辑可复用时才保留父类；`SceneService`/`SessionService` 基类已删除（零共享签名或仅 WS 传输约定）。

### 2.1 场景准备职责（Scene Preparation）

`SceneService` 基类已删除；每个场景 Service 都是直接实现类，`generate` 由具体类声明（如 `CustomSceneService.generate`）。场景准备仍须满足以下职责：

- 校验登录用户、资源归属和业务权限。
- 校验每日次数、配额或前置条件。
- 选择或生成场景内容。
- 组装 Prompt、音色和场景上下文。
- 持久化场景及其内容。
- 返回后续流程所需的 `sceneId` 和场景结果。

禁止：

- 在 `generate` 内启动 Session。
- 把场景准备工作推给会话层。

归属校验是具体 Service 私有逻辑或薄 `OwnershipPolicy` 组件（折叠错误 + 身份来源）。

### 2.2 `SceneFlowService`

位置：`service/scene/SceneFlowService.java`

```java
public class SceneFlowService<S> {
    public S start(String sceneId) { ... }
    public S current(String sceneId) { ... }
    public S next(String sceneId) { ... }
    public boolean isCompleted(String sceneId) { ... }
    public void clear(String sceneId) { ... }
}
```

职责：管理有阶段场景的全部流程状态。FreeChat 无阶段，不继承此父类。

有阶段的场景通过具体类继承它，例如 `CustomSceneFlowService extends
SceneFlowService<CustomStage>`，并显式 `@Override` 全部公共流转方法。

它既负责场景级阶段（例如 IELTS 的 Part 1/2/3），也负责场景专属的会话内子流程
（例如题目推进、Part 2 准备/作答、自定义对话目标）。这些子流程方法只声明在对应的
场景专用 Flow 类中，不得放入 Session Service。Flow 不负责生成内容、创建会话、保存
消息或评分。真实流程状态必须可以从数据库恢复；进程内状态机只负责运行时判断和转换。

### 2.3 会话生命周期（由 Component 承载）

`SessionService` 基类已删除。会话生命周期由 `component/session/SessionLifecycleManager` 承载，`SessionMessageDispatcher` 按 `SceneType` 将 WS 帧路由到各场景会话具体类。接受 WS 实时帧的 `FreeChatSessionService`、`CustomSessionService`、`IeltsSessionService` 必须各自声明 `startSession/addMessage/endSession` 生命周期形状（`addMessage` 由 `SessionMessageDispatcher` 消费）。

场景会话 Service 的职责：

- 基于已经准备好的 `sceneId` 启动会话。
- 创建 Realtime 会话、维护会话生命周期。
- 接收、验证并持久化消息。
- 结束会话和释放临时资源。

边界：

- 不调用 `AuthService` 重新完成场景权限或次数校验；这些已由场景生成阶段完成。
- 仍必须校验当前请求者是否拥有目标 `sceneId/sessionId`，防止越权访问。
- 不生成场景、不拼 Prompt、不选择题目、不推进业务阶段、不生成评分。
- 不得创建通用 `SessionService` 或 `SessionServiceImpl`。

会话查询与生命周期实现属于 `SessionLifecycleManager`。

### 2.4 `EvaluationService`

位置：`service/evaluation/EvaluationService.java`

```java
public class EvaluationService<R, D> {
    public DialogueTurnEvaluationResult evaluateTurn(DialogueTurnEvaluationCommand command) { ... }
    public R generateReport(String sceneId) { ... }
    public D getEvaluation(String sceneId) { ... }
}
```

职责：逐轮评分、场景报告生成和评分结果查询。它可读取 Session 消息和语音证据，但不能
管理会话生命周期或推进 Scene Flow。

FreeChat 当前不评分，因此不继承。Custom 与 IELTS 分别继承具体父类；不得创建通用
`EvaluationServiceImpl`。

支持评分的场景必须以具体 Evaluation 类继承公共父类，并显式 `@Override` 三个公共方法；
额外的历史、详情或专项评分方法放在具体子类中。

### 2.5 `AiProvider`

位置：`provider/AiProvider.java`

```java
public interface AiProvider {
    String exchangeRealtimeSdp(String offerSdp, String token);
    byte[] generateSpeechAudio(String text, String token);
    String executeLlmTask(String prompt, String token);
    String convertAudioToText(byte[] audio, String token);
    String evaluatePronunciation(String text, byte[] audio, String token);
}
```

职责：定义供应商无关的 AI 能力。业务代码只依赖 Provider 接口或 Registry；七牛 RTI、
七牛 MaaS、Qwen、Doubao、DeepSeek、MiniMax、讯飞等供应商差异全部留在
`infrastructure`。
Realtime 默认路由为七牛 RTI `qwen3.5-omni-plus-realtime`，百炼
`qwen3.5-omni-flash-realtime` 仅作为可回退错误的后备。七牛控制面 Session 的创建、
短期媒体凭证使用和 Stop 均由 Realtime Provider/Component 承担；短期凭证不得返回客户端
或持久化。
LLM 默认路由为七牛 MaaS `qwen/qwen3.5-plus`，可重试错误时回退到百炼
`qwen3.5-plus`；七牛 MaaS DeepSeek 与 DeepSeek 官方直连 Provider 仅保留为显式回滚能力。
七牛 MaaS API Key 不得返回客户端、持久化或写入日志。

## 3. 当前实现矩阵

> "场景准备"与"会话"列是直接实现类；`SceneService`/`SessionService` 基类已删除。"Flow/Evaluation"是保留的具体公共父类。

| 场景 | 场景准备 | Flow | 会话 | Evaluation |
|---|---|---|---|---|
| FreeChat | `FreeChatSceneService` | 无 | `FreeChatSessionService` | 无 |
| Custom | `CustomSceneService` | `CustomSceneFlowService` | `CustomSessionService` | `CustomEvaluationService` |
| IELTS | `IeltsSceneService` | `IeltsSceneFlowService` | `IeltsSessionService` | `IeltsEvaluationService` |

`scene`、`session`、`evaluation` 目录不使用配套 `impl` 子目录。以下通用类不允许存在：

```text
SceneServiceImpl / SceneService 接口
SceneFlowServiceImpl / SceneFlowService 接口
SessionServiceImpl / SessionService 接口
EvaluationServiceImpl / EvaluationService 接口
```

这些通用实现会把场景职责重新耦合到一起，与当前架构冲突。

## 4. 依赖方向与职责边界

```text
Controller / WebSocket
        │
        ▼
场景 Service 具体类
        │
        ├── Component / Domain
        ├── Provider
        └── Repository
                 ▲
                 │
Infrastructure（外部厂商、数据库、存储和框架实现）
```

规则：

- Controller 不得依赖 Mapper、Repository 或厂商实现。
- Service 不得依赖 Controller、数据库 Entity 或 Mapper。
- Component 不拥有跨重启的业务真相。
- Repository 是 Service 访问数据库的唯一入口。
- Infrastructure 实现技术细节，不拥有业务流程。
- 不同场景实现不得相互调用来复用业务；应下沉真正通用的纯组件或 Provider。
- 遇到循环依赖应重新划分职责，不能用 `@Lazy` 掩盖。

## 5. 目录规范

源码根目录：

```text
src/main/java/com/unispeaking
├── controller
├── websocket
├── service
│   ├── scene
│   │   ├── SceneFlowService.java

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [1024XEngineer/UniSpeaking](https://github.com/1024XEngineer/UniSpeaking) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-15 -->
