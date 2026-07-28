---
trigger: always_on
description: Agent配置文件(`agent.yml`)是COTA智能体的核心配置文件，定义了智能体的身份、对话策略、知识管理和动作能力。它是智能体的"大脑"，决定了智能体如何思考、决策和响应用户。
---

# Agent配置详解

Agent配置文件(`agent.yml`)是COTA智能体的核心配置文件，定义了智能体的身份、对话策略、知识管理和动作能力。它是智能体的"大脑"，决定了智能体如何思考、决策和响应用户。

## 🎯 Agent配置的作用与角色

**核心作用**：
- **身份定义**：通过`system`配置智能体的角色和性格
- **对话控制**：通过`dialogue`控制对话流程和参数
- **策略管理**：通过`policies`定义智能体的思维和决策模式
- **知识整合**：通过`knowledge`配置知识来源和检索策略
- **能力扩展**：通过`actions`定义智能体可执行的具体动作

**在COTA架构中的角色**：
- 被`Agent`类加载并解析为智能体实例
- 与`endpoints.yml`协同工作，前者定义能力，后者提供服务连接
- 通过`Processor`类驱动整体对话流程

## 📋 配置项总览

```yaml
system:      # 智能体身份 - 定义角色和基本信息
dialogue:    # 对话控制 - 控制对话模式和参数限制  
policies:    # 决策策略 - 定义思维模式和决策逻辑
knowledge:   # 知识管理 - 配置知识来源和检索策略
actions:     # 动作定义 - 定义智能体的具体能力和行为
```

| 配置项 | 对应模块 | 核心作用 |
|--------|----------|----------|
| `system` | `Agent`类初始化 | 设定智能体身份和描述 |
| `dialogue` | `Processor`对话处理 | 控制对话流程和限制 |
| `policies` | `DPL`策略学习 | 驱动智能体思考和决策 |
| `knowledge` | `Knowledge`知识管理 | 提供外部知识支持 |
| `actions` | `Action`系统 | 定义智能体的具体行为 |

## 🔧 详细配置说明

### 1. System配置 - 智能体身份

**作用**：定义智能体的基本身份信息，影响所有后续交互的语调和行为风格。

```yaml
system:
  description: "你是一个智能助手，你需要认真负责的回答帮用户解决问题"
  name: "assistant"  # 可选
```

**配置参数**：

| 参数 | 必需 | 默认值 | 说明 |
|------|------|--------|------|
| `description` | ✅ | - | 智能体角色描述，作为系统提示词的基础 |
| `name` | ❌ | "agent" | 智能体名称，用于日志和多Agent场景 |

### 2. Dialogue配置 - 对话控制

**作用**：控制对话的基本参数和流程限制，对应`Processor`类的对话处理逻辑。

```yaml
dialogue:
  use_proxy_user: false   # 是否启用代理用户模式
  max_proxy_step: 20      # 代理模式下的最大步骤数
  max_tokens: 500         # LLM生成最大令牌数
```

**配置参数**：

| 参数 | 必需 | 默认值 | 说明 |
|------|------|--------|------|
| `use_proxy_user` | ❌ | false | 是否启用代理用户功能，用于自动化模拟用户交互 |
| `max_proxy_step` | ❌ | 20 | 代理模式下的最大对话步数，防止无限循环 |
| `max_tokens` | ❌ | 500 | LLM生成的最大令牌数，控制回复长度 |

### 3. Policies配置 - 决策策略

**作用**：配置智能体的思维模式和决策策略，对应COTA的`DPL`(Dialogue Policy Learning)系统。

```yaml
policies:
  - type: trigger    # 触发式策略，基于规则快速响应
  - type: match      # 匹配策略，基于标注数据进行思维链学习
  - type: llm        # LLM策略，基于大模型推理 (三种策略中的一个或过个)
    config:
      llms:                   # LLM配置列表
        - name: rag-glm-4    # 默认LLM
        - name: rag-utter    # BotUtter动作专用LLM
          action: BotUtter
        - name: rag-selector  # Selector动作专用LLM
          action: Selector
```

**策略类型说明**：

| 策略类型 | 对应类 | 作用机制 |
|----------|--------|----------|
| `trigger` | `TriggerDPL` | 基于`policy/rules.yml`中定义的触发规则进行快速响应 |
| `match` | `MatchDPL` | 基于`policy/data.yml`中的标注数据学习思维链推理过程 |
| `llm` | `LLMDPL` | 基于大模型推理，支持动作级别的LLM绑定配置 |

