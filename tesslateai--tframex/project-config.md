---
trigger: always_on
description: Agents are the core intelligent actors in TFrameX. They process inputs, make decisions, execute tools, and collaborate with other agents to accomplish complex tasks.
---


# Agents

Agents are the core intelligent actors in TFrameX. They process inputs, make decisions, execute tools, and collaborate with other agents to accomplish complex tasks.

## What is an Agent?

An agent in TFrameX is an autonomous entity that:
- Receives and processes natural language inputs
- Uses an LLM to reason about tasks
- Executes tools to interact with external systems
- Maintains conversation context
- Can delegate to other agents

## Types of Agents

### LLMAgent

The most common agent type, powered by a language model:

```python
from tframex.agents.llm_agent import LLMAgent
from tframex.util.llms import OpenAIChatLLM

agent = LLMAgent(
    name="Assistant",
    description="A helpful AI assistant",
    llm=OpenAIChatLLM(),
    system_prompt="You are a helpful assistant. Be concise and friendly.",
    tools=["calculator", "web_search"],
    callable_agents=["Specialist"],
    max_tool_iterations=10
)
```

**Key Features:**
- Full LLM reasoning capabilities
- Tool execution
- Agent delegation
- Memory management
- Configurable behavior

### ToolAgent

A lightweight agent that directly executes a tool without LLM reasoning:

```python
from tframex.agents.tool_agent import ToolAgent

# Wrap a tool as an agent
tool_agent = ToolAgent(
    name="Calculator",
    tool_name="calculate",
    description="Direct calculation agent"
)
```

**Use Cases:**
- Performance-critical operations
- Deterministic workflows
- Tool exposure in multi-agent systems

### Custom Agents

Extend `BaseAgent` for specialized behavior:

```python
from tframex.agents.base import BaseAgent
from tframex.models.primitives import Message

class CustomAgent(BaseAgent):
    async def run(self, input_message: Message, **kwargs) -> Message:
        # Custom processing logic
        result = await self.process_custom_logic(input_message)
        return Message(role="assistant", content=result)
    
    async def process_custom_logic(self, message: Message) -> str:
        # Your implementation
        return "Custom response"
```

## Agent Configuration

### Basic Configuration

```python
agent = LLMAgent(
    name="ResearchAssistant",  # Unique identifier
    description="Specializes in research and analysis",  # Used by other agents
    llm=llm_instance,  # Language model to use
    system_prompt="You are a research specialist..."  # Defines behavior
)
```

### Advanced Configuration

```python
agent = LLMAgent(
    name="AdvancedAssistant",
    description="Feature-rich assistant",
    llm=custom_llm,
    
    # Tool configuration
    tools=["web_search", "calculator", "file_reader"],
    
    # Agent delegation
    callable_agents=["Researcher", "Writer", "Analyst"],
    
    # Memory configuration
    memory_store=CustomMemoryStore(max_messages=100),
    
    # Behavior configuration
    system_prompt="Complex prompt with {custom_var}",
    additional_prompt_variables={"custom_var": "dynamic value"},
    strip_think_tags=True,  # Remove <think> tags from output
    max_tool_iterations=15,  # Maximum tool calls per interaction
    
    # MCP integration
    mcp_tools_from_servers=["server1", "server2"]
)
```

## System Prompts

System prompts define agent behavior and capabilities:

### Effective System Prompt Structure

```python
system_prompt = """You are a [ROLE] specializing in [DOMAIN].

Your capabilities:
1. [CAPABILITY 1]: Description
2. [CAPABILITY 2]: Description
3. [CAPABILITY 3]: Description

Available tools:
- tool_name: What it does
- tool_name: What it does

Guidelines:
- [GUIDELINE 1]
- [GUIDELINE 2]
- [GUIDELINE 3]

Examples:
User: [Example query]
You: [Example response using tools]

Remember: [KEY REMINDERS]"""
```

### Real Example

```python
system_prompt = """You are a Financial Analyst specializing in investment research.

Your capabilities:
1. **Market Analysis**: Analyze market trends and provide insights
2. **Risk Assessment**: Evaluate investment risks and opportunities
3. **Report Generation**: Create comprehensive investment reports

Available tools:
- get_stock_data: Retrieve real-time stock information
- calculate_metrics: Compute financial metrics (P/E, ROI, etc.)
- generate_chart: Create visualization of financial data
- save_report: Save analysis reports to file

Guidelines:
- Always cite data sources
- Provide balanced analysis (pros and cons)
- Use appropriate financial terminology
- Quantify risks when possible

When asked about an investment:
1. Gather current market data
2. Calculate relevant metrics
3. Assess risks and opportunities
4. Provide a clear recommendation

Remember: Past performance doesn't guarantee future results. Always include appropriate disclaimers."""
```

## Tool Integration

Agents can use tools to extend their capabilities:

```python
# Define tools
@app.tool(description="Search financial news")
async def search_news(query: str, days_back: int = 7) -> str:
    # Implementation
    return news_results

@app.tool(description="Get stock quote")
async def get_quote(symbol: str) -> dict:
    # Implementation
    return {"symbol": symbol, "price": 150.25, "change": 2.5}

# Agent uses tools
analyst = LLMAgent(
    name="FinancialAnalyst",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TesslateAI/TFrameX](https://github.com/TesslateAI/TFrameX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
