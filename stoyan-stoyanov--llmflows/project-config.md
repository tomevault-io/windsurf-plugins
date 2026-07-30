---
trigger: always_on
description: You can find the [complete example](https://github.com/stoyan-stoyanov/llmflows/tree/main/examples/react_agent) on Github.
---

## TL;DR

You can find the [complete example](https://github.com/stoyan-stoyanov/llmflows/tree/main/examples/react_agent) on Github.

## Guide
The concept of LLM-powered agents became very popular after the release of GPT-4. 
Agents are programs that include one or multiple LLMs that can invoke traditional 
functions (often called tools).

LLMs gain the ability to utilize tools by being forced to generate specific text 
through prompts that list the available tools and provide examples of how 
to use them.

For example:

```python
system_prompt = """
You are a 150IQ assistant that answers questions.
To answer the questions you have access to the following tools:
- search
- calculator

You can use tools in the following way:
search: [Who is Albert Einsten?]
calculator: [2 + 3]
"""
```

Every time the LLM generates text based on the rules above, we can parse the text to 
extract the function name and the function argument (e.g., "search" and "Who is Albert 
Einstein?) and invoke the respective function. Then we can evaluate the result of the 
function and even add it to the conversation history, so the LLM can use it the next 
time it generates text.

One popular agent architecture is the <a href="https://arxiv.org/abs/2210.03629" target="_blank">ReAct architecture</a>. 
In this paradigm, the agent runs in a loop of reasoning, acting, and observing the 
action outcome to solve a problem:

1. The agent starts by reasoning how it can solve the appointed task. 
2. Afterward, it specifies an action based on the reasoning. 
3. The agent invokes a function corresponding to the action and the result makes up the observation.
4. Once it has the observation, the agent starts the reasoning phase again, and the 
loop continues until the agent decides it has all the required information and generates a final answer.

Example:
```commandline
Question: What is the elevation range for the area that the eastern sector of the 
Colorado orogeny extends into?
---
Thought: I need to search Colorado orogeny, find the area that the eastern sector of 
the Colorado orogeny extends into, then find the elevation range of the area.
---
Action: wikipedia: Colorado orogeny
---
Observation: The Colorado orogeny was an episode of mountain building (an orogeny) in 
Colorado and surrounding areas.
---
Thought: It does not mention the eastern sector. So I need to look up eastern sector.
---
Action: wikipedia: eastern sector
---
Observation: (Result 1 / 1) The eastern sector extends into the High Plains and is 
called the Central Plains orogeny.
---
Thought: The eastern sector of Colorado orogeny extends into the High Plains. So I need 
to search High Plains and find its elevation range.
---
Action: wikipedia: High Plains
---
Observation: High Plains refers to one of two distinct land regions
---
Thought: I need to instead search High Plains (United States).
---
Action: wikipedia: High Plains (United States)
---
Observation: The High Plains are a subregion of the Great Plains. From east to west, 
the High Plains rise in elevation from around 1,800 to 7,000 ft (550 to 2,130 m).[3] 
Thought: High Plains rise in elevation from around 1,800 to 7,000 ft, so the answer 
is 1,800 to 7,000 ft.
---
Action: final answer: 1,800 to 7,000 ft,
```

Let's build a simple version of this agent. To keep things tidy, we will create three separate files:

- `tools.py`, where we will define the tool functions.
- `prompts.py` to store all the prompts
- `agent.py`, where we will implement the agent flow.


We will start by specifying the tools it will have access to. To keep it simple, let's begin by creating two tools in `tools.py` - a calculator and a Wikipedia search tool.

```python
def calculator_tool(calc):
    """ 
    A simple calculator tool that uses eval() to calculate the result of a given 
    expression. 
    """
    return "Observation: the calculation result is " + str(eval(calc))
```
The calculator tool is just a simple function that gets a string representing an expression and then returns 
the observation with the result of the expression.

Here is also the code for the Wikipedia tool:

```python
from mediawiki import MediaWiki

wikipedia = MediaWiki()

def wikipedia_tool(query: str) -> str:
    """
    Retrieves the summary of a Wikipedia article for a given query.

    Args:
        query: A string representing the title of a Wikipedia article.

    Returns:
        A string representing the summary of the article, or a preset string if the 
        article was not found.
    """
    try:
        wikipedia_page = wikipedia.page(query)
        return f"Observation: {wikipedia_page.summary}"
    except:
        return "Observation: The search didn't return any data"
```

This function uses the <a href="https://github.com/barrust/mediawiki" target="_blank">`pymediawiki`</a> package to get the summary of a Wikipedia article. The exact details are out of the scope of this guide, but you can read more about the package if you follow the link above. 

!!! info

    To run this function, you will need to install the `pymediawiki` package:
    ```
    pip install pymediawiki
    ```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stoyan-stoyanov/llmflows](https://github.com/stoyan-stoyanov/llmflows) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
