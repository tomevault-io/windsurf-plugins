---
trigger: always_on
description: url = '/features/agents'
---

+++
disableToc = false
title = "Agents"
weight = 20
url = '/features/agents'
+++

![The in-process agent loop: agents call LocalAI's own chat API in a loop, streaming progress over SSE](/images/diagrams/agents-loop.png)

LocalAI includes a built-in agent platform powered by [LocalAGI](https://github.com/mudler/LocalAGI). Agents are autonomous AI entities that can reason, use tools, maintain memory, and interact with external services, all running locally as part of the LocalAI process.

LocalAGI is embedded in LocalAI. There is nothing separate to install or run.

{{% notice info %}}
**Looking for something else?** LocalAI has three related agentic features that are easy to confuse:

- **Agents** (this page): autonomous agents you build that reason, use tools, and act on their own. Start here if you want to create an agent.
- **LocalAI Assistant** ({{% relref "features/localai-assistant" %}}): an admin chat modality for administering LocalAI itself (install models, manage backends) by chatting.
- **MCP** ({{% relref "features/mcp" %}}): a way to give a model external tools through the Model Context Protocol.
{{% /notice %}}

{{% notice tip %}}
New to agents? The [Build your first agent]({{% relref "getting-started/first-agent" %}}) walkthrough takes you from an empty Agents page to an agent that answers a message and uses one tool.
{{% /notice %}}

## Overview

The agent system provides:

- **Autonomous agents** with configurable goals, personalities, and capabilities
- **Tool/Action support** - agents can execute actions (web search, code execution, API calls, etc.)
- **Knowledge base (RAG)** - per-agent collections with document upload, chunking, and semantic search
- **Skills system** - reusable skill definitions that agents can leverage, with git-based skill repositories
- **SSE streaming** - real-time chat with agents via Server-Sent Events
- **Import/Export** - share agent configurations as JSON files
- **Agent Hub** - browse and download ready-made agents from [agenthub.localai.io](https://agenthub.localai.io)
- **Web UI** - full management interface for creating, editing, chatting with, and monitoring agents

## Getting Started

Agents are enabled by default. To disable them, set:

```bash
LOCALAI_DISABLE_AGENTS=true
```

### Creating an Agent

1. Navigate to the **Agents** page in the web UI
2. Click **Create Agent** or import one from the [Agent Hub](https://agenthub.localai.io)
3. Configure the agent's name, model, system prompt, and actions
4. Save and start chatting

### Importing an Agent

You can import agent configurations from JSON files:

1. Download an agent configuration from the [Agent Hub](https://agenthub.localai.io) or export one from another LocalAI instance
2. On the **Agents** page, click **Import**
3. Select the JSON file - you'll be taken to the edit form to review and adjust the configuration before saving
4. Click **Create Agent** to finalize the import

## Configuration

### Environment Variables

All agent-related settings can be configured via environment variables:

| Variable | Default | Description |
|----------|---------|-------------|
| `LOCALAI_DISABLE_AGENTS` | `false` | Disable the agent pool feature entirely |
| `LOCALAI_AGENT_POOL_API_URL` | _(self-referencing)_ | Default API URL for agents. By default, agents call back into LocalAI's own API (`http://127.0.0.1:<port>`). Set this to point agents to an external LLM provider. |
| `LOCALAI_AGENT_POOL_API_KEY` | _(LocalAI key)_ | Default API key for agents. Defaults to the first LocalAI API key. Set this when using an external provider. |
| `LOCALAI_AGENT_POOL_DEFAULT_MODEL` | _(empty)_ | Default LLM model for new agents |
| `LOCALAI_AGENT_POOL_MULTIMODAL_MODEL` | _(empty)_ | Default multimodal (vision) model for agents |
| `LOCALAI_AGENT_POOL_TRANSCRIPTION_MODEL` | _(empty)_ | Default transcription (speech-to-text) model for agents |
| `LOCALAI_AGENT_POOL_TRANSCRIPTION_LANGUAGE` | _(empty)_ | Default transcription language for agents |
| `LOCALAI_AGENT_POOL_TTS_MODEL` | _(empty)_ | Default TTS (text-to-speech) model for agents |
| `LOCALAI_AGENT_POOL_STATE_DIR` | _(data path)_ | Directory for persisting agent state. Defaults to `LOCALAI_DATA_PATH` if set, otherwise falls back to `LOCALAI_CONFIG_DIR` |
| `LOCALAI_AGENT_POOL_TIMEOUT` | `5m` | Default timeout for agent operations |
| `LOCALAI_AGENT_POOL_ENABLE_SKILLS` | `false` | Enable the skills service |
| `LOCALAI_AGENT_POOL_VECTOR_ENGINE` | `chromem` | Vector engine for knowledge base (`chromem` or `postgres`) |
| `LOCALAI_AGENT_POOL_EMBEDDING_MODEL` | `granite-embedding-107m-multilingual` | Embedding model for knowledge base |
| `LOCALAI_AGENT_POOL_CUSTOM_ACTIONS_DIR` | _(empty)_ | Directory for custom action plugins |
| `LOCALAI_AGENT_POOL_DATABASE_URL` | _(empty)_ | PostgreSQL connection string for collections (required when vector engine is `postgres`) |
| `LOCALAI_AGENT_POOL_MAX_CHUNKING_SIZE` | `400` | Maximum chunk size for document ingestion |
| `LOCALAI_AGENT_POOL_CHUNK_OVERLAP` | `0` | Overlap between document chunks |
| `LOCALAI_AGENT_POOL_ENABLE_LOGS` | `false` | Enable detailed agent logging |
| `LOCALAI_AGENT_POOL_COLLECTION_DB_PATH` | _(empty)_ | Custom path for the collections database |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mudler/LocalAI](https://github.com/mudler/LocalAI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
