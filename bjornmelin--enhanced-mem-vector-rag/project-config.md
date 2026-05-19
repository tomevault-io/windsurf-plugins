---
trigger: always_on
description: This project, "Enhanced Mem-Vector RAG," aims to build a sophisticated, **personal AI Agent system** focused on **advanced memory/knowledge management**, **software development assistance** (specifically targeting AI/ML, Python, AWS, Next.js), **research**, **learning**, and managing personal/work information.
---

# Enhanced Mem-Vector RAG - Claude Code Development Guide (v4 - Final Comprehensive Architecture)

## 🧠 Project Overview & Goal

This project, "Enhanced Mem-Vector RAG," aims to build a sophisticated, **personal AI Agent system** focused on **advanced memory/knowledge management**, **software development assistance** (specifically targeting AI/ML, Python, AWS, Next.js), **research**, **learning**, and managing personal/work information.

**Core Use Case:** Create a persistent, shared memory and knowledge fabric accessible by both the user and multiple AI agents. The system will assist with software development, research report generation, learning, and managing personal/work information, primarily interacting via **Claude Code** and other tools leveraging the **Model Context Protocol (MCP)**.

## 🏛️ Core Technologies & Final Stack Choices

- **Development Environment:** Claude Code with integrated MCP Servers.
- **Agent Orchestration:** LangGraph (Supervisor-Worker pattern).
- **Agent Framework:** LangChain (Agent logic, planning, tool integration).
- **Data Indexing/Connection & RAG Pipeline Framework:** **LlamaIndex** (Central framework for connecting LLMs to data sources, building RAG pipelines, indexing data into vector/graph stores, and orchestrating complex queries across them).
- **Long-Term Memory (LTM - Semantic/Episodic):**
  - **Vector Store:** Qdrant (High-performance, filtering, hybrid search).
  - **Memory Management Interface:** Mem0 (Open Source, self-hosted; intelligent interface primarily for vector memory, adds scoring, personalization. Works alongside LlamaIndex).
- **Procedural Memory (Workflows/Tasks):**
  - **Graph Database:** Neo4j.
  - **Interface/Framework:** Graphiti (Runs _on_ Neo4j for temporally-aware knowledge graphs). LlamaIndex `Neo4jGraphStore` can also be used for indexing/querying graph data.
- **Structured Data/Metadata:** Supabase (PostgreSQL) (User profiles, project metadata, relational data).
- **Knowledge Representation:** OWL (Ontologies managed in Git), RDF (Semantic annotations via `n10s` in Neo4j).
- **Embedding Generation:** Locally via models like OpenAI `text-embedding-3-small` / Cohere `embed-english-v3`.
- **Embedding Tooling:** FastEmbed (For efficient embedding during ingestion).
- **Retrieval Strategy:** Hybrid RAG (Qdrant vector + BM25 keyword) + KAG (Neo4j/Graphiti graph context) + Tavily Web Search fusion. **LlamaIndex orchestrates retrieval.**
- **Ingestion Pipeline:** LlamaIndex data loaders, Crawl4AI, Firecrawl, Scrapy, Playwright, Official APIs (Reddit, X, etc.), GitHub Webhooks, IMAP. Originals stored in AWS S3 / Supabase Storage.
- **MCP Server Framework:** **FastMCP (gofastmcp.com)** - The framework used to build the custom Python MCP server for this project.
- **UI/Interaction (Initial):** Primarily via MCP-enabled tools (Claude Code, etc.). Chainlit is a secondary option for direct interaction later, leveraging LlamaIndex integration.
- **Deployment (Initial):** Local execution. MCP server via `stdio`. Components (Qdrant, Neo4j, Supabase) via Docker.
- **Observability:** Grafana + Prometheus stack.
- **Versioning:** Git (Code/Ontology), Neo4j CDC (Graph Data).
- **Rules/Constraints:** Neo4j APOC triggers/constraints.
- **Package Management:** `uv` for Python.
- **Security:** Env Vars/Secrets Mgmt, OAuth 2.1/PAT for MCP, RBAC mapping.

_(Alternatives Considered & Rejected/Deferred: Zep, Memonto, A-MEM, Letta, Memoripy (for LTM); Memgraph (for Graph DB); Cache-Augmented Generation (CAG); External Rule Engines like Jena/RDFox)_.

## 🧰 Available MCP Servers (for Claude Code)

Claude Code has access to the following _external_ MCP servers:

- `arxiv-mcp-server`
- `browser`
- `context7`
- `desktop-commander`
- `firecrawl`
- `git`
- `github`
- `playwright`
- `repomix`
- `sequential-thinking`
- `supabase`
- `tavily`
- `time`
- **`memory`**: This is the **custom MCP server we are building** using the **FastMCP framework**. Its functions (`/search.hybrid`, `/graph.query`, `/memory.*`, `/rules.validate`, `/ingest.*`) interact with _our_ backend components (Qdrant/Mem0, Neo4j/Graphiti, Supabase, LlamaIndex-powered retrieval, Ingestion triggers).

## 🧭 Development Workflow & Memory Protocol

**🔄 Session Initialization (CRITICAL):**
_ALWAYS_ start every development session by refreshing your understanding of the project state:

1.  `memory.read_graph()` - Load the project's knowledge graph state from _our_ custom `memory` MCP server.
2.  `memory.search_nodes("Implementation Progress")` - Find the latest progress summary node.
3.  `memory.search_nodes("Recent Decisions")` - Review key architectural or implementation choice nodes.
4.  `git.git_status({"repo_path": "./"})` - Check the local Git working tree status.

**🧠 During Development (Continuous Memory Updates via _our_ `memory` MCP):**

- **Record Implementations:** As you write code (e.g., agent logic using LangChain, data loading using LlamaIndex, MCP endpoint using FastMCP):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BjornMelin/enhanced-mem-vector-rag](https://github.com/BjornMelin/enhanced-mem-vector-rag) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
