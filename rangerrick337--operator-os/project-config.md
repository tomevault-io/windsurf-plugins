---
trigger: always_on
description: > **⚠️ CORE FRAMEWORK DOCUMENTATION**
---

# Agent Instructions

> **⚠️ CORE FRAMEWORK DOCUMENTATION**
>
> This file contains the canonical instructions for the 6-Layer Operator OS.
> It defines the architecture, operating principles, and file organization for AI agent systems.

You operate within a 6-layer architecture that separates concerns to maximize reliability. LLMs are probabilistic, whereas most business logic is deterministic and requires consistency. This system fixes that mismatch.

## The 6-Layer Architecture


**Layer 1: SOPs (What to do)**

- Written in Markdown, live in `Operator Team OS/1. SOPs/`
- Human-readable process documentation
- Define goals, context, inputs, outputs, and edge cases
- Natural language instructions, like you'd give a mid-level employee

**Layer 2: Agents (Who to be)**

- Specialized personas in `Operator Team OS/2. Agents/`
- Define voice, expertise, model preference, and decision-making style
- Activated when specific persona/expertise is needed

**Layer 2a: The Activation Pattern (How to "Be" an Agent)**
To create a switchable persona:
1.  **Define (`Operator Team OS/2. Agents/MyAgent.md`)**: The "Resume" (Voice, Model, Tools).
2.  **Trigger (`Operator Team OS/4. Workflows/be-my-agent.md`)**: The "Job Order".
    - Content: "You are now [Agent Name]. Read `Operator Team OS/2. Agents/MyAgent.md` and adopt persona."
3.  **Sync**: Run `workflow-sync` to make the slash command available.


**Layer 3: Skills (How to execute)**

- Anthropic-format skills in `Operator Team OS/3. Skills/`
- Each skill has `SKILL.md` (instructions + YAML frontmatter) + `scripts/` (code)

- **Progressive disclosure**: Read YAML frontmatter first, load full instructions only when needed
- Scripts are deterministic Python—reliable, testable, fast

**Layer 4: Workflows (Sequences)**

- Live in `Operator Team OS/4. Workflows/` (Canonical Source)
- Symlinked to `.agent/workflows` for Antigravity compatibility (allows `/` slash commands)
- Sequential "fire and forget" instructions for repetitive technical tasks
- If Antigravity is used, supports `// turbo` mode for auto-execution

**Layer 5: Knowledge Base (Where data lives)**

- `Drive - [YourProject]/` (Unstructured Data)
- **Distinction**:
    - **SOPs** tell you *how* to do it.
    - **Drive** is where you *put* the result or read the background info.
- **Example**: `Drive - ProjectX/Data/list.csv`

**Layer 6: Memory (What the system knows)**

- Tiered markdown files in `Operator Team OS/6. Memory/`
- **Long-Term** (`LONG_TERM.md`): Immutable canonical facts. Never write directly.
- **Active** (`ACTIVE.md`): Current priorities and decisions. Agent can write, date-stamp entries.
- **Logs** (`logs/YYYY-MM-DD.md`): Daily session summaries. Agent auto-generates.
- When a significant decision or fact is established, offer to commit it to memory.
- Memory skill: `Operator Team OS/3. Skills/memory-manager/SKILL.md`

**Why this works:** if you do everything yourself, errors compound. The solution is push complexity into deterministic code and clear instructions.

## Skill Discovery

When user requests a task:

1. **Scan `Operator Team OS/3. Skills/`** for matching skill folders
2. **Read YAML frontmatter** in `SKILL.md` to check relevance (description field)
3. **Load full skill** only if relevant—keeps context lean
4. **Execute scripts** in `scripts/` subfolder as directed by SKILL.md

Example skill structure:
```
Operator Team OS/3. Skills/
├── data-processing/      # Domain-specific skill
│   ├── SKILL.md          # YAML frontmatter + detailed instructions
│   └── scripts/          # Python scripts for this skill
│       ├── process_data.py
│       └── ...
```

## MCP Integration (Model Context Protocol)

MCP servers provide the raw connections to external tools (GitHub, Database, etc.). They sit *below* the Skills layer.

**1. Installation (The "Single Source")**
- **Master Config**: `.agent/mcp_config.json`
- **Sync**: This file defines which MCP servers are available to your agents.

**2. How to Add a Tool**
- Open `.agent/mcp_config.json`.
- Add the server definition under `"mcpServers"` (e.g., `github`, `postgres`).
- Your AI platform will auto-detect the new server.

**3. Access Control (The Permission Gate)**
- **Availability**: Adding it to the config makes it *technically* available.
- **Permission**: An Agent will **only** use the tool if you explicitly list it in their persona file (`Operator Team OS/2. Agents/*.md`) under the `tools:` key.
- **Protocol**: Never give an Agent "all tools". Only license them for what they need.

**4. Best Practice: Dual Listing**
Since we use both Agents and Skills, list tools in **both** places:
1.  **In `Operator Team OS/2. Agents/*.md`**: Grants the *Permission* for the agent to use it.
2.  **In `Operator Team OS/3. Skills/*/SKILL.md`**: Adds `allowed-tools: [tool_name]` to the frontmatter. This helps the Orchestrator know *what capabilities a skill requires*.

## Operating Principles

**1. Check for skills first**
Before writing a script, check `Operator Team OS/3. Skills/` for an existing skill. Only create new skills/scripts if none exist.

**2. Self-anneal when things break**

- Read error message and stack trace
- Fix the script and test it again (unless it uses paid tokens/credits—check with user first)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rangerrick337/operator-os](https://github.com/rangerrick337/operator-os) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
