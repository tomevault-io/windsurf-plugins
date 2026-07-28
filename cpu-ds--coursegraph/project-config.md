---
trigger: always_on
description: > 该框架受到了 OpenAI [swarm](https://github.com/openai/swarm) 项目的启发
---

# Agent编排框架

<ArticleMetadata/>

> 该框架受到了 OpenAI [swarm](https://github.com/openai/swarm) 项目的启发

`course_graph.agent` 是一款通用的多智能体编排框架, 不仅支持多智能体的 **主动切换**, 也可以使用 **人工编排**, 从而形成工作流。`course_graph.agent` 也拥有比 swarm 更好的 function2json 的能力。基于 `course_graph.llm` , 该框架也支持调用更多类型的大模型。

接下来介绍该框架的一般用法, 如果想了解如何使用智能体抽取知识图谱以及与大模型抽取知识图谱的区别, 请参考 [使用Agent抽取知识图谱](../agent/kg)。

## 创建一个智能体

```python
from course_graph.llm import Qwen
from course_graph.agent import Agent

llm = Qwen()
translator = Agent(name='translator',
                   llm=llm,
                   instruction='你是一个翻译, 擅长将中文翻译成英文。')
```

`Agent` 类是对 `LLM` 类的包装, 为 `LLM` 的多轮对话、长期记忆和工具调用提供了更加友好的接口。

创建一个 `Agent` 类, 需要提供智能体的名称 `name`、基础大模型 `llm` 和 系统指令 `instruction`, 其中 `instruction` 将作为 `system message` 在每次对话时传递给大模型以提供高层指导。

当然你也可以将具体的指令需求直接写在 `instruction` 中, 这种方式将在 [工作流编排](#工作流编排) 中具体解释。

### 指令参数

在创建 `Agent` 对象时, `instruction` 不仅可以是一个字符串，也可以是一个函数，但这个函数 **必需** 返回一个字符串。该函数可以通过 `instruction_args` 传递参数, 这些参数将会在智能体初始化时传递给 `instruction` 函数。

```python
def get_instruction(name: str):
    return f'你的名字是{name}'

agent = Agent(llm=llm,
              instruction=get_instruction,
              instruction_args={'name': 'Mike'})
```

> [!TIP]
>  `instruction` 参数也可以是一个文本文件路径, 文件内容将作为指令。

## 创建一个控制器

```python
from course_graph.agent import Controller
controller = Controller()
```

`Controller` 负责启动 `Agent` 并为 `Agent` 提供上下文、具体执行外部工具等功能。

## 启动智能体

```python
resp = controller.run_sync(agent=translator, message="请帮我翻译蛋白质。")
```

其中 `message` 参数代表用户的具体指令或者是用户与智能体对话的开始。

`controller.run_sync` 方法返回一个 `ControllerResponse` 对象, 其中 `agent` 代表最后响应的 `Agent` 对象, `message` 代表智能体最后的相应内容。

## 使用外部工具

智能体可以使用外部工具, 在这里我们定义 **外部工具** 等价于一个或多个 **函数对象**。在解决相应任务时, 智能体会自动选择外部工具、提供相应的参数并生成响应。之后则由控制器自动注入参数值并执行相应的函数。

接下来的描述中可能混用函数与工具、外部工具等说法。

### 添加外部工具

你可以在创建 `Agent` 时指定提供的外部工具:

```python
def get_weather(location):
    ....

assistant = Agent(name="assistant",
                  llm=llm,
                  functions=[get_weather],
                  instruction="你是一个通用的助手。")
                  
```

也可以在后续继续添加:

```python
assistant.add_too_functions(get_weather)
```

当然，在函数定义时就可以声明其为一个工具函数:

```python
@assistant.tool()
def get_weather(location):
    ...
```

> [!IMPORTANT]
> 工具函数应该是一个已经被定义的函数对象且上述语句所在的作用域需要具有对其拥有访问权限。

> [!WARNING]
> 工具函数不能是 lambda 函数。

### 外部工具的描述

智能体很难通过函数名称推断出函数的作用和参数, 所以需要尽可能多的为函数添加描述信息。

#### 通过标注和文档 (推荐)

对于一个函数我们应该使用文档清晰的描述函数的形参、形参类型、返回值描述、返回值类型和函数的功能。当然也可以使用类型标注来描述形参类型和返回值类型。

以 `get_weather` 为例, 最清晰的函数定义应该是：

```python
def get_weather(location: str) -> str:
    """ 获取某个位置当前的天气状况。

    Args:
        location (str): 待查询的位置

    Returns:
        str: 该位置的天气状况
    """
    return f"{location} 当前的天气是多云, 温度22-27摄氏度。"  # 模拟数据
```

`Agent` 则会将函数转换为 json 格式的描述信息传递给大模型。

> [!NOTE]
> 文档支持 ReST、Google、Numpydoc-style 和 Epydoc 风格。

#### 通过 Tool 接口

如果工具函数来自外部库, 没有办法控制标注和文档时, 可以有以下两种解决方式:

- 对函数进行再次封装, 封装时标注类型并编写文档

- 使用 `Tool` 接口, 编写 json 格式的函数描述

`Tool` 类型是一个 TypedDict, 其中必需包含 `function` 和 `tool` 两个字段。其中 `function` 字段传递函数对象, `tool` 字段传递函数描述。

函数描述需要遵守 `openai.types.chat.ChatCompletionToolParam` 规范。可以点击 [这里](https://cookbook.openai.com/examples/how_to_call_functions_with_chat_models) 查看官方指南。

仍然以 `get_weather` 为例, 编写 `Tool` 接口并使用 `add_tools` 函数添加工具函数 (注意不是 `add_tool_functions` 函数):

```python
from course_graph.agent import Tool

get_weather_tool: Tool = {
    'function': get_weather,
    'tool': {
        'type': 'function',
        'function': {
            'name': 'get_weather',
            'description': '获取某个位置当前的天气况。',
            'parameters': {
                'type': 'object',
                'properties': {
                    "location": {
                        "type": "string",
                        "description": "待查询的位置",
                    },
                },
                'required': ['location']
            }
        }
    }
}

assistant.add_tools(get_weather_tool)
```
> [!TIP]
> 可能你注意到了, 在这里我们并没有提供函数的返回值描述和类型, 事实上大模型不需要这些信息, 但是我们仍然建议你在编写函数文档的时候添加上这些信息。

> [!TIP]
> 对于同一个工具函数, 不需要重调用 `add_tools` 和 `add_tool_functions` 进行添加。 

### 外部工具的返回值

工具函数可以返回任意值，但以下几种返回值类型需要被特殊关注到:

- 字符串: 字符串一般表示函数的执行结果, 例如天气查询的返回值, 此返回值会交还给智能体。

- `Agent` 对象: 表示要切换到新的智能体上继续执行任务。这种情况下工具函数一般被视为 **智能体转移函数**<a id='transfer-function'></a>。

- `ContextVariables` 对象: 表示要更新的上下文变量。详细见 [上下文变量](#上下文变量)。

- `Result` 对象: 以上三种类型的组合类。

除此之外的返回值都将会被忽略, 其隐藏含义是只关心函数的副作用而不关心函数的返回值。

另外, 我们通常使用历史对话传递消息, 在智能体切换的时候, 之前智能体的与用户的对话或是工具函数调用的结果都会被保存起来, 但是这种方式容易造成历史对话过长。如果你明确不需要这种传递机制, 可以在 `Result` 对象中设置 `message` 字段为 `False` 来实现转换智能体但并不携带历史对话。

```python
result = Result(agent=assistant, message=False)
```
> [!TIP]
> 在这种情况下你可以使用上下文变量或 `add_assistant_message` 方法实现信息的传递。


## 上下文变量

智能体拥有短期记忆和长期记忆, 在这里我们将短期记忆定义为对话的历史记录, 而使用上下文变量实现长期记忆。

上下文变量使用 `ContextVariables` 类, 其本质上就是一个字典类型。只不过为了在函数的返回值类型中区分字典类型和上下文变量类型, 就单独创造了一个类型。

### 初始化上下文变量

在创建控制器时初始化上下文变量:

```python
controller = Controller(context_variables={'current_time': '2024/09/01'})
```

### 使用上下文变量

智能体有两种方式可以使用到上下文变量。

#### instruction中使用

当 `instruction` 类型为函数时, 可以额外传递一个`ContextVariables` 类型的形参, 但 **必需** 标注这个形参的类型为 `ContextVariables` 类型。

``` python
from course_graph.agent import ContextVariables

def assistant_instruction(context_variables: ContextVariables):
    return f"你是一个通用的助手, 当前的时间是: {context_variables['current_time']}。"


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CPU-DS/CourseGraph](https://github.com/CPU-DS/CourseGraph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
