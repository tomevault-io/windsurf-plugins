---
trigger: always_on
description: Explore, analyze, and synthesize agentic coding frameworks. The goal is to:
---

# Agentic Coding Framework Research

## Project Purpose
Explore, analyze, and synthesize agentic coding frameworks. The goal is to:
1. **Discover** — Find and catalog agentic coding frameworks, patterns, and tools
2. **Analyze** — Deep-dive into each framework's architecture, strengths, and weaknesses
3. **Synthesize** — Extract the best ideas and combine them into novel approaches
4. **Build** — Create our own improved framework leveraging the best discoveries

## Project Structure
```
research/
  frameworks/    # One markdown per framework analyzed
  analysis/      # Comparative analyses and deep-dives
  synthesis/     # Combined insights and new ideas
docs/            # Project documentation
hooks/           # Custom hook scripts
tools/           # Utility scripts
.claude/
  commands/      # Custom slash commands
```

## Research Methodology
When analyzing a framework:
1. Document its core architecture and design philosophy
2. Identify unique patterns (hooks, skills, MCPs, agents, etc.)
3. Rate strengths/weaknesses on: extensibility, composability, developer UX, reliability
4. Extract reusable patterns with concrete code examples
5. Note what's novel vs. borrowed from other frameworks

## Framework Analysis Template
Each framework analysis in `research/frameworks/` should follow:
```markdown
# [Framework Name]
## Overview & Philosophy
## Architecture
## Key Patterns
## Strengths
## Weaknesses
## Unique Ideas Worth Extracting
## Code Examples
```

## Conventions
- All research notes in Markdown
- Use descriptive filenames: `research/frameworks/claude-code.md` not `fw1.md`
- Tag insights with categories: `[hook]`, `[skill]`, `[mcp]`, `[agent]`, `[pattern]`
- When discovering a new pattern, immediately document it before moving on
- Cross-reference between documents using relative links

## Key Concepts We're Tracking
- **Hooks** — Pre/post execution interceptors, validation gates, behavioral guardrails
- **Skills** — Reusable, composable capabilities that can be invoked on demand
- **MCPs** — Model Context Protocol servers for tool/resource integration
- **Agent Orchestration** — Multi-agent coordination patterns (swarms, hierarchies, meshes)
- **Memory Systems** — Persistent context across sessions and agents
- **Code Generation** — How frameworks approach generating vs. templating code
- **Sandboxing** — Isolation and safety mechanisms for agent execution
- **Human-in-the-loop** — Approval workflows and intervention patterns

## Working Guidelines
- Prefer depth over breadth — a thorough analysis of 5 frameworks beats a surface scan of 20
- Always verify claims by reading actual source code when available
- Document surprises and anti-patterns, not just best practices
- When in doubt about a framework's behavior, test it rather than assume

## Lazy Fetch (CLI Companion)

This project uses [lazy-fetch](https://github.com/Clemens865/Lazy-Fetch) for context, persistence, and process tracking.

### Available Commands

**Start every session with:** `lazy read` — loads git state, plan progress, and stored memory.

| Command | When to use |
|---------|------------|
| `lazy plan <goal>` | Break a goal into phased tasks |
| `lazy plan --file <file>` | Import tasks from a bullet-point markdown file |
| `lazy status` | Check current plan progress |
| `lazy done <task or #>` | Mark a task complete (supports name or index number) |
| `lazy add <task>` | Add a task to the current plan |
| `lazy gather <task>` | Find relevant files before starting a task |
| `lazy check` | Validate: typecheck, tests, lint |
| `lazy remember <key> <value>` | Store a decision or fact for future sessions |
| `lazy recall [key]` | Retrieve stored knowledge |
| `lazy journal <entry>` | Log a decision or milestone |
| `lazy yolo <prd-file>` | Autonomous mode: parse PRD into sprints, execute end-to-end |

### Blueprints — Use These for Common Tasks

Blueprints are pre-built workflows that handle the full cycle: gather context, checkpoint, implement, validate, and remember. **Prefer blueprints over ad-hoc implementation** when the user's intent matches one.

| Blueprint | When to trigger | Command |
|-----------|----------------|---------|
| **fix-bug** | User reports a bug, error, crash, or unexpected behavior. Keywords: "bug", "broken", "error", "fix", "crash", "doesn't work", "500", "fails" | `lazy bp run fix-bug "<description>"` |
| **add-feature** | User wants new functionality added. Keywords: "add", "implement", "build", "create", "new feature", "support for" | `lazy bp run add-feature "<description>"` |
| **experiment** | User wants to try something without committing. Keywords: "try", "experiment", "what if", "explore", "prototype", "test an idea", "spike" | `lazy bp run experiment "<description>"` |
| **review-code** | User wants code reviewed. Keywords: "review", "check my code", "audit", "look over", "any issues", "code quality" | `lazy bp run review-code "<description>"` |

**How blueprints work:**
- Deterministic steps (gather, typecheck, tests, git checkpoint) run automatically
- Agentic steps (analyze, implement, document) return prompts — follow them in order
- Validation gates retry on failure (typecheck + tests must pass)
- Results are persisted to memory via `lazy remember`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Clemens865/Lazy-Fetch](https://github.com/Clemens865/Lazy-Fetch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
