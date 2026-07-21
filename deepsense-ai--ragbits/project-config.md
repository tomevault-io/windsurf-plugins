---
trigger: always_on
description: Let's build a multi-agent system for automated trip planning with Ragbits. In this tutorial, we'll create:
---

# Tutorial: Multi-Agent System with A2A and MCP

Let's build a multi-agent system for automated trip planning with Ragbits. In this tutorial, we'll create:

1. A Flight Finder Agent that searches for available flights
2. A City Explorer Agent that gathers information about destinations
3. An Orchestrator Agent that coordinates both agents to create comprehensive trip plans

**What you'll learn:**

- How to create specialized agents with [tools/function calling](https://platform.openai.com/docs/guides/function-calling?api-mode=responses)
- How to use the [Model Context Protocol (MCP)](https://modelcontextprotocol.io/introduction) for external data
- How to expose agents through [Agent-to-Agent (A2A)](https://github.com/a2aproject/A2A) protocol
- How to build an orchestrator that manages conversation context

## Configuring the environment

Install the latest Ragbits via `pip install -U ragbits[a2a,mcp]` to follow along.

During development, we will use OpenAI's `gpt-4.1` model. To authenticate, Ragbits will look into your `OPENAI_API_KEY`. You can easily swap this with [other providers](../how-to/llms/use_llms.md).

!!! tip "Recommended: Set up OpenTelemetry tracing to understand what's happening under the hood."
    OpenTelemetry is an LLMOps tool that natively integrates with Ragbits and offer explainability and experiment tracking. In this tutorial, you can use OpenTelemetry to visualize prompts and optimization progress as traces to understand the Ragbits' behavior better. Check the full setup guide [here](../how-to/audit/use_tracing.md/#using-opentelemetry-tracer).

## Building the Flight Finder Agent

We start by defining the [prompt](../how-to/prompts/use_prompting.md) that will lead this agent.

```py title="flight_agent.py"
from pydantic import BaseModel
from ragbits.core.prompt import Prompt

--8<-- "examples/agents/a2a/flight_agent.py:37:53"

print(FlightPrompt(FlightPromptInput(input="I need to fly from New York to Paris. What flights are available?")).chat)
```

```json
[{'role': 'system', 'content': 'You are a helpful travel assistant that finds available flights between two cities.'},
{'role': 'user', 'content': 'I need to fly from New York to Paris. What flights are available?'}]
```

Next, we [define a tool](../how-to/llms/use_tools_with_llms.md) that will provide flight information. **Note**: in a real application, you'd connect to the actual flight APIs:

```py title="flight_agent.py"
import json

--8<-- "examples/agents/a2a/flight_agent.py:11:34"
```

This agent will call this function as needed, and the results will be injected back to the conversation.

Now let's create the agent and test it:

```py title="flight_agent.py"
from ragbits.agents import Agent
from ragbits.core.llms import LiteLLM

--8<-- "examples/agents/a2a/flight_agent.py:55:59"

async def main() -> None:
    result = await flight_agent.run(FlightPromptInput(input="I need to fly from New York to Paris. What flights are available?"))
    print(result.content)

--8<-- "examples/agents/a2a/flight_agent.py:71:74"
```

Run it:

```bash
python flight_agent.py
```

A typical response looks like this:
```text
Here are some available flights from New York to Paris:

1. **British Airways**
   - **Departure:** 10:00 AM
   - **Arrival:** 10:00 PM

2. **Delta**
   - **Departure:** 1:00 PM
   - **Arrival:** 1:00 AM
```

**Please note** that the results may differ among the runs due to undeterministic nature of LLM.

!!! example "Try it yourself"
    You can try to connect this agents to a real flight API, such as [aviationstack](https://aviationstack.com/).

## Building the City Explorer Agent

Let's create a City Explorer Agent that gather and synthesize city information from the internet. Again we start with the prompt:

```py title="city_explorer_agent.py"
from pydantic import BaseModel
from ragbits.core.prompt import Prompt

--8<-- "examples/agents/a2a/city_explorer_agent.py:9:31"
```

Now define the agent, We will not [build an MCP server from scratch](https://github.com/modelcontextprotocol/python-sdk?tab=readme-ov-file#quickstart), but run an already existing one - [Web Fetcher](https://github.com/modelcontextprotocol/servers/tree/main/src/fetch). Start by installing it with:

```bash
pip install mcp-server-fetch
```

```py title="city_explorer_agent.py"
from ragbits.agents import Agent
from ragbits.agents.mcp import MCPServerStdio
from ragbits.core.llms import LiteLLM
from ragbits.core.prompt import Prompt
--8<-- "examples/agents/a2a/city_explorer_agent.py:34:47"
        result = await city_explorer_agent.run(CityExplorerPromptInput(input="Tell me something interesting about Paris."))
        print(result.content)

--8<-- "examples/agents/a2a/city_explorer_agent.py:59:63"
```

Test this agent by running:
```bash
python city_explorer_agent.py
```

```text
Paris is the capital and largest city of France, located in the Île-de-France region. Renowned for its historical landmarks, the city is a significant cultural and economic center in Europe. Key attractions include the Eiffel Tower, Notre-Dame Cathedral, the Louvre Museum, and the Arc de Triomphe. Known for its romantic ambiance, Paris is often referred to as "The City of Light."


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [deepsense-ai/ragbits](https://github.com/deepsense-ai/ragbits) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
