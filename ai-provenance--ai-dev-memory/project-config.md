---
trigger: always_on
description: DevMemory agent coordination — shared persistent memory across sessions and agents
---


# DevMemory: Shared Agent Memory

You have access to a shared project memory via the `agent-memory` MCP server.
This memory persists across all sessions and is shared between every agent working on this project.
Use it as a **knowledgebase** (look up past decisions) and **coordination tool** (leave context for future sessions).

The project also maintains **knowledge files** in `.devmemory/knowledge/*.md` — these are the canonical source of architecture decisions, conventions, and gotchas. You are responsible for keeping them up to date.

## 1. Before Starting Any Task

**Always search memory first.** Before writing code, look up what's already known:

```
search_long_term_memory(text="<describe what you're about to work on>", namespace="default:git-github-com-AI-Provenance-ai-dev-memory-git")
```

Search for a specific topic:
```
search_long_term_memory(text="...", topics=["<topic>"], namespace="default:git-github-com-AI-Provenance-ai-dev-memory-git")
```

Search for:
- Past decisions related to your task
- Known issues or gotchas in the area you're touching
- Established patterns and conventions
- Previous attempts that failed and why

If your task involves a specific file or module, also search for it:
```
search_long_term_memory(text="<module or file name> issues and patterns", namespace="default:git-github-com-AI-Provenance-ai-dev-memory-git")
```

Also **read the relevant knowledge files** before starting:
- `.devmemory/knowledge/architecture.md` — architecture decisions and design rationale
- `.devmemory/knowledge/gotchas.md` — known issues, workarounds, and pitfalls
- Any other `.md` files in `.devmemory/knowledge/` relevant to your task

Incorporate any relevant context into your approach. If a previous session already tried and rejected an approach, don't repeat it.

## 2. After Making Significant Decisions

You have two ways to persist knowledge, use the right one:

### Quick capture: `devmemory add` (CLI)

For a single discovery or decision during a session, run:
```bash
devmemory add "<what you learned>" --topic <topic> --entity <entity>
```

Use this for:
- A gotcha you just hit and solved
- An API quirk you discovered mid-task
- A quick decision that doesn't need a full write-up

### Structured knowledge: update `.devmemory/knowledge/` files

For anything that future agents should know about, **update the knowledge files directly** and then sync:

**When to update knowledge files:**
- You made an architecture decision (add to `architecture.md`)
- You discovered a gotcha or workaround (add to `gotchas.md`)
- You established a new convention or pattern (add to `conventions.md` — create if needed)
- You added/changed a major dependency and why
- You fixed a non-obvious bug that could regress

**How to update:**
1. Edit the appropriate `.devmemory/knowledge/*.md` file (or create a new one)
2. Add a new `## Section Heading` with the content
3. Run `devmemory learn` to sync the updated files into the memory store

**Format for knowledge files:**
```markdown
---
topics: [architecture, decisions]
entities: [Redis, AMS]
---

## Section Title
Content explaining what, why, and any relevant details.
Each ## section becomes a separate searchable memory.
```

**If no existing file fits, create a new one.** Good filenames:
- `architecture.md` — why we chose X over Y, system design
- `gotchas.md` — things that break if you're not careful
- `conventions.md` — coding patterns and project rules
- `api-notes.md` — external API quirks and limitations
- `dependencies.md` — why we use specific libraries

### Also store via MCP for immediate availability

After updating knowledge files, also store via MCP so the memory is searchable immediately (before the next `devmemory learn` run):

```
create_long_term_memories(memories=[{
    "text": "<what was decided and why>",
    "memory_type": "semantic",
    "topics": ["<relevant>", "<topics>"],
    "entities": ["<technologies>", "<modules>"],
    "namespace": "default:git-github-com-AI-Provenance-ai-dev-memory-git"
}])
```

### What to store as memories

| Store | Don't store |
|-------|-------------|
| Architecture decisions with rationale | Implementation details obvious from code |
| Known gotchas and workarounds | Temporary debugging notes |
| Bug root causes and how they were fixed | Things that change every commit |
| Project conventions and patterns | Redundant copies of commit messages |
| API quirks and limitations discovered | Personal preferences |
| Why approach A was chosen over B | Simple variable renames or formatting |

## 3. Keeping Knowledge Files Fresh

**This is a core responsibility.** Treat `.devmemory/knowledge/` files like living documentation.

### During every session, ask yourself:

1. **Did I discover something non-obvious?** → Add to `gotchas.md`
2. **Did I make a design choice between alternatives?** → Add to `architecture.md`
3. **Did I establish or follow a pattern?** → Add to `conventions.md` (create if missing)
4. **Is any existing knowledge file outdated?** → Update it

### After updating knowledge files:

Always run the sync to push changes into the memory store:
```bash
devmemory learn
```

### Periodic review


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AI-Provenance/ai-dev-memory](https://github.com/AI-Provenance/ai-dev-memory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
