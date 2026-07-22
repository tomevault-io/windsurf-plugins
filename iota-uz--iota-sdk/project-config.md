---
trigger: always_on
description: Complete reference for creating and managing specialized agents in Claude Code.
---

# Claude Code Agents Reference

Complete reference for creating and managing specialized agents in Claude Code.

**Official Documentation:** https://docs.claude.com/en/docs/claude-code/sub-agents

## Agent Format

```markdown
---
allowed-tools: |
  Tool1, Tool2, Tool3
description: "When this agent should be invoked"
model: inherit  # Optional: inherit (default), sonnet, opus, or haiku
---

# Agent Name

## Role
Single, focused purpose (one responsibility only)

## When to Use
Specific triggering scenarios

## Responsibilities
- Focused task 1
- Focused task 2

## Knowledge Base
Domain-specific patterns and practices
```

### Key Features

- **Single Responsibility:** ONE clear purpose per agent
- **Context Isolation:** Separate windows prevent pollution
- **Clear Triggers:** Description defines activation conditions
- **Model Selection:** Choose optimal model for task (inherit, sonnet, opus, haiku)
- **Version Control:** Commit project agents to `.claude/agents/`

## Agent Locations

Agents can be stored in two locations:

| Location | Scope | Use Case | Committed to Git |
|----------|-------|----------|------------------|
| `.claude/agents/*.md` | Project-level | Shared team agents, project-specific workflows | ✅ Yes |
| `~/.claude/agents/*.md` | User-level | Personal agents, cross-project utilities | ❌ No |

**Best Practice:**
- **Project agents** - Domain experts (database, UI, testing) that team shares
- **User agents** - Personal productivity tools, custom workflows, learning aids

## Guidelines for Creating Agents

### Single Responsibility

- **ONE clear purpose** (non-negotiable)
- **Isolated Context:** Separate context windows prevent pollution
- **No Overlap:** Verify zero responsibility overlap with existing agents

### Description

- Write clear description: "When this agent should be invoked" in frontmatter
- Include triggering scenarios and conditions

### Domain Expertise

- Include relevant domain knowledge (e.g., IOTA SDK patterns)
- Document file-type mandates and boundaries
- Specify exact file patterns the agent handles

### Model Selection

Agents can specify which model to use:

| Model | Use Case | Cost | Recommendation |
|-------|----------|------|----------------|
| `inherit` | Use parent conversation's model | Varies | ✅ Default - recommended for most agents |
| `sonnet` | Balanced performance and cost | Medium | ✅ Best for almost all use cases |
| `opus` | Maximum capability for complex tasks | High | ⚠️ Rarely needed - only for extremely complex reasoning |
| `haiku` | Fast, lightweight tasks | Low | ⚠️ Limited capability - only for simple, well-defined tasks |

**Best Practice:** Use `inherit` (default) or explicitly set `sonnet`. Sonnet provides excellent performance for nearly all agent tasks. Only use `opus` for exceptionally complex reasoning tasks, or `haiku` for trivial operations.

### Tool Permissions

**Recommended patterns:**
- ✅ `Read(/path/to/module/**)` - Restrict to specific paths
- ✅ `Bash(go test:*)`, `Bash(go vet:*)` - Specific patterns
- ✅ Tool inheritance when appropriate

**Security risks:**
- ❌ `Bash(*)` - Never grant unrestricted access
- ❌ `Read` without path restriction - Overly permissive

## Built-in Agents

**Note:** `general-purpose` is a built-in agent type provided by Claude Code. It does not need to be defined in `.claude/agents/` and is always available for complex, multi-step tasks.

## Examples

### Example 1: Focused Module Agent

```markdown
---
allowed-tools: |
  Read(back/internal/modules/insurance/**), Bash(go test:*), Bash(go vet:*)
description: "Fix failing tests in insurance module only"
---

# Insurance Test Fixer

## Role
Fix failing unit tests in insurance module exclusively.

## When to Use
When `make test` shows insurance module test failures.

## Responsibilities
- Analyze test failures in back/internal/modules/insurance/
- Fix broken tests without changing business logic
- Ensure tests pass with `cd back && go test ./internal/modules/insurance/...`

## Knowledge Base
- Uses IOTA SDK testing patterns (itf.Setup, itf.MockDB)
- Follows table-driven test conventions
- Maintains test isolation and cleanup
```

### Example 2: Repository Agent

```markdown
---
allowed-tools: |
  Read(back/internal/**/*_repository.go), Read(back/internal/**/*_repository_test.go), Edit, Bash(go test:*)
description: "Manage repository layer: interfaces, implementations, and tests"
---

# Repository Expert

## Role
Create, update, and optimize repository layer code.

## When to Use
- Creating new repository interfaces or implementations
- Optimizing database queries
- Fixing repository-related bugs
- Writing repository tests

## Responsibilities
- Define repository interfaces in domain layer
- Implement repositories in infrastructure/persistence layer
- Write comprehensive repository tests
- Optimize query performance
- Ensure tenant isolation in queries

## Knowledge Base
- IOTA SDK repo patterns (QueryBuilder, filters)
- PostgreSQL query optimization
- Transaction handling with pgx
- Multi-tenant query patterns
```

### Example 3: UI Component Agent

```markdown
---
allowed-tools: |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iota-uz/iota-sdk](https://github.com/iota-uz/iota-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
