---
trigger: always_on
description: You have access to a persistent memory system via the `omnimem` MCP server. It is the primary persistent memory store for all sessions.
---

## OmniMem — Persistent Semantic Memory

You have access to a persistent memory system via the `omnimem` MCP server. It is the primary persistent memory store for all sessions.

-----

### Tool Priority

Before using web_search or answering from training data, ALWAYS query OmniMem first:

1. Call `recall("<relevant query>")` or `recall_index("<relevant query>")` to check for prior solutions, patterns, or knowledge articles
2. Only fall back to web_search if OmniMem returns nothing useful
3. Combine both when recency matters — use OmniMem for project context and prior decisions, web for the latest information

-----

### Session Start

At the beginning of every session:

1. **Determine the project name** — use the current working directory name as the default. If uncertain, call `list_projects()` and match against known projects. If the project is genuinely ambiguous, ask the human before proceeding.
1. Call `briefing(project="<project_name>")` — this single call aggregates project context, experience summary, stale memories, new knowledge articles, contradiction warnings, and reinstate candidates.
1. If `briefing()` returns no project context but the project has episodic memories, call `compile_project_context("<project_name>", auto_save=True)` to auto-generate the context from stored memories. Review the compiled draft with the human and refine via `set_project_context()` if needed. If there are no episodic memories either (genuinely new project), call:

   ```
   set_project_context(
     name="<project_name>",
     description="<ask the human for a brief description>",
     stack=["<technologies>"],
     goals=["<current goals>"],
     current_state="<starting point>",
     domains=["<kinds of work, e.g. python, docker, design>"]
   )
   ```
1. If the project context has no `domains`, call `compile_project_domains("<project_name>")` — it proposes them from the stack and the project's own recurring tags, with the evidence for each. Show the human the draft and only save with `auto_save=True` if they agree. Domains are what let a later session search across projects rather than inside one.
1. Briefly summarise what you found:
- Current project state and goals
- Recent decisions and discovered patterns
- Abandoned approaches to avoid (graveyard)
- Stale memories that may need reviewing
- **Contradiction warnings** — surface these explicitly and ask the human which version reflects current reality before proceeding with any work
- **Skill suggestions** — if the briefing recommends compiled skills, offer them to the human; on a greenfield project (no context yet) lead with them. Load with `get_skill()` only if agreed — never auto-load
- **Skill updates** — pending changes to compiled skills. Low-risk additions can be accepted in a batch; rewrites or removals of existing rules must be reviewed individually via `compile_skill(domain, mode="propose")`
- **Knowledge watch** — if the briefing includes `skill_knowledge_watch`, recent articles look relevant to a compiled skill; entries flagged `possible_contradiction` may mean the world moved under a rule. Surface them and, if the human agrees an article belongs in the skill, call `promote_knowledge(key, domain="<domain>")` then recompile
- **Auto-proposed skills** — if the briefing includes `auto_proposed_skills`, the server's periodic scan found recurring cross-project lessons worth a new skill (or a changed skill worth a fresh draft) and has already stashed the proposal. Surface each one; review with `compile_skill(domain, mode="propose")` and accept with `mode="write"` only if the human agrees. Ignoring a draft declines it
1. If the MCP server seems unresponsive or recall is slow, call `health()` and report the status to the human before continuing.

-----

### During a Session

**Before attempting any problem** where you would normally reach for documentation or a search engine:

- Call `recall("<problem description>")` — you may find a prior solution, a relevant pattern, or a knowledge article that gives you a head start
- If a recalled knowledge article seems relevant, mention it: *“I found an article from [source] about X — shall I use that as a research base?”*
- **When the problem is about a kind of work rather than this project** — a Python gotcha, a CSS layout trap, a Docker build failure — add `domain_filter`: `recall("<problem>", domain_filter="python")` searches every project doing that kind of work. Compiled skills hold the lessons that already cleared the reinforcement gate; the domain filter reaches the raw memories underneath, including the ones that never became a rule. If the reply starts with a `domain_filter_notice` saying the filter was not applied, no project declares that domain and the results you are reading span everything — say so rather than presenting them as a targeted search

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [richarvey/OmniMem](https://github.com/richarvey/OmniMem) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
