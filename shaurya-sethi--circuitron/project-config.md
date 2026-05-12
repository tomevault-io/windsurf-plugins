---
trigger: always_on
description: Circuitron is an AI-powered PCB design accelerator that converts natural language requirements into SKiDL scripts, KiCad schematics, and PCB layouts using a multi-agent orchestration pipeline.
---

# Instructions for working on Circuitron

Circuitron is an AI-powered PCB design accelerator that converts natural language requirements into SKiDL scripts, KiCad schematics, and PCB layouts using a multi-agent orchestration pipeline.

## Architecture Overview

### Core Pipeline Flow
The system follows a sophisticated, multi-stage agent workflow defined in `circuitron/pipeline.py`. It is not a simple linear sequence but includes robust, nested correction loops.

1.  **Planner** → **Plan Editor** (optional user feedback loop)
2.  **Part Finder** → **Part Selector**
3.  **Documentation Agent** (gathers context via RAG)
4.  **Code Generator**
5.  **Validation & Correction Loop**:
    *   The **Validator** agent checks the generated code.
    *   If validation fails, the **Corrector** agent attempts to fix it. This loop continues until the code is valid or a retry limit is reached.
6.  **Runtime Check & Correction Loop**:
    *   A runtime check is performed in a Docker container.
    *   If it fails, the **Runtime Corrector** agent attempts a fix. This loop continues until the script runs successfully.
7.  **ERC (Electrical Rule Check) & Handling Loop**:
    *   The **ERC Handler** agent runs ERC checks.
    *   If errors or unapproved warnings are found, it attempts to fix them. This loop continues until ERC passes or warnings are explicitly accepted by the agent.
8.  **Final Execution**: The final, validated script is executed to produce KiCad files.

Each agent has specific responsibilities and uses the OpenAI Agents SDK with structured Pydantic models defined in `circuitron/models.py`.

### Key Components
- **Agents** (`circuitron/agents.py`): Specialized OpenAI agents for each pipeline stage.
- **MCP Server** (`circuitron/mcp_manager.py`): A single, shared connection for RAG documentation, code validation, and hallucination detection.
- **Tools** (`circuitron/tools.py`): Docker-isolated functions for calculations, KiCad library searches, and ERC checks.
- **UI** (`circuitron/ui/app.py`): A Rich-based terminal interface for progress tracking and interactive plan editing.

## Project Map (high‑value files)
- `circuitron/pipeline.py` – Orchestration and correction loops.
- `circuitron/agents.py` – Agent factory functions and `_tool_choice_for_mcp`.
- `circuitron/tools.py` – Function tools and Docker integration.
- `circuitron/models.py` – Pydantic schemas.
- `circuitron/docker_session.py` – Container/session/mount logic.
- `circuitron/mcp_manager.py` – Shared MCP server connection.
- `circuitron/prompts.py` – System instructions for each agent.
- `circuitron/ui/app.py` – Terminal UI.
- `circuitron/correction_context.py` – Correction attempt tracking.
- `circuitron/settings.py`, `circuitron/config.py` – Defaults and env bootstrap.
- `circuitron/cli.py` – CLI entrypoint.
- `tests/` – Unit/integration tests with mocks.
- `README.md`, `SETUP.md` – End‑to‑end setup and diagrams.
- `collab_progress/` – Mandatory progress notes and changelog entries.

## Documentation & Knowledge Sources

**CRITICAL: ALWAYS reference official documentation for OpenAI Agents SDK (And other libraries/frameworks) before implementing anything new or changing existing behavior.**

###  **Primary Sources**: 

> **Note:** If working in a cloud environment (not locally), you would not have `context7` access. In that case, rely on the official docs.

1. The available tools with the `context7` MCP server will provide you with documentation for ANY library/framework/language,etc in the world.
2. If `context7` is not available, your primary source for technical reference should be the official documentation available on the internet (if you have access)

When using any API, class, method, or tool:
1. Look up the relevant section in the official documentation.
2. Verify syntax, arguments, return types, and intended behavior.
3. Follow usage patterns recommended in the official examples.
4. If uncertain, re-check the official docs or ask the user for clarification — never guess or hallucinate API usage.

**SKiDL Documentation**
*   **URL**: [https://devbisme.github.io/skidl/](https://devbisme.github.io/skidl/)
*   **Use For**: Understanding SKiDL functions, classes, and circuit generation patterns.

## Development Patterns

### Agent Creation Pattern
Agents are created in `circuitron/agents.py` as pure functions. They are configured with a prompt, model, output type, tools, and model settings. When an agent needs RAG/validation, attach the shared MCP server. Use guardrails where appropriate.

```python
# From circuitron/agents.py
def create_planning_agent() -> Agent:
    """Create and configure the Planning Agent."""
  model_settings = ModelSettings(tool_choice="required")

  tools: list[Tool] = [execute_calculation]

  return Agent(
    name="Circuitron-Planner",
    instructions=PLAN_PROMPT,
    model=settings.planning_model,
    output_type=PlanOutput,
    tools=tools,
    # Optional: constrain inputs
    input_guardrails=[pcb_query_guardrail],
    # Attach MCP for agents that need RAG/validation (omit here if not needed):
    # mcp_servers=[mcp_manager.get_server()],
    model_settings=model_settings,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Shaurya-Sethi/circuitron](https://github.com/Shaurya-Sethi/circuitron) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
