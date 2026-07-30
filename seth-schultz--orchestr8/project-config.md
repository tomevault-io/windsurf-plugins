---
trigger: always_on
description: Agents represent domain expertise and specialized knowledge. This guide covers creating effective agent fragments for the Orchestr8 dynamic expertise system.
---

# Agent Creation Guide

Agents represent domain expertise and specialized knowledge. This guide covers creating effective agent fragments for the Orchestr8 dynamic expertise system.

## Table of Contents

1. [What is an Agent?](#what-is-an-agent)
2. [Agent vs Skill vs Pattern](#agent-vs-skill-vs-pattern)
3. [Agent Fragmentation Strategy](#agent-fragmentation-strategy)
4. [Creating Core Agent Fragments](#creating-core-agent-fragments)
5. [Creating Specialized Agent Fragments](#creating-specialized-agent-fragments)
6. [Agent Composition](#agent-composition)
7. [Testing Agent Effectiveness](#testing-agent-effectiveness)
8. [Examples & Patterns](#examples--patterns)

## What is an Agent?

An **agent** represents **WHO knows** a technology, domain, or role. Agents embody expertise and provide domain-specific knowledge.

**Agents answer:**
- WHO has expertise in this technology?
- WHO can help with this domain?
- WHO understands this framework/platform?

**Examples:**
- `typescript-core` - TypeScript language expert
- `python-fastapi-dependencies` - FastAPI dependency injection specialist
- `rust-expert-advanced` - Advanced Rust programming expert
- `devops-expert-cicd` - CI/CD pipeline specialist
- `database-architect-sql` - SQL database design expert

**Key characteristics:**
- **Domain-focused:** Represents expertise in specific technology/domain
- **Composable:** Core + specialized fragments combine for complete expertise
- **Role-based:** Often maps to job roles (Developer, Architect, Engineer)
- **Technology-specific:** Tied to particular frameworks, languages, or platforms

## Agent vs Skill vs Pattern

Understanding the distinction is critical for proper categorization.

### Agent: WHO (Domain Expertise)

**Create an agent for:**
- Technology expertise: TypeScript Developer, Rust Expert, Python Engineer
- Platform knowledge: AWS Architect, GCP Specialist, Kubernetes Expert
- Domain specialization: Frontend Developer, Backend Developer, Data Engineer
- Role-based expertise: DevOps Engineer, Security Specialist, QA Engineer

**Examples:**
```yaml
# Agent fragments
typescript-core
typescript-api-development
python-fastapi-validation
rust-expert-core
devops-expert-cicd
frontend-react-expert
database-architect-sql
```

### Skill: HOW (Technique)

**Create a skill for:**
- Reusable techniques applicable across technologies
- Methodologies and approaches
- Cross-cutting concerns
- Implementation patterns

**Examples:**
```yaml
# Skill fragments (NOT agents)
error-handling-resilience
testing-integration-patterns
security-authentication-jwt
api-design-rest
performance-optimization
git-workflow
```

### Pattern: WHY/WHEN (Architecture)

**Create a pattern for:**
- Architectural approaches
- Design patterns
- System design strategies
- Structural decisions

**Examples:**
```yaml
# Pattern fragments (NOT agents)
architecture-microservices
event-driven-cqrs
database-indexing-strategies
performance-caching
autonomous-organization
```

### Decision Tree

```
Is this domain expertise tied to a specific technology?
├─ YES → Agent
│  └─ Examples: typescript-core, python-fastapi
└─ NO → Is this an architectural approach?
   ├─ YES → Pattern
   │  └─ Examples: microservices, event-driven
   └─ NO → Skill
      └─ Examples: error-handling, testing-strategies
```

## Agent Fragmentation Strategy

### The Monolithic Problem

**❌ Don't create monolithic agents:**

```markdown
python-expert.md (3000 tokens)
├─ Python fundamentals
├─ FastAPI development
├─ Async programming
├─ Data science
├─ Testing
├─ Deployment
└─ Database integration

Problems:
- Always loads 3000 tokens
- User may only need FastAPI (500 tokens)
- 83% token waste
- Slow to load
- Hard to maintain
```

### The Fragmented Solution

**✅ Create focused agent fragments:**

```markdown
Core Fragment (always-relevant):
python-core.md (650 tokens)
├─ Language fundamentals
├─ Type system (type hints)
├─ Common patterns
├─ Standard library
└─ Package management

Specialized Fragments (load as needed):
python-fastapi-dependencies.md (500 tokens)
python-fastapi-validation.md (480 tokens)
python-async-fundamentals.md (450 tokens)
python-async-concurrency.md (520 tokens)

Benefits:
- Load only what's needed
- Generic query: python-core (650 tokens)
- Specific query: python-core + python-fastapi-dependencies (1150 tokens)
- 62% token savings on specific queries
- 78% savings on generic queries
```

### Fragmentation Guidelines

**When to fragment:**
1. **Domain has multiple specializations**
   - TypeScript: core, api, async, testing
   - Python: core, fastapi, async, data-science

2. **Knowledge exceeds 1000 tokens**
   - Split into core (600-750) + specializations (450-650 each)

3. **Users need different subsets**
   - Some need API development
   - Others need async programming
   - Few need both

**How to fragment:**

```markdown
Step 1: Identify core knowledge (always relevant)
└─ Language fundamentals, syntax, common patterns

Step 2: Identify specializations (sometimes relevant)
└─ API development, async, testing, data science

Step 3: Create fragments
├─ Core: 600-750 tokens (always loaded)
└─ Specializations: 450-650 tokens each (load as needed)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [seth-schultz/orchestr8](https://github.com/seth-schultz/orchestr8) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