**配置说明**：
- **trigger策略**：适用于简单、确定性的对话场景，响应速度快
- **match策略**：适用于复杂推理场景，通过学习标注数据生成思维链
- **llm策略**：基于大模型推理，支持为不同动作配置专用LLM
- **策略组合**：可以同时配置多种策略，系统会根据场景选择合适的策略

### 4. Knowledge配置 - 知识管理（可选）

**作用**：配置智能体的知识来源和检索策略，对应`Knowledge`和`KnowledgeFactory`类。主要用于RAG（检索增强生成）功能。

```yaml
knowledge:
  - type: llm                 # LLM类型知识源
    config:
      llms: 
        - name: rag-glm-4    # 知识检索使用的LLM
          action: BotUtter   # 绑定到BotUtter动作
        - name: rag-glm-4
          action: Selector   # 绑定到Selector动作
        - name: rag-glm-4    # 默认知识LLM
```

**知识配置参数**：

| 参数 | 说明 |
|------|------|
| `type` | 知识源类型，当前支持"llm" |
| `config.llms` | 知识检索使用的LLM配置列表 |

### 5. Actions配置 - 动作定义

**作用**：定义智能体可执行的具体动作，每个动作对应一个`Action`类或其子类。

#### 5.1 基础动作

**UserUtter - 用户输入处理**
```yaml
UserUtter:
  description: "用户的action - 用户向智能体提问"
  prompt: |
    历史对话:
    {{history_messages}}
    请输出对医生说的话
    
  breaker:  # 对话中断判断器
    description: "判断是否跳出"
    prompt: |
      根据对话内容，判断对话是否满足要求
      对话内容: {{history_messages}}
      如果对话完整且可以结束, 输出标识符true。
      如果对话还需要继续, 输出标识符false。
      输出格式为: <标识符>
```

**BotUtter - 智能体回复**
```yaml
BotUtter:
  description: "回复用户"
  prompt: |
    你是一个智能助手，需要根据当前对话历史生成合适的回复。
    
    **任务描述：** {{task_description}}
    **输出格式要求：**
    你必须严格按照以下JSON格式响应，不要有任何其他内容：
    ```json
    {"thought": "<你的推理过程>", "text": "<你的回复内容>"}
    ```
    
    **学习参考资料：** {{policies}}
    **实际对话历史：** {{history_actions}}
    
    请分析上述实际对话历史，参考学习资料中的思维模式，生成合适的JSON格式回复：
```

**Selector - 动作选择器**
```yaml
Selector:
  description: "选择合适的Actions"
  prompt: |
    你是一个智能对话助手，需要根据当前对话状态选择下一个最合适的Action。
    
    **输出格式要求：**
    ```json
    {"thought": "<你的推理过程>", "action": "<工具名称>"}
    ```
    
    **可用的Action工具：** {{action_descriptions}}
    **决策参考模式：** {{policies}}
    **当前对话状态：** {{history_actions}}
    
    请分析当前对话状态，参考决策模式，选择最合适的下一个Action并输出JSON格式结果：
```

#### 5.2 Form动作 - 表单处理

**作用**：Form动作用于收集用户信息并执行外部API调用，对应`Form`类。

**核心组件**：
- **prompt**: 统一处理槽位更新和结果返回的主要提示词
- **slots**: 需要收集的信息槽位定义
- **executer**: 外部服务执行器

**配置示例**：
```yaml
Weather:  # 天气查询Form动作
  description: "查询天气"
  prompt: |
    当前正在执行{{current_form_name}}，其描述为{{current_form_description}}。
    根据对话内容及Action序列，结合当前slot的状态，填充或重置slot的值。
    
    历史Action序列为:
    {{history_actions_with_thoughts}}
    
    Action的描述为:
    {{action_descriptions}}
    
    当前slots为:
    {{current_form_slot_states}}
    
    slots的含义为:
    {{current_form_slot_descriptions}}
    
    填充或重置slot的值，保持slots格式输出json字符串。

  slots:  # 槽位定义
    city:
      description: "城市，注意：接口只支持输入单个城市"
      prompt: |
        当前正在执行Action {{current_form_name}}, 其描述为 {{current_form_description}}。
        接下来需要询问用户，需要查询哪个城市的天气。
        
        你必须严格按照以下JSON格式响应：
        {"text": "<你的回复内容>"}

    time:
      description: "时间"
      prompt: |
        当前正在执行Action {{current_form_name}}, 其描述为 {{current_form_description}}。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CotaAI/cota](https://github.com/CotaAI/cota) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
