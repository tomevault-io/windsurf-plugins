---
trigger: always_on
description: > Single source of truth for all IDE configurations (Claude Code, Cursor, Windsurf, Codex)
---

# MCP Gateway - Project Configuration

> Single source of truth for all IDE configurations (Claude Code, Cursor, Windsurf, Codex)

**Project Path**: (set to your local clone path)

---

## FIRST: Check Skill Triggers (LAZY LOADING)

**Skill Loading Strategy** (Token-Optimized):
- For **SIMPLE** tasks: Use your built-in knowledge, don't load skills
- For **COMPLEX** tasks: Load the appropriate skill file only when needed

**Simple Tasks** (Don't load skills):
- Single-line fixes, typos, obvious bugs
- Basic questions about concepts
- Simple git commands (status, log)
- File reads, basic edits

**Complex Tasks** (Load skill first):
- Multi-file refactoring
- Architecture design decisions
- Security reviews or audits
- Advanced git workflows (interactive rebase, conflict resolution)
- Production deployments
- Database schema migrations
- Performance optimization

**If loading a skill:**
1. Read the skill: `.agents/skills/{skill-name}/SKILL.md`
2. Follow the skill's instructions in your response

**This applies to ALL IDEs** - Claude Code, Cursor, Windsurf, VS Code, Codex, etc.

---

## What is MCP Gateway?

**MCP Gateway** is a universal Model Context Protocol (MCP) aggregation server that:
- Combines 305+ tools from multiple backend MCP servers into a single endpoint
- Provides 15 layers of token optimization (95-98% reduction)
- Works with Claude Desktop, Cursor, VS Code Copilot, OpenAI Codex
- Includes a web dashboard for tool/backend management

---

## Cipher Memory Protocol (MANDATORY)

**CRITICAL RULE**: If the user asks ANY memory-related question (e.g., "What have we worked on today?", "Recall context"), **NEVER check git log, git status, or conversation history summaries**. ALWAYS use the `cipher_ask_cipher` tool.

### Tool Schema

```typescript
cipher_ask_cipher({
  message: string,      // Required: What to store or ask
  projectPath: string   // MANDATORY: Full project path
})
```

### projectPath Rules

1. **ALWAYS** use full absolute path: `/path/to/mcp-gateway`
2. **NEVER** use placeholders like `{cwd}` - they don't resolve!
3. **NEVER** use just the project name

### Quick Reference

| Action | Message Format |
|--------|----------------|
| **Recall context** | `"Recall context for this project. What do you remember?"` |
| **Store decision** | `"STORE DECISION: [description]. Reasoning: [why]"` |
| **Store bug fix** | `"STORE LEARNING: Fixed [bug]. Root cause: [cause]. Solution: [fix]"` |
| **Store milestone** | `"STORE MILESTONE: Completed [feature]. Key files: [files]"` |
| **Store pattern** | `"STORE PATTERN: [pattern_name]. Usage: [when_to_use]"` |
| **Store blocker** | `"STORE BLOCKER: [description]. Attempted: [what_tried]"` |
| **Search memory** | `"Search memory for: [topic]. What patterns or learnings are relevant?"` |
| **Session end** | `"Consolidate session. Accomplishments: [list]. Open: [items]"` |

---

## Skill Auto-Activation

Skills auto-activate via hook in Claude Code. For other IDEs, see `.agents/AGENTS.md`.

Skills location: `.agents/skills/{skill-name}/SKILL.md`

### AI Delegation Workflow (MANDATORY)

**CRITICAL**: Use a team-of-agents pass for direction, then apply one orchestrator-owned minimal-risk patch.

#### Orchestrator Rule

- Claude is the orchestrator and owns architecture, coding, and final edits.
- Specialist agents provide support outputs only (research/analysis/summarization/translation/file ops).

#### Routing Rules

| Task Type | Primary | Fallback | Why |
|-----------|---------|----------|-----|
| **Research** | Kimi | Z.AI | Fast exploration, multilingual |
| **Analysis** | Z.AI | Kimi | Reasoning cross-check |
| **Creative** | Kimi | Z.AI | Ideation quality |
| **Translation** | Kimi | Z.AI | Multilingual quality |
| **Summarization** | Minimax | Kimi | Long context + cost |
| **Long-context** | Minimax | Kimi | 1M-context processing |
| **Fast/Cheap** | Minimax | Z.AI | Latency and cost control |
| **File editing** | Codex CLI | Kimi | Local workspace operations |
| **General** | Kimi | Minimax | Balanced default |

#### Correct CLI Usage

Gemini does **not** support `-w`; use `--include-directories`:

```bash
gemini --include-directories "/path/to/mcp-gateway" -p "Plan implementation for <task>" --yolo
```

Kimi workspace form (requires valid CLI login/auth):

```bash
kimi -w "/path/to/mcp-gateway" -p "Research/analyze <task>" --yolo
```

#### Workflow Pattern

1. Delegate to one or two specialists for direction.
2. Compare outputs and resolve conflicts.
3. Apply one final patch in the main codebase.
4. Validate with tests/build.

#### Security: NEVER Share Credentials

**CRITICAL**: Only Abdullah's primary orchestrator is trusted with credentials.

- **NEVER** expose passwords, API keys, or secrets to delegated agents
- **NEVER** delegate raw secret-bearing files
- Sanitize/redact sensitive data before delegation

---

### Loading Skills

**Action Required**: When triggers match, read the skill file BEFORE responding:

```
.agents/skills/{skill-name}/SKILL.md
```

Example: User says "review this code" → Read `.agents/skills/code-review/SKILL.md` → Follow its instructions.

**Claude Code**: Auto-loads via hook (backup mechanism).
**Other IDEs**: Must follow this instruction manually.

---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [abdullah1854/MCPGateway](https://github.com/abdullah1854/MCPGateway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
