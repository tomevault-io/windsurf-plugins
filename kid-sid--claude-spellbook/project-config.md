---
trigger: always_on
description: At the start of every session, before doing anything else:
---

## Session Setup (Required)
At the start of every session, before doing anything else:
1. Call `load_memory` with `C:\Users\Sidhartha\claude-spellbook`
2. Call `load_history` with `C:\Users\Sidhartha\claude-spellbook`
3. Read both outputs — they contain saved context and conversation history
4. Do not explore files or ask clarifying questions that memory already answers

Save or update memory entries whenever you learn something worth keeping across sessions.
If something loaded from memory is no longer accurate, update it with `save_memory` using the same key.
Use short, lowercase keys: `stack`, `current_work`, `gotchas`, `key_files`, etc. Keep values concise — one or two sentences max.

---

## MCP Server (memory_map)
Persistent memory and conversation history is provided by the standalone [memory_map](https://github.com/kid-sid/memory_map) repo — not bundled in this repo.

**Tools available:**
- `load_memory` / `save_memory` / `delete_memory` — per-project key-value context store
- `load_history` / `save_history` — rolling conversation history (20 chunks)
- `set_compression` — set output compression level (0=raw, 1=compact, 2=dense)
- `get_local_structure` — local directory tree (gitignore-aware)
- `get_github_structure` — GitHub repo file tree
- `get_git_history` — recent commits

**One-time setup:**

```bash
# 1. Clone and install
git clone https://github.com/kid-sid/memory_map.git
cd memory_map
python -m venv venv
venv\Scripts\pip install -r requirements.txt   # Windows
# source venv/bin/activate && pip install -r requirements.txt  # Mac/Linux

# 2. Register globally (available in ALL projects)
claude mcp add -s user memory_map C:/Users/yourname/memory_map/venv/Scripts/python.exe C:/Users/yourname/memory_map/server.py
# Mac/Linux: claude mcp add -s user memory_map python3 /home/yourname/memory_map/server.py
```

**3. Add hooks to `~/.claude/settings.json`** so history saves automatically in every project:

```json
{
  "hooks": {
    "UserPromptSubmit": [{ "matcher": "", "hooks": [{ "type": "command", "command": "python C:/Users/yourname/memory_map/history_hook.py", "timeout": 10 }] }],
    "PreCompact":       [{ "matcher": "", "hooks": [{ "type": "command", "command": "python C:/Users/yourname/memory_map/history_hook.py --force", "timeout": 15 }] }],
    "Stop":             [{ "matcher": "", "hooks": [{ "type": "command", "command": "python C:/Users/yourname/memory_map/history_hook.py --force", "timeout": 15, "async": true }] }]
  }
}
```

**4. Enable memory for a project** — copy `CLAUDE.md` from the memory_map repo into the project root. Claude reads it at session start and calls `load_memory` / `load_history` automatically.

**Manual history save:** use `/mem_save` at any time to checkpoint the current conversation.

---

# Claude Spellbook

This is a curated library of Claude Code skills, agents, and slash commands covering the full software engineering lifecycle.

## Repository Layout

```
skills/<skill-name>/skill.md     — skill reference files (install to ~/.claude/skills/)
.claude/agents/<name>.md         — autonomous subagents (install to ~/.claude/agents/)
.claude/commands/<name>.md       — slash commands (install to ~/.claude/commands/)
```

## Skill Inventory

### Requirements & Design
- `requirements-planning` — user stories, PRDs, acceptance criteria, story pointing
- `system-design` — HLD/LLD, capacity estimation, ADRs, scalability/reliability patterns
- `api-design` — REST API conventions, pagination, error responses, versioning
- `database-design` — schema design, indexing, migrations, query optimization
- `microservices` — service decomposition, sync/async communication, circuit breaker, saga, CQRS, API gateway

### Development
- `coding-standards` — naming conventions, SOLID, design patterns, code smells
- `development-workflow` — branching, conventional commits, PR workflow, code review
- `frontend` — React component design, state management (Zustand/Redux Toolkit), TanStack Query, React Hook Form, routing, performance, testing
- `event-driven` — Kafka producer/consumer, topic partitioning, outbox pattern, dead-letter queues, idempotency, event sourcing
- `caching` — Redis patterns, cache-aside/write-through/write-behind, TTL design, stampede prevention, HTTP Cache-Control, invalidation
- `claude-code` — skills/commands/agents setup, hooks (PreToolUse/PostToolUse/SessionStart/Stop), settings.json permissions, MCP servers, CLAUDE.md

### Testing
- `unit-testing` — AAA pattern, mocking, parameterized tests, TDD, coverage
- `integration-testing` — Testcontainers, HTTP API tests, contract testing, test data
- `solution-testing` — Playwright E2E, BDD/Gherkin, smoke tests, flakiness prevention
- `test-strategy` — Pyramid/Trophy/Honeycomb models, coverage targets, test plans
- `performance-testing` — k6, Locust, Go benchmarks, SLO-based pass/fail

### Security & Quality
- `security` — OWASP Top 10, JWT/OAuth2, secrets management, STRIDE, dependency scanning
- `accessibility` — WCAG 2.1/2.2 conformance, ARIA, keyboard navigation, focus management, contrast, screen readers
- `claude-api` — Anthropic SDK patterns, tool use, streaming, agent SDK
- `azure` — DefaultAzureCredential, Blob Storage, AI Search (vector/hybrid), Document Intelligence, Key Vault, retry patterns

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kid-sid/claude-spellbook](https://github.com/kid-sid/claude-spellbook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
