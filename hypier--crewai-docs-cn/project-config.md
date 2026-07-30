---
trigger: always_on
description: 什么是 crewAI Agent 以及如何使用它们。
---


## 什么是代理？
!!! note "什么是代理？"
    代理是一个**自主单元**，其编程目的是：
    <ul>
      <li class='leading-3'>执行任务</li>
      <li class='leading-3'>做出决策</li>
      <li class='leading-3'>与其他代理沟通</li>
    </ul>
      <br/>
    可以将代理视为团队中的一员，拥有特定的技能和特定的工作。代理可以担任不同的角色，如“研究员”、“撰稿人”或“客户支持”，每个角色都为团队的整体目标做出贡献。

## 代理属性

| 属性                       | 参数      | 描述                                                                                                                                                                                                                                          |
| :------------------------ | :---- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **角色**                   | `role`  | 定义代理在团队中的功能。它决定了代理最适合执行的任务类型。                                                                                                                                                                                    |
| **目标**                   | `goal`  | 代理旨在实现的个人目标。它指导代理的决策过程。                                                                                                                                                                                                  |
| **背景故事**               | `backstory`  | 提供代理角色和目标的背景，使互动和协作动态更加丰富。                                                                                                                                                                                        |
| **语言模型** *(可选)*      | `llm`  | 表示将运行代理的语言模型。它动态从 `OPENAI_MODEL_NAME` 环境变量中获取模型名称，如果未指定，则默认使用 "gpt-4"。                                                                                                                        |
| **工具** *(可选)*          | `tools`  | 代理可以用来执行任务的一组能力或功能。预计是与代理执行环境兼容的自定义类的实例。工具的默认值为一个空列表。                                                                                                                               |
| **函数调用语言模型** *(可选)* | `function_calling_llm`  | 指定将处理该代理工具调用的语言模型，覆盖传递的团队函数调用语言模型。默认值为 `None`。                                                                                                                                                      |
| **最大迭代次数** *(可选)*   | `max_iter` | 最大迭代次数是代理在被迫给出最佳答案之前可以执行的最大迭代次数。默认值为 `25`。                                                                                                                                                             |
| **最大请求频率** *(可选)*   | `max_rpm`  | 最大请求频率是代理为避免速率限制而每分钟可以执行的最大请求次数。它是可选的，可以不指定，默认值为 `None`。                                                                                                                                  |
| **最大执行时间** *(可选)*   | `max_execution_time`  | 最大执行时间是代理执行任务的最大执行时间。它是可选的，可以不指定，默认值为 `None`，意味着没有最大执行时间。                                                                                                                             |
| **详细信息** *(可选)*       | `verbose`  | 将其设置为 `True` 会配置内部记录器提供详细的执行日志，有助于调试和监控。默认值为 `False`。                                                                                                                                              |
| **允许委托** *(可选)*      | `allow_delegation`  | 代理可以将任务或问题委托给彼此，确保每个任务由最合适的代理处理。默认值为 `True`。                                                                                                                                                       |
| **步骤回调** *(可选)*      | `step_callback`  | 在代理的每一步之后调用的函数。这可以用于记录代理的操作或执行其他操作。它将覆盖团队的 `step_callback`。                                                                                                                               |
| **缓存** *(可选)*          | `cache`  | 指示代理是否应使用工具使用的缓存。默认值为 `True`。                                                                                                                                                                                      |
| **系统模板** *(可选)*      | `system_template`  | 指定代理的系统格式。默认值为 `None`。                                                                                                                                                                                                      |
| **提示模板** *(可选)*      | `prompt_template`  | 指定代理的提示格式。默认值为 `None`。                                                                                                                                                                                                      |
| **响应模板** *(可选)*      | `response_template`  | 指定代理的响应格式。默认值为 `None`。                                                                                                                                                                                                      |

## 创建代理

!!! note "代理交互"
    代理可以使用crewAI内置的委托和通信机制相互交互。这允许在团队内进行动态任务管理和问题解决。

要创建一个代理，通常需要初始化一个`Agent`类的实例，并设置所需的属性。以下是一个包含所有属性的概念示例：

```python
# Example: Creating an agent with all attributes
from crewai import Agent

agent = Agent(
  role='Data Analyst',
  goal='Extract actionable insights',
  backstory="""You're a data analyst at a large company.
  You're responsible for analyzing data and providing insights
  to the business.
  You're currently working on a project to analyze the
  performance of our marketing campaigns.""",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hypier/crewAI_docs_cn](https://github.com/hypier/crewAI_docs_cn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
