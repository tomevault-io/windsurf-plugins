---
trigger: always_on
description: This repository powers the WayFlow agent platform. These instructions exist so
---

# WayFlow Core – Assistant Operating Manual

This repository powers the WayFlow agent platform. These instructions exist so
coding assistants act like productive teammates: fast, accurate, and aligned
with how the project actually runs. Read the “Zero-Step Contract” before every
task. Treat everything else as the living field guide.


## Zero-Step Contract (always obey)

1. **Plan mode** – Max 4 bullets, ≤6 words each. End with open questions or
   write `Questions: None`.
2. **Validation loop** – `pytest wayflowcore/tests/<specific_file>`. Only run the tests associated to the feature that is being developed.
3. **Bounded actions** – No background processes, no guessing commands. Ask for missing info in the plan.
4. **Scope discipline** – Touch only relevant files; mirror each code change with matching tests/docs when applicable.
5. **Report** - Summarize your actions (a few words) and add entry in changelog (`docs/wayflowcore/source/core/changelog.rst`)


## Project Snapshot

- **Purpose** – Provide a composable runtime for agents, flows, swarms, and tooling across multiple LLM providers (OCI, OpenAI, Ollama, VLLM, etc.).
- **Packaging** – Ships to PyPI as `wayflowcore`; source under `src/wayflowcore/`.
- **Key abstractions** – Agent (`agent.py`), Flow (`flow.py`), Swarm (`swarm.py`), Manager worker (`managerworkers.py`), OCI Agent (`ociagent.py`).
- **Primary services** – Tooling layer (`tools/`), model integrations (`models/`), agent server (`agentserver/`), persistence (`datastore/`), evaluation/telemetry (`evaluation/`, `events/`, `tracing/`).
- **AgentSpec adapters** - Layer to convert agent spec components into wayflow and the other way around. AgentSpec is a python SDK for an agentic specification. Most abstractions are common between AgentSpec and WayFlow, but have small differences.

Different types of conversational components can be built, such as:
- Agents for autonomous and/or conversational task completion with tools
- Flows for completing tasks with a structured sequence of steps
- Multi-agent patterns (Swarms, ManagerWorkers)

All conversational components have inputs and outputs. They run using a conversation:
```python
from wayflowcore.executors.executionstatus import FinishedStatus, UserMessageRequestStatus, ToolRequestStatus

conversation = component.start_conversation()
status = conversation.execute()
if isinstance(status, UserMessageRequestStatus):
    status.submit_user_response('response_from_the_user')
elif isinstance(status, FinishedStatus):
    outputs = status.output_values
elif isinstance(status, ToolRequestStatus):
    for tool_request in status.tool_requests:
        status.submit_tool_result("tool_execution_result")
status = conversation.execute() # to continue given the results ...
```

## Repository Map

| Area                        | Location                                                                                                       | Notes                                                                                            |
|-----------------------------|----------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------|
| Agents & Conversations      | `agent.py`, `conversation.py`, `messagelist.py`, `tokenusage.py`                                               | Manages agent prompts, descriptors, token accounting.                                            |
| Flows & Steps               | `flow.py`, `steps/`, `flowhelpers.py`, `controlconnection.py`, `dataconnection.py`                             | Flow graphs, transitions, data edges, descriptor resolution.                                     |
| Swarms & A2A                | `swarm.py`, `a2a/`, `ociagent.py`, `managerworkers.py`, `_threading.py`                                        | Multi-agent orchestration, A2A agent protocol, OCI agents                                        |
| Execution logic             | `executors/`                                                                                                   | Execution logic of all classes.                                                                  |
| Tools                       | `tools/`, `toolbox.py`, `servertools.py`, `remotetools.py`, `toolhelpers.py`, `mcp/`                           | Tool definitions (client/server), MCP adapters (Model context protocol), conversion helpers.     |
| Models                      | `models/`                                                                                                      | OCI (`ocigenaimodel.py`), OpenAI (`openaimodel.py`), Ollama, VLLM, factory + generation configs. |
| Prompts & Templates         | `templates/`, `outputparser.py`, `transforms/`,                                                                | Schema generators, debugging scripts, regression tracking, flow validation notes.                |
| Data handling & Persistence | `datastore/`                                                                                                   | Handles connection to databases, Oracle DB/Postgres persistence.                                 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oracle/wayflow](https://github.com/oracle/wayflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
