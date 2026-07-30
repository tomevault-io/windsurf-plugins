---
trigger: always_on
description: - 1990-2010s schmidhuber work https://people.idsia.ch/~juergen/lecun-rehash-1990-2022.html https://twitter.com/SchmidhuberAI/status/1683870175299239937
---

## timeline

- 1990-2010s schmidhuber work https://people.idsia.ch/~juergen/lecun-rehash-1990-2022.html https://twitter.com/SchmidhuberAI/status/1683870175299239937
- adept fundraising
- webgpt
- nov 2022 - [harrison included agents](https://twitter.com/hwchase17/status/1595456660507459585)
- tool use - react, toolformer, chatgpt plugins

## Survey articles

- https://www.sequoiacap.com/article/autonomous-agents-perspective/

## React Model

- https://react-lm.github.io and [google blog](https://ai.googleblog.com/2022/11/react-synergizing-reasoning-and-acting.html)
	- https://interconnected.org/home/2023/03/16/singularity
	- https://blog.langchain.dev/agent-toolkits/
	- https://www.geoffreylitt.com/2023/01/29/fun-with-compositional-llms-querying-basketball-stats-with-gpt-3-statmuse-langchain.html
- Automatic Reasoning & Tool-Use of LLMs
	- Retrieves examples of reasoning & tool use from task library -Writes its own program, pauses when tool call is encountered, integrates output -Much better than few-shot prompting [tweet](https://mobile.twitter.com/johnjnay/status/1636555772082499584?s=12&t=90xQ8sGy63D2OtiaoGJuww)
- alternative
	- Karpas, E., Abend, O., Belinkov, Y., Lenz, B., Lieber, O., Ratner, N., . . ., Tenenholtz, M., MRKL Systems: A modular, neuro-symbolic architecture that combines large language models, external knowledge sources and discrete reasoning, 2022.

## other agent paradigms

https://www.geeksforgeeks.org/agents-artificial-intelligence/#

Reflexion: an autonomous agent with dynamic memory and self-reflection https://arxiv.org/abs/2303.11366 https://twitter.com/johnjnay/status/1638696539257184258
A Self-Reflecting LLM Agent

Equips LLM-based agent w/ 
-dynamic memory
-a self-reflective LLM
-a method for detecting hallucinations
self critique works well to fix first shot problems 
- https://twitter.com/ericjang11/status/1639882111338573824

browser agents
- hyperwrite personal assistant https://venturebeat.com/ai/hyperwrite-unveils-breakthrough-ai-agent-that-can-surf-the-web-like-a-human/
- [Mind2Web](https://osu-nlp-group.github.io/Mind2Web/) is a dataset for developing and evaluating generalist agents for the web that can follow language instructions to complete complex tasks on any website. Mind2Web contains 2,350 tasks from 137 websites spanning 31 domains that:
	- Reflect diverse and practical use cases on the web.
	- Provide challenging yet realistic environments with real-world websites.
	- Test generalization ability across tasks and environments.




https://twitter.com/jh_damm/status/1646233661832929280?s=20
  
React Style Prompting 
BabyAGI: task planning & prioritization for endless iteration 
Agent Duos: collaborative problem-solving 
Stacking Agents: task delegation & collaboration


### Roleplaying / Multi-Agent systems
- camel and autogen
	- https://arxiv.org/pdf/2308.08155v1.pdf 
	- https://github.com/camel-ai/camel
- metagpt
- Multi-Agent Systems: • 
	- BabyAGI: BabyAGI [6] is an example implementation of an AI-powered task management system in a Python script (according to its own documentation). In this implemented system, multiple LLM-based agents are used. For example, there is an agent for creating new tasks based on the objective and the result of the previous task, an agent for prioritizing the task list, and an agent for completing tasks/sub-tasks. BabyAGI is a multi-agent system with a static agent communication pattern, i.e., a predefined order of agent communication. • 
	- CAMEL: CAMEL [28] is a communicative agent framework. It demonstrates how role-playing can be used to let chat agents communicate with each other for task completion. It also records agent conversations for behavior analysis and capability understanding. CAMEL does not support tool-using, such as code execution. Inception prompting technique is used to achieve autonomous cooperation between agents. • 
	- MetaGPT: MetaGPT [20] is a multi-agent framework for assigning different roles to GPTs to form a collaborative software entity for complex tasks. It is a specialized LLM-based multi-agent framework for collaborative software development. • 
	- Multi-Agent Debate: Two recent works investigate and show that multi-agent debate is an effective way to encourage divergent thinking in LLMs [29] and to improve the factuality and reasoning of LLMs [16]. In both works, multiple LLM inference instances are constructed as multiple agents to solve problems with agent debate. Each agent is simply an LLM inference instance, while no tool or human is involved. The conversation also needs to follow a pre-defined order.

## major agents projects

- AutoGPT
	- https://twitter.com/karpathy/status/1642598890573819905
		- Next frontier of prompt engineering imo: "AutoGPTs" . 1 GPT call is just like 1 instruction on a computer. They can be strung together into programs. Use prompt to define I/O device and tool specs, define the cognitive loop, page data in and out of context window, .run().

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [swyxio/ai-notes](https://github.com/swyxio/ai-notes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
