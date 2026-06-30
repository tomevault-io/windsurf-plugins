---
trigger: always_on
description: <!--Copyright 2024 The HuggingFace Team. All rights reserved.
---

<!--Copyright 2024 The HuggingFace Team. All rights reserved.

Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except in compliance with
the License. You may obtain a copy of the License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on
an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the
specific language governing permissions and limitations under the License.

⚠️ Note that this file is in Markdown but contain specific syntax for our doc-builder (similar to MDX) that may not be
rendered properly in your Markdown viewer.

-->
# Agents and tools

[[open-in-colab]]

### What is an agent?

Large Language Models (LLMs) trained to perform [causal language modeling](./tasks/language_modeling.) can tackle a wide range of tasks, but they often struggle with basic tasks like logic, calculation, and search. When prompted in domains in which they do not perform well, they often fail to generate the answer we expect them to.

One approach to overcome this weakness is to create an *agent*.

An agent is a system that uses an LLM as its engine, and it has access to functions called *tools*.

These *tools* are functions for performing a task, and they contain all necessary description for the agent to properly use them.

The agent can be programmed to:
- devise a series of actions/tools and run them all at once,  like the [`CodeAgent`]
- plan and execute actions/tools one by one and wait for the outcome of each action before launching the next one, like the [`ReactJsonAgent`]

### Types of agents

#### Code agent

This agent has a planning step, then generates python code to execute all its actions at once. It natively handles different input and output types for its tools, thus it is the recommended choice for multimodal tasks.

#### React agents

This is the go-to agent to solve reasoning tasks, since the ReAct framework ([Yao et al., 2022](https://huggingface.co/papers/2210.03629)) makes it really efficient to think on the basis of its previous observations.

We implement two versions of ReactJsonAgent: 
- [`ReactJsonAgent`] generates tool calls as a JSON in its output.
- [`ReactCodeAgent`] is a new type of ReactJsonAgent that generates its tool calls as blobs of code, which works really well for LLMs that have strong coding performance.

> [!TIP]
> Read [Open-source LLMs as LangChain Agents](https://huggingface.co/blog/open-source-llms-as-agents) blog post to learn more about ReAct agents.

<div class="flex justify-center">
    <img
        class="block dark:hidden"
        src="https://huggingface.co/datasets/huggingface/documentation-images/resolve/main/transformers/Agent_ManimCE.gif"
    />
    <img
        class="hidden dark:block"
        src="https://huggingface.co/datasets/huggingface/documentation-images/resolve/main/transformers/Agent_ManimCE.gif"
    />
</div>

![Framework of a React Agent](https://huggingface.co/datasets/huggingface/documentation-images/resolve/main/blog/open-source-llms-as-agents/ReAct.png)

For example, here is how a ReAct Code agent would work its way through the following question.

```py3
>>> agent.run(
...     "How many more blocks (also denoted as layers) in BERT base encoder than the encoder from the architecture proposed in Attention is All You Need?",
... )
=====New task=====
How many more blocks (also denoted as layers) in BERT base encoder than the encoder from the architecture proposed in Attention is All You Need?
====Agent is executing the code below:
bert_blocks = search(query="number of blocks in BERT base encoder")
print("BERT blocks:", bert_blocks)
====
Print outputs:
BERT blocks: twelve encoder blocks

====Agent is executing the code below:
attention_layer = search(query="number of layers in Attention is All You Need")
print("Attention layers:", attention_layer)
====
Print outputs:
Attention layers: Encoder: The encoder is composed of a stack of N = 6 identical layers. Each layer has two sub-layers. The first is a multi-head self-attention mechanism, and the second is a simple, position- 2 Page 3 Figure 1: The Transformer - model architecture.

====Agent is executing the code below:
bert_blocks = 12
attention_layers = 6
diff = bert_blocks - attention_layers
print("Difference in blocks:", diff)
final_answer(diff)
====

Print outputs:
Difference in blocks: 6

Final answer: 6
```

### How can I build an agent?

To initialize an agent, you need these arguments:

- an LLM to power your agent - the agent is not exactly the LLM, it’s more like the agent is a program that uses an LLM as its engine.
- a system prompt: what the LLM engine will be prompted with to generate its output
- a toolbox from which the agent pick tools to execute
- a parser to extract from the LLM output which tools are to call and with which arguments

Upon initialization of the agent system, the tool attributes are used to generate a tool description, then baked into the agent’s `system_prompt` to let it know which tools it can use and why.

To start with, please install the `agents` extras in order to install all default dependencies.

```bash
pip install transformers[agents]
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xuyang-liu16/V2Drop](https://github.com/xuyang-liu16/V2Drop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
