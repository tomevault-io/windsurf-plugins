---
trigger: always_on
description: 学习如何使用ReactAgent将语言模型与工具结合，创建能够推理任务、决定工具使用并迭代解决问题的智能系统
---


# Agents

Agents 将大语言模型与工具结合，创建具备任务推理、工具使用决策、工具调用的自动化系统，系统具备持续推理、工具调用的循环迭代能力，直至问题解决。

Spring AI Alibaba 提供了基于 `ReactAgent` 的生产级 Agent 实现。

**一个 LLM Agent 在循环中通过运行工具来实现目标**。Agent 会一直运行直到满足停止条件 —— 即当模型输出最终答案或达到迭代限制时。

<!-- ![react-agent-architecture](../imgs/reactagent.png) -->

## ReactAgent 理论基础

### 什么是 ReAct

ReAct（Reasoning + Acting）是一种将推理和行动相结合的 Agent 范式。在这个范式中，Agent 会：

1. **思考（Reasoning）**：分析当前情况，决定下一步该做什么
2. **行动（Acting）**：执行工具调用或生成最终答案
3. **观察（Observation）**：接收工具执行的结果
4. **迭代**：基于观察结果继续思考和行动，直到完成任务

这个循环使 Agent 能够：
- 将复杂问题分解为多个步骤
- 动态调整策略基于中间结果
- 处理需要多次工具调用的任务
- 在不确定的环境中做出决策

### ReactAgent 的工作原理

Spring AI Alibaba 中的`ReactAgent` 基于 **Graph 运行时**构建。Graph 由节点（steps）和边（connections）组成，定义了 Agent 如何处理信息。Agent 在这个 Graph 中移动，执行如下节点：

- **Model Node (模型节点)**：调用 LLM 进行推理和决策
- **Tool Node (工具节点)**：执行工具调用
- **Hook Nodes (钩子节点)**：在关键位置插入自定义逻辑

ReactAgent 的核心执行流程：

<img src="/img/agent/agents/reactagent.png" alt="reactagent" width="360" />

## 核心组件

### Model（模型）

Model 是 Agent 的推理引擎。Spring AI Alibaba 支持多种配置方式。

#### 基础模型配置

最直接的方式是使用 `ChatModel` 实例：

<Code
  language="java"
  title="ReactAgent 基础配置示例" sourceUrl="https://github.com/alibaba/spring-ai-alibaba/tree/main/examples/documentation/src/main/java/com/alibaba/cloud/ai/examples/documentation/framework/tutorials/AgentsExample.java"
>
{`import com.alibaba.cloud.ai.dashscope.api.DashScopeApi;
import com.alibaba.cloud.ai.dashscope.chat.DashScopeChatModel;
import com.alibaba.cloud.ai.graph.agent.ReactAgent;

// 创建 DashScope API 实例
DashScopeApi dashScopeApi = DashScopeApi.builder()
    .apiKey(System.getenv("AI_DASHSCOPE_API_KEY"))
    .build();

// 创建 ChatModel
ChatModel chatModel = DashScopeChatModel.builder()
    .dashScopeApi(dashScopeApi)
    .build();

// 创建 Agent
ReactAgent agent = ReactAgent.builder()
    .name("my_agent")
    .model(chatModel)
    .build();`}
</Code>

#### 高级模型配置

通过 `ChatOptions` 可以精细控制模型行为：

<Code
  language="java"
  title="DashScopeChatModel 高级配置" sourceUrl="https://github.com/alibaba/spring-ai-alibaba/tree/main/examples/documentation/src/main/java/com/alibaba/cloud/ai/examples/documentation/framework/tutorials/AgentsExample.java"
