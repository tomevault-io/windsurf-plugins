---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is a **meta portfolio repository** serving as the professional hub for AI research into LLM-driven developer tooling. It follows a structured approach with:
- Research narrative and timeline documentation
- Architecture Decision Records (ADRs) for technical decisions
- Reproducible demos and evaluation methodologies
- Links to two active phase repositories for implementation details

## Documentation Commands

**Build and serve documentation locally:**
```bash
pip install mkdocs-material
mkdocs serve
```

**Deploy to GitHub Pages:**
```bash
mkdocs gh-deploy --force
```

Documentation is automatically deployed via GitHub Actions on push to `main` branch.

## GitHub Repository Strategy

This repository implements a professional research presentation strategy:
- **Hub Structure**: Central documentation linking to phase-specific implementation repositories
- **GitHub Pages**: Polished documentation site using MkDocs Material theme
- **GitHub Features**: Projects board for roadmap tracking, Releases for milestone summaries, Discussions for research notes
- **Professional Presentation**: Timeline visualization, reproducible instructions, demo scenarios

## Architecture Overview

This repository documents AI research focused on LLM-driven developer tooling across multiple phases:

**Core Architecture Pattern (ADR-0002):**
- Extraction → Retrieval → Planning → Execution → Validation → Formatting
- Hybrid retrieval combining semantic similarity with symbolic metadata filters
- Symbolic planner for endpoint/tool selection based on entities, roles, and constraints
- Constraint trees with AND/OR logic and provenance-aware fallbacks

**Phase 1 - TMDB Chatbox:**
- Natural language movie/TV show query system using TMDB API
- Multi-entity constraint solving with role-aware validation
- ChromaDB RAG API endpoint routing using sentence transformers
- Progressive constraint relaxation with comprehensive logging
- Repository: https://github.com/FinnMacCumail/tmdbGPT

**Phase 2 - NetBox MCP Server (Shared Infrastructure):**
- Official NetBoxLabs MCP server with 3 generic tools (get_objects, get_object_by_id, get_changelogs)
- Field filtering for token optimization
- Read-only interface to NetBox data
- Shared foundation infrastructure used by both Phase 4 agent frameworks
- Repository: https://github.com/netboxlabs/netbox-mcp-server

**Phase 3 - OpenAI Orchestration ❌ FAILED:**
- Attempted multi-agent orchestration with 0% success rate
- See ADR-0013 for failure analysis
- Individual NetBox tools achieved 93.8% success rate

**Phase 4 - Agent Framework Comparison ✅ COMPLETED:**
- **Research Focus**: Empirical comparison of flexible vs production-ready agent frameworks
- **Implementation A - Deepagents**: LangChain-based, maximum flexibility, DIY orchestration
  - Repository: https://github.com/FinnMacCumail/deepagents
  - Features: Dynamic tool discovery, LangGraph orchestration, sub-agent delegation
- **Implementation B - Claude SDK**: Anthropic SDK, production-ready, managed orchestration
  - Repository: https://github.com/FinnMacCumail/claude-agentic-netbox
  - Features: MCP protocol, WebSocket streaming, built-in permission system
- **Key Finding**: Framework choice is context-dependent; both successfully build production agents with different trade-offs

## Development Milestones

### Completed Phases:
1. **Phase 1 ✅**: TMDB Chatbox - Natural language movie/TV query system
2. **Phase 2 ✅**: NetBox MCP Server - Official NetBoxLabs MCP server with 3 generic tools
3. **Phase 3 ❌**: OpenAI Orchestration - Failed with 0% success rate
4. **Phase 4 ✅**: Agent Framework Comparison Study
   - **Research Achievement**: Empirical validation of framework trade-offs
   - **Implementation A**: [Deepagents NetBox Agent](https://github.com/FinnMacCumail/deepagents) - Flexible, research-oriented
   - **Implementation B**: [Claude SDK NetBox Agent](https://github.com/FinnMacCumail/claude-agentic-netbox) - Production-ready, managed
   - **Key Insight**: Framework choice is context-dependent (flexibility vs convenience spectrum)

### Upcoming Milestones:
5. **Phase 5**: Neo4j graph database integration for instant complex queries
6. **Phase 6**: RAG-powered semantic intelligence with institutional memory
7. **Phase 7**: Advanced analytics platform using graph algorithms

**Phase 4 Framework Comparison Achievements:**
- Empirical comparison of two production agent implementations
- Development time metrics: 46h (Deepagents) vs 2.5h (Claude SDK)
- Maintenance burden analysis: 8-12h/month vs 2-4h/month
- Framework selection guidance based on project context
- Validated that both approaches successfully build production agents

## Repository Structure

Complete documentation organization following research best practices:

```
ai-research-2025/
├─ README.md                    # Project overview and links
├─ RESEARCH_LOG.md             # Chronological findings and pivots
├─ CHANGELOG.md                # Conventional changelog for releases
├─ CITATION.cff                # Academic citation metadata
├─ LICENSE                     # Open source license

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FinnMacCumail/rtf-research](https://github.com/FinnMacCumail/rtf-research) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
