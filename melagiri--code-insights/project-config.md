---
trigger: always_on
description: > Agent coordination, development ceremony, and code review processes. Linked from [CLAUDE.md](../CLAUDE.md).
---

# Multi-Agent Orchestration — Code Insights

> Agent coordination, development ceremony, and code review processes. Linked from [CLAUDE.md](../CLAUDE.md).

---

## Agent Suite

| Agent | Model | Domain |
|-------|-------|--------|
| `engineer` | sonnet | Implementation across CLI, dashboard, and server — features, fixes, tests |
| `technical-architect` | opus | Architecture, type alignment, SQLite schema, code review, LLD standards |
| `ux-engineer` | opus | UX design (wireframes, flows, specs) and UI implementation (React/Tailwind/shadcn) |
| `product-manager` | sonnet | Task tracking (GitHub Issues), sprint planning, ceremony coordination |
| `journey-chronicler` | opus | Capture learning moments, breakthroughs, course corrections |
| `devtools-cofounder` | opus | DevTools strategy, DX critique, competitive positioning (on-demand) |
| `llm-expert` | opus | LLM integration review, prompt design, token optimization, model selection, cost analysis |

Agent definitions live in `.claude/agents/`.

---

## Orchestrator Role (Main Claude)

**You CAN:**
- Edit `CLAUDE.md` directly (you own it)
- Delegate implementation to the appropriate agent
- Run agents in parallel IF no dependencies exist
- Make final decisions when agents disagree

**You MUST NOT:**
- Implement code directly when an agent should do it
- Skip the ceremony steps
- Merge PRs (only the founder does this)

### Unresponsive Agent Protocol

1. **Retry once** — attempt one more communication
2. **Terminate if still unresponsive** — do not wait indefinitely
3. **Re-spawn or take over** — either spawn a fresh agent or handle the task directly
4. **Log the failure** — note which agent failed and at what step

**Do NOT** spawn duplicate agents alongside a stale one. Terminate first, then replace.

### Pre-Spawn Dependency Check (MANDATORY)

Before parallelizing agents, verify:

1. List each agent's **inputs** — What does it need?
2. List each agent's **outputs** — What does it produce?
3. Map **dependencies** — Does B need A's output?
4. Decide: **Sequential or Parallel**

**Safe to Parallelize:** Independent domains, read-only research, CLI bug fix + Dashboard UI fix (if no shared state)

**Must Run Sequentially:** TA (type decision) -> Engineer (implement types), TA (schema decision) -> Engineer (implement), any change touching `types.ts`

---

## Development Ceremony (MANDATORY)

All feature work follows this 12-step ceremony:

```
Step 1:   Founder assigns task or identifies work
Step 2:   Orchestrator identifies the right agent(s)
Step 3:   Dev agent reviews context (source files, types, existing patterns)
Step 4:   Dev agent clarifies with TA (if schema impact)
Step 5:   TA reviews approach and gives approval
Step 6:   Consensus checkpoint (TA + dev agent agree on approach)
Step 7:   Dev agent: git prechecks + create feature branch
Step 8:   Dev agent: implement, commit in logical chunks
Step 9:   Dev agent: pre-PR verification (build, test, functional check, dep audit)
Step 10:  Pre-review gates (evidence in PR description verified)
Step 11:  Triple-layer code review (loops until 0 FIX NOW items)
Step 12:  Founder merges PR
```

### Step-by-Step Ownership

| Step | Owner | Gate Criteria |
|------|-------|---------------|
| 1-2 | Orchestrator | Correct agent identified |
| 3 | Dev agent | Files reviewed, understanding confirmed |
| 4 | Dev agent -> TA | Questions resolved, no assumptions |
| 5 | TA | Explicit approval or changes requested |
| 6 | TA + Dev agent | Both confirm ready to implement |
| 7 | Dev agent | Clean repo, feature branch created |
| 8 | Dev agent | Code implemented |
| 9 | Dev agent | Build passes, tests pass, functional verification (screenshots, artifacts, curl) |
| 10 | Orchestrator | PR description has verification evidence, dep audit (if applicable) |
| 11 | TA + Outsider + LLM Expert (if applicable) | All FIX NOW items resolved (0 remaining) |
| 12 | **Founder only** | PR merged to main |

### When to Engage TA (Steps 4-5)

**Required:** Adding/modifying SQLite columns or tables, changing type definitions in `types.ts`, modifying data contract, changing config format, adding new server API endpoints

**Not required:** New command flags, parser improvements, terminal UI changes, dashboard component styling, LLM provider additions

### When to Engage LLM Expert

**Required:** Adding/modifying prompt templates (`server/src/llm/`), new LLM-powered features, changing model assignments or token budgets, SSE streaming or structured output schema changes, debugging inconsistent LLM output, cost optimization

**Not required:** CLI commands without LLM, dashboard UI (unless LLM rendering logic), source tool providers, SQLite schema (unless for LLM results storage)

**Proactive dispatch:** Auto-invoke `llm-expert` when conversation touches prompt design, token optimization, model selection, or when engineer writes new code in `server/src/llm/`.

### CI Simulation Gate (Step 8 — BLOCKING)

```bash
pnpm build    # Must pass across the workspace
```

**If ANY check fails:** Fix before creating PR. Never rely on CI.

---

## Dynamic Team Workflow

For non-trivial features, use `/start-feature` to spin up a coordinated agent team.

| Command | Purpose | When to Use |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [melagiri/code-insights](https://github.com/melagiri/code-insights) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