>
{`import com.alibaba.cloud.ai.dashscope.chat.DashScopeChatOptions;

ChatModel chatModel = DashScopeChatModel.builder()
    .dashScopeApi(dashScopeApi)
    .defaultOptions(DashScopeChatOptions.builder()
		.withModel(DashScopeChatModel.DEFAULT_MODEL_NAME)
        .withTemperature(0.7)    // 控制随机性
        .withMaxToken(2000)      // 最大输出长度
        .withTopP(0.9)           // 核采样参数
        .build())
    .build();`}
</Code>

**常用参数说明**：
- `temperature`：控制输出的随机性（0.0-1.0），值越高越有创造性
- `maxTokens`：限制单次响应的最大 token 数
- `topP`：核采样，控制输出的多样性
- 更多参数请参考 ChatModel 适配

### Tools（工具）

工具赋予 Agent 执行操作的能力，支持顺序执行、并行调用、动态选择和错误处理。

#### 定义和使用工具

<Code
  language="java"
  title="SearchTool 自定义工具示例" sourceUrl="https://github.com/alibaba/spring-ai-alibaba/tree/main/examples/documentation/src/main/java/com/alibaba/cloud/ai/examples/documentation/framework/tutorials/AgentsExample.java"
>
{`import org.springframework.ai.tool.ToolCallback;
import org.springframework.ai.tool.function.FunctionToolCallback;
import org.springframework.ai.tool.annotation.ToolParam;
import org.springframework.ai.chat.model.ToolContext;
import java.util.function.BiFunction;

// 定义工具（示例：仅一个搜索工具）
public class SearchTool implements BiFunction<String, ToolContext, String> {
    @Override
    public String apply(String query, ToolContext context) {
        // 实现搜索逻辑
        return "搜索结果: " + query;
    }
}

// 创建工具回调
ToolCallback searchTool = FunctionToolCallback.builder("search", new SearchTool())
    .description("搜索工具")
    .build();

// 在Agent中使用
ReactAgent agent = ReactAgent.builder()
    .name("search_agent")
    .model(chatModel)
    .tools(searchTool)
    .build();`}
</Code>

#### 工具错误处理

<Code
  language="java"
  title="ToolErrorInterceptor 工具错误处理" sourceUrl="https://github.com/alibaba/spring-ai-alibaba/tree/main/examples/documentation/src/main/java/com/alibaba/cloud/ai/examples/documentation/framework/tutorials/AgentsExample.java"
>
{`import com.alibaba.cloud.ai.graph.agent.interceptor.ToolInterceptor;
import com.alibaba.cloud.ai.graph.agent.interceptor.ToolCallRequest;
import com.alibaba.cloud.ai.graph.agent.interceptor.ToolCallResponse;
import com.alibaba.cloud.ai.graph.agent.interceptor.ToolCallHandler;

public class ToolErrorInterceptor extends ToolInterceptor {
    @Override
    public ToolCallResponse interceptToolCall(ToolCallRequest request, ToolCallHandler handler) {
        try {
            return handler.call(request);
        } catch (Exception e) {
            return ToolCallResponse.of(request.getToolCallId(), request.getToolName(),
                "Tool failed: " + e.getMessage());
        }
    }

    @Override
    public String getName() {
        return "ToolErrorInterceptor";
    }
}

ReactAgent agent = ReactAgent.builder()
    .name("my_agent")
    .model(chatModel)
    .interceptors(new ToolErrorInterceptor())
    .build();`}
</Code>

**ReAct 循环示例**：Agent 自动交替进行推理和工具调用，直到获得最终答案。

```
用户: 查询杭州天气并推荐活动
→ [推理] 需要查天气 → [行动] get_weather("杭州") → [观察] 晴，25°C
→ [推理] 需要推荐活动 → [行动] search("户外活动") → [观察] 西湖游玩...
→ [推理] 信息充足 → [行动] 生成答案
```

### System Prompt（系统提示）

System Prompt 塑造 Agent 处理任务的方式。

#### 基础用法

通过 `systemPrompt` 参数提供字符串：

<Code

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [spring-ai-alibaba/website](https://github.com/spring-ai-alibaba/website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
