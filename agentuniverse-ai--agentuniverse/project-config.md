---
trigger: always_on
description: An agent is capable of autonomously acting to achieve goals set by humans, having capabilities for learning, reasoning, decision-making, and execution. It accomplishes objectives through steps such as task decomposition, using tools and knowledge, and progress control, and then independently completes its tasks. Within agentUniverse, an agent is one of the most critical domain components. It integrates a series of other domain components, including tools, knowledge, and plans, into a cohesive an
---

# Agent
An agent is capable of autonomously acting to achieve goals set by humans, having capabilities for learning, reasoning, decision-making, and execution. It accomplishes objectives through steps such as task decomposition, using tools and knowledge, and progress control, and then independently completes its tasks. Within agentUniverse, an agent is one of the most critical domain components. It integrates a series of other domain components, including tools, knowledge, and plans, into a cohesive and efficient system, ultimately completing the tasks assigned to it by people.
The performance of an agent directly impacts the effectiveness of application service. In complex service scenarios, often one or multiple competent agents are required to complete tasks. In agentUniverse, utilizing the achievements of industry and academia as well as practical experience in industry implementation, the following definition for agents is established, as shown in the figure.
![](../../../../_picture/agent.jpg)

Let's introduce the roles of the various components within the Agent component separately.

## Profile
This section is the global settings of the Agent, including the Agent's Target, Introduction, and LLM parts.

### Introduction
For the description of the Agent's role, such as being an AI assistant skilled in information analysis.

### Target
The goal that the Agent needs to achieve. The Agent will focus on this goal to complete a series of subsequent tasks.
  
```text
When setting the goal for an agent, it needs to be concise and impactful, describing a directional duty goal. Providing a specific and specialized domain scope is optimal.

✅ Good examples: answering questions in the finance domain, rewriting for code performance optimization.

❌ Bad examples: aimless descriptions such as "just chatting."
```
  
### Instruction
It includes specific settings for the Agent, such as personality traits, background knowledge, and behavioral patterns.

```text
How to write a good Instruction? We can adopt the following paradigm:
A clear scope of responsibilities the agent excels in (recommended) + Personality setting (fill in as needed) + 
Background knowledge (fill in as needed) + 
User input format (fill in as needed) + 
Behavioral guidance (recommended) + 
Agent output format (fill in as needed) + 
Practical examples - fewshot (recommended)

Following the above paradigm, here is an example definition:
✅ For instance, a good example would be:
You are Jerry's personal exclusive chatbot [The role played by the agent].
You excel at answering various professional questions encountered in life and casual conversations, particularly proficient at answering financial questions [A clear scope of responsibilities the agent excels in].
Your personality is cheerful and lively [Personality setting].
Jerry is especially interested in financial news on weekdays and also pays attention to macro-level policies [Background knowledge].
Jerry will communicate with you in the form of natural language for questions and answers [User input format]. 
Please adhere to the following principles in your responses: for professional-type questions, give priority to using knowledge retrieval tools for summarization and answering. Take professional questions seriously, and if you cannot arrive at a confident conclusion, please honestly communicate that to Jerry [Behavioral guidance].
Please output your response in the form of natural language [Agent output format].
Here is an actual example of communication with Jerry: Jerry asks how to view the macroeconomic data of December 2023? Response: The CPI has turned from a decrease to an increase month-on-month, with the year-on-year decline narrowing in December. Affected by factors such as cold weather and increased demand for consumption before the holiday, the CPI has turned from a decrease to an increase month-on-month, with the year-on-year decline narrowing; excluding food and energy prices, the core CPI rose by 0.6% year-on-year, with the increase remaining stable. The PPI declined month-on-month, with the year-on-year decline narrowing in December, affected by continuing decreases in international oil prices and insufficient demand for some industrial products, the national PPI declined month-on-month, with the year-on-year decline narrowing. [Practical example - fewshot]
(The text within [] symbols is for explanation, and actual submissions do not require prompts).

❌ For instance, a bad example would be:
You are skilled at providing various fitness plans tailored to different groups of people (the information provided is too scant, resulting in high uncertainty in outcomes, leading to generic advice that often does not meet the actual demands).
```

### LLM

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agentuniverse-ai/agentUniverse](https://github.com/agentuniverse-ai/agentUniverse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
