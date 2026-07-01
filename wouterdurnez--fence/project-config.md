---
trigger: always_on
description: Agents are where Fence really shines! We've built **production-ready agentic systems** using the ReAct (Reasoning + Acting) pattern. Agents can think, use tools, delegate to other agents, and maintain memory across conversations.
---

# 🤖 Agents

Agents are where Fence really shines! We've built **production-ready agentic systems** using the ReAct (Reasoning + Acting) pattern. Agents can think, use tools, delegate to other agents, and maintain memory across conversations.

---

## Agent Types

Fence provides three types of agents:

1. **`Agent`** - Classic ReAct agent with tool use and multi-level delegation
2. **`BedrockAgent`** - Native Bedrock tool calling with streaming support
3. **`ChatAgent`** - Conversational agent for multi-agent systems

---

## ReAct Agent

The classic agent implementation using the ReAct pattern: **Thought → Action → Observation → Answer**.

### Basic Example

```python
from fence.agents import Agent
from fence.models.openai import GPT4omini
from fence.tools.math import CalculatorTool

# Create an agent with tools
agent = Agent(
    identifier="math_wizard",
    model=GPT4omini(source="demo"),
    description="An agent that can perform calculations",
    tools=[CalculatorTool()],
    max_iterations=5
)

# Ask it something that requires tool use
result = agent.run("What is 1337 * 42 + 999?")
print(result)
```

**What happens:**

1. **[THOUGHT]** - Agent reasons about the problem
2. **[ACTION]** - Agent uses the calculator tool
3. **[OBSERVATION]** - Agent sees the tool result
4. **[ANSWER]** - Agent provides the final answer

### Agent Configuration

```python
from fence.agents import Agent
from fence.memory import DynamoDBMemory

agent = Agent(
    identifier="my_agent",              # Unique identifier
    model=GPT4omini(source="app"),      # LLM to use
    description="A helpful assistant",   # Agent description
    role="You are an expert analyst",    # System message role
    tools=[CalculatorTool()],            # Available tools
    delegates=[],                        # Other agents to delegate to
    memory=DynamoDBMemory(...),          # Memory backend
    environment={"api_key": "..."},      # Environment variables
    prefill="Let me help you with that", # Assistant prefill
    max_iterations=5,                    # Max reasoning loops
    iteration_timeout=30.0,              # Timeout per iteration
    log_agentic_response=True,           # Log agent thoughts
    are_you_serious=False                # Fun logging mode
)
```

### Multi-Level Agent Delegation

Agents can delegate to other specialized agents!

```python
from fence.agents import Agent
from fence.tools.math import CalculatorTool, PrimeTool
from fence.tools.text import SecretStringTool

# Create a specialist agent
math_agent = Agent(
    identifier="mathematician",
    model=GPT4omini(source="agent"),
    description="An agent specialized in mathematical operations",
    tools=[CalculatorTool(), PrimeTool()]
)

# Create a coordinator agent
coordinator = Agent(
    identifier="coordinator",
    model=GPT4omini(source="agent"),
    description="A coordinator that can delegate to specialists",
    delegates=[math_agent],
    tools=[SecretStringTool()]
)

# The coordinator will delegate math questions to the specialist
result = coordinator.run("Is 17 a prime number? Also, what's the secret string?")
```

**What happens:**

1. Coordinator receives the question
2. Recognizes it needs math expertise
3. **[DELEGATE]** to the math_agent
4. Math agent uses PrimeTool
5. Coordinator uses SecretStringTool
6. Coordinator combines results and answers

### Complex Hierarchies

Build multi-level agent hierarchies:

```python
# Level 3: Specialist agents
calculator_agent = Agent(
    identifier="calculator",
    model=GPT4omini(source="agent"),
    tools=[CalculatorTool()],
    description="Performs calculations"
)

text_agent = Agent(
    identifier="text_processor",
    model=GPT4omini(source="agent"),
    tools=[TextInverterTool()],
    description="Processes text"
)

# Level 2: Department coordinators
math_coordinator = Agent(
    identifier="math_dept",
    model=GPT4omini(source="agent"),
    delegates=[calculator_agent],
    description="Coordinates mathematical tasks"
)

# Level 1: Master coordinator
master = Agent(
    identifier="master",
    model=GPT4omini(source="agent"),
    delegates=[math_coordinator, text_agent],
    description="Master coordinator for all tasks"
)

# The master will delegate down the hierarchy as needed
result = master.run("Calculate 42 * 1337, then invert the result as text")
```

---

## BedrockAgent

Native Bedrock agent with streaming support and event handlers. Uses Bedrock's native tool calling API for better performance.

### Basic Example

```python
from fence.agents.bedrock import BedrockAgent
from fence.models.bedrock import Claude35Sonnet
from fence.tools.text import TextInverterTool

# Create Bedrock agent
agent = BedrockAgent(
    identifier="text_processor",
    model=Claude35Sonnet(region="us-east-1"),
    tools=[TextInverterTool()],
    description="An agent that processes text"
)

result = agent.run("Invert the text: Hello World!")
print(result.answer)
```

### Event Handlers

BedrockAgent supports event handlers for real-time visibility:

```python
from fence.agents.bedrock import BedrockAgent

# Define event handlers

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [WouterDurnez/fence](https://github.com/WouterDurnez/fence) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
