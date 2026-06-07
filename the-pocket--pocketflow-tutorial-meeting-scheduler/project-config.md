---
trigger: always_on
description: ================================================
---



================================================
File: docs/guide.md
================================================
---
layout: default
title: "Design Guidance"
parent: "Apps"
nav_order: 1
---

# LLM System Design Guidance


## Example LLM Project File Structure

```
my_project/
├── main.py
├── flow.py
├── utils/
│   ├── __init__.py
│   ├── call_llm.py
│   └── search_web.py
├── tests/
│   ├── __init__.py
│   ├── test_flow.py
│   └── test_nodes.py
├── requirements.txt
└── docs/
    └── design.md
```


### `docs/`

Store the documentation of the project.

It should include a `design.md` file, which describes 
- Project requirements
- Required utility functions
- High-level flow with a mermaid diagram
- Shared memory data structure
- For each node, discuss
  - Node purpose and design (e.g., should it be a batch or async node?)
  - How the data shall be read (for `prep`) and written (for `post`)
  - How the data shall be processed (for `exec`)

### `utils/`

Houses functions for external API calls (e.g., LLMs, web searches, etc.). 

It’s recommended to dedicate one Python file per API call, with names like `call_llm.py` or `search_web.py`. Each file should include:

- The function to call the API
- A main function to run that API call

For instance, here’s a simplified `call_llm.py` example:

```python
from openai import OpenAI

def call_llm(prompt):
    client = OpenAI(api_key="YOUR_API_KEY_HERE")
    response = client.chat.completions.create(
        model="gpt-4o",
        messages=[{"role": "user", "content": prompt}]
    )
    return response.choices[0].message.content

def main():
    prompt = "Hello, how are you?"
    print(call_llm(prompt))

if __name__ == "__main__":
    main()
```

### `main.py`

Serves as the project’s entry point.

### `flow.py`

Implements the application’s flow, starting with node followed by the flow structure.


### `tests/`

Optionally contains all tests. Use `pytest` for testing flows, nodes, and utility functions.
For example, `test_call_llm.py` might look like:

```python
from utils.call_llm import call_llm

def test_call_llm():
    prompt = "Hello, how are you?"
    assert call_llm(prompt) is not None
```

## System Design Steps

1. **Project Requirements**  
   - Identify the project's core entities.  
   - Define each functional requirement and map out how these entities interact step by step.

2. **Utility Functions**  
   - Determine the low-level utility functions you’ll need (e.g., for LLM calls, web searches, file handling).  
   - Implement these functions and write basic tests to confirm they work correctly.

3. **Flow Design**  
   - Develop a high-level process flow that meets the project’s requirements.  
   - Specify which utility functions are used at each step.  
   - Identify possible decision points for *Node Actions* and data-intensive operations for *Batch* tasks.  
   - Illustrate the flow with a Mermaid diagram.

4. **Data Structure**  
   - Decide how to store and update state, whether in memory (for smaller applications) or a database (for larger or persistent needs).  
   - Define data schemas or models that detail how information is stored, accessed, and updated.

5. **Implementation**  
   - Start coding with a simple, direct approach (avoid over-engineering at first).  
   - For each node in your flow:
     - **prep**: Determine how data is accessed or retrieved.  
     - **exec**: Outline the actual processing or logic needed.  
     - **post**: Handle any final updates or data persistence tasks.

6. **Optimization**  
   - **Prompt Engineering**: Use clear and specific instructions with illustrative examples to reduce ambiguity.  
   - **Task Decomposition**: Break large, complex tasks into manageable, logical steps.

7. **Reliability**  
   - **Structured Output**: Verify outputs conform to the required format. Consider increasing `max_retries` if needed.  
   - **Test Cases**: Develop clear, reproducible tests for each part of the flow.  
   - **Self-Evaluation**: Introduce an additional Node (powered by LLMs) to review outputs when the results are uncertain.

================================================
File: docs/agent.md
================================================
---
layout: default
title: "Agent"
parent: "Paradigm"
nav_order: 6
---

# Agent

For many tasks, we need agents that take dynamic and recursive actions based on the inputs they receive.
You can create these agents as **Nodes** connected by *Actions* in a directed graph using [Flow](./flow.md).


### Example: Search Agent

This agent:
1. Decides whether to search or answer
2. If searches, loops back to decide if more search needed
3. Answers when enough context gathered

```python
class DecideAction(Node):
    def prep(self, shared):
        context = shared.get("context", "No previous search")
        query = shared["query"]
        return query, context
        
    def exec(self, inputs):
        query, context = inputs
        prompt = f"""
Given input: {query}
Previous search results: {context}
Should I: 1) Search web for more info 2) Answer with current knowledge
Output in yaml:
```yaml
action: search/answer
reason: why this action
search_term: search phrase if action is search
```"""
        resp = call_llm(prompt)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [The-Pocket/PocketFlow-Tutorial-Meeting-Scheduler](https://github.com/The-Pocket/PocketFlow-Tutorial-Meeting-Scheduler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-07 -->
