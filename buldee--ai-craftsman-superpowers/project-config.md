---
trigger: always_on
description: The plugin provides **11 agents** organized in two categories:
---

# Agents Reference

The plugin provides **11 agents** organized in two categories:
- **4 Reviewers** - read-only analysis and code review
- **7 Craftsmen** - implementation specialists with domain expertise

## How to Use Agents

**Reviewers** are invoked for code review:

```
> Review this PR with the architecture-reviewer agent

> Run ai-engineer on the ML pipeline code
```

**Craftsmen** are invoked for implementation tasks:

```
> Use backend-craftsman to implement this use case

> Ask the architect to validate this design
```

**Agent Teams** (experimental, v1.5.0): Enable `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` to create multi-agent teams. The `team-lead` agent orchestrates other agents as teammates.

---

## Craftsman Agents (v1.5.0)

### team-lead

**Model**: Sonnet | **Effort**: high | **Memory**: user | **Max Turns**: 50

**Mission**: Orchestrator that delegates, challenges decisions, and validates deliverables. **Never codes directly.**

**Skills**: plan, challenge, verify

**Behavior**:
- Decomposes complex tasks into subtasks for specialists
- Challenges architectural decisions before implementation
- Validates deliverables against specifications
- Ensures conventional commits and test coverage

---

### backend-craftsman

**Model**: Sonnet | **Effort**: high | **Memory**: project | **Max Turns**: 30

**Mission**: PHP/Symfony implementation expert.

**Skills**: entity, usecase, spec, test

**Expertise**:
- Symfony 7.4/8, API Platform 4, Doctrine ORM
- DDD tactical patterns (Aggregates, Value Objects, Domain Events)
- Messenger/Scheduler patterns
- References: symfony.com/doc, api-platform.com/docs/symfony/

**Mandatory Rules**: `strict_types`, `final` classes, private constructors, no setters, Clock abstraction.

---

### frontend-craftsman

**Model**: Sonnet | **Effort**: high | **Memory**: project | **Max Turns**: 30

**Mission**: React/TypeScript implementation expert.

**Skills**: component, hook, spec, test

**Expertise**:
- React 19, TypeScript 5, Tailwind, shadcn/ui, TanStack Query
- 65 Vercel React best practices (waterfalls, bundle, server, client, re-renders)
- React 19 composition patterns (Server Components, Actions, use())

**Mandatory Rules**: No `any`, `readonly` by default, branded types, named exports only.

---

### architect

**Model**: Sonnet | **Effort**: high | **Memory**: project | **Max Turns**: 20

**Mission**: DDD/Clean Architecture validation. **Read-only - cannot edit or write files.**

**Skills**: design, challenge

**Disallowed Tools**: Edit, Write

**Validates**:
- Strategic DDD (bounded contexts, context maps, ubiquitous language)
- Tactical DDD (aggregates, entities, value objects, domain events)
- Clean Architecture layer dependencies
- CQRS and Event-Driven patterns

---

### ai-engineer

**Model**: Sonnet | **Effort**: high | **Memory**: project | **Max Turns**: 30

**Mission**: AI/ML implementation specialist.

**Skills**: rag, agent-design, mlops

**Expertise**:
- RAG pipelines (chunking, embeddings, retrieval, generation)
- LLM integration (Claude, OpenAI)
- MCP server design and implementation
- Agent patterns (3P: Perceive/Plan/Perform)

---

### api-craftsman

**Model**: Sonnet | **Effort**: high | **Memory**: project | **Max Turns**: 30

**Mission**: Senior API architect.

**Expertise**:
- API Platform 4, REST/HATEOAS standards
- OpenAPI specification, JSON-LD/Hydra
- API security (OAuth2, JWT)
- API design reviews and RESTful architecture decisions

---

### ui-ux-director

**Model**: Sonnet | **Effort**: high | **Memory**: project | **Max Turns**: 20

**Mission**: UX quality and accessibility guardian.

**Validates**:
- WCAG 2.1 AA compliance (color contrast, keyboard nav, screen readers)
- Design token systems (spacing, typography, colors)
- Data visualization best practices
- SaaS dashboard UX patterns

---

### doc-writer

**Model**: Haiku (cost-optimized) | **Effort**: medium | **Memory**: project | **Max Turns**: 20

**Mission**: Technical documentation specialist.

**Produces**:
- ADRs (Architecture Decision Records)
- README and CHANGELOG entries
- API documentation (OpenAPI)
- Runbooks and operational guides

**Verification**: Cross-references documentation against actual code to detect drift.

---

## Reviewer Agents

### Core Pack

### architecture-reviewer

**Mission**: Review code against Clean Architecture principles.

**Checks**:
- Dependencies point inward (domain has no external imports)
- Domain layer purity
- Use case single responsibility
- Controller thinness
- Infrastructure isolation

**Severity Levels**:

| Level | Examples |
|-------|----------|
| BLOCKING | Domain imports infrastructure, business logic in controller |
| MUST FIX | Anemic domain, primitive obsession |
| IMPROVE | Missing value objects, unclear naming |

**Report Format**:
```markdown
## Architecture Review

### BLOCKING
1. **src/Domain/User.php:15** - Imports Doctrine EntityManager
   - Impact: Domain coupled to infrastructure
   - Fix: Inject repository interface instead

### VERDICT
[ ] APPROVE
[x] REQUEST_CHANGES
[ ] BLOCK
```

---

## Symfony Pack

### symfony-reviewer

**Mission**: Review PHP/Symfony code against DDD best practices.

**Checks**:
- Final classes (except Doctrine entities)
- Private constructors with static factories

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BULDEE/ai-craftsman-superpowers](https://github.com/BULDEE/ai-craftsman-superpowers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
