---
trigger: always_on
description: **Purpose**: AI-optimized execution guide for coding agents working with the Codemie codebase
---

# AGENTS.md

**Purpose**: AI-optimized execution guide for coding agents working with the Codemie codebase

---

## GUIDE IMPORTS

This document references detailed guides stored in `.codemie/guides/`. Use those guides as the primary source for implementation patterns and repo conventions.

### MANDATORY RULE: Check Guides First

**Before searching the codebase broadly or editing files**: classify the task, load the relevant P0 guides, check the documented patterns, then search code only as needed.

**Why**: the guides capture established architecture, implementation rules, and anti-patterns to avoid.

---

### Guide References by Category

**Agents & Tools**:
- Agent patterns: `.codemie/guides/agents/langchain-agent-patterns.md`
- Tool usage: `.codemie/guides/agents/agent-tools.md`
- Tool overview: `.codemie/guides/agents/tool-overview.md`
- Custom tools: `.codemie/guides/agents/custom-tool-creation.md`

**API Development**:
- REST patterns: `.codemie/guides/api/rest-api-patterns.md`
- Endpoint conventions: `.codemie/guides/api/endpoint-conventions.md`

**Architecture**:
- Layered architecture: `.codemie/guides/architecture/layered-architecture.md`
- Service layer: `.codemie/guides/architecture/service-layer-patterns.md`
- Project structure: `.codemie/guides/architecture/project-structure.md`

**Data & Database**:
- Database patterns: `.codemie/guides/data/database-patterns.md`
- Repository patterns: `.codemie/guides/data/repository-patterns.md`
- Database optimization: `.codemie/guides/data/database-optimization.md`
- Elasticsearch: `.codemie/guides/data/elasticsearch-integration.md`

**Development Practices**:
- Error handling: `.codemie/guides/development/error-handling.md`
- Logging patterns: `.codemie/guides/development/logging-patterns.md`
- Security patterns: `.codemie/guides/development/security-patterns.md`
- Performance: `.codemie/guides/development/performance-patterns.md`
- Configuration: `.codemie/guides/development/configuration-patterns.md`
- Setup guide: `.codemie/guides/development/setup-guide.md`
- Local testing: `.codemie/guides/development/local-testing.md`

**Testing**:
- Testing patterns: `.codemie/guides/testing/testing-patterns.md`
- API testing: `.codemie/guides/testing/testing-api-patterns.md`
- Service testing: `.codemie/guides/testing/testing-service-patterns.md`

**Integrations**:
- Cloud services: `.codemie/guides/integration/cloud-integrations.md`
- LLM providers: `.codemie/guides/integration/llm-providers.md`
- External services overview: `.codemie/guides/integration/external-services.md`
- Confluence: `.codemie/guides/integration/confluence-integration.md`
- Jira: `.codemie/guides/integration/jira-integration.md`
- X-ray: `.codemie/guides/integration/xray-integration.md`
- Google Docs: `.codemie/guides/integration/google-docs-integration.md`
- MCP integration: `.codemie/guides/integration/mcp-integration.md`

**Standards**:
- Code quality: `.codemie/guides/standards/code-quality.md`
- Git workflow: `.codemie/guides/standards/git-workflow.md`

**Workflows**:
- LangGraph workflows: `.codemie/guides/workflows/langgraph-workflows.md`

---

## INSTANT START

### 1. Critical Rules

| Rule | Trigger | Action Required |
|------|---------|-----------------|
| Check Guides First | Any new prompt or task | Always check relevant guides before broad search or edits |
| Testing | User explicitly asks to write, fix, or run tests | Only then work on tests |
| Git Ops | User explicitly asks to commit, push, or create a PR | Only then do git operations |
| VirtualEnv | Any Python or Poetry command | Activate `.venv` first |
| Shell | Any shell command | Use bash/Linux command syntax |

**Recovery**: if Python tooling fails, check the troubleshooting section and verify the virtualenv is active.

### 2. Task Classifier

| Category | Intent | P0 Guide |
|----------|--------|----------|
| Architecture | System structure, file placement | `layered-architecture.md`, `project-structure.md` |
| Agents & Tools | Create or modify agents, add tools | `langchain-agent-patterns.md` |
| API | Create endpoints, schemas, validation | `rest-api-patterns.md` |
| Database | Queries, models, repositories, optimization | `database-patterns.md`, `repository-patterns.md` |
| Search | Elasticsearch, vector search | `elasticsearch-integration.md` |
| Error Handling | Exceptions, failure paths | `error-handling.md` |
| Logging | Add logs, debug flows | `logging-patterns.md` |
| Security | Auth, validation, permissions | `security-patterns.md` |
| Performance | Optimize code, async patterns | `performance-patterns.md` |
| Configuration | Settings, environment variables | `configuration-patterns.md` |
| Testing | Write or fix tests | `testing-patterns.md` |
| Cloud | AWS, Azure, GCP integration | `cloud-integrations.md` |
| LLM | Model configuration, provider selection | `llm-providers.md` |
| Confluence/Jira/X-ray/GDocs | External service indexing or integration | Service-specific integration guides |
| MCP | Model Context Protocol | `mcp-integration.md` |
| Code Quality | Linting, formatting, implementation hygiene | `code-quality.md` |
| Git | Commits, PRs, branch workflow | `git-workflow.md` |
| Workflows | LangGraph workflows | `langgraph-workflows.md` |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [codemie-ai/codemie](https://github.com/codemie-ai/codemie) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
