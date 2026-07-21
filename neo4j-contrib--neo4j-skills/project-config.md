---
trigger: always_on
description: Use when writing Go code that connects to Neo4j. Does NOT handle Cypher — use neo4j-cypher-skill.
---

# Writing Skills for this Repository

A `SKILL.md` is an on-demand operational playbook for an autonomous coding agent — runbook with judgment, checklist with decision logic, guardrail against improvisational mistakes.

All skills live in `neo4j-*-skill/` directories. Run `python3 scripts/lint_skills.py` before every commit.

## Language Style — Write for Agents, Not Humans

Skills are read by agents that execute instructions, not humans that interpret prose. Strip every word that doesn't add information. The [Caveman compression](https://github.com/juliusbrussee/caveman) principle: remove articles, politeness, conjunctions, and explanatory padding. Keep technical content (code, commands, tables) untouched.

```
❌ You should always make sure to check the schema first before writing any queries.
✅ Check schema before writing any query.

❌ The following defaults must be applied to every query that you generate.
✅ Defaults — apply every query:

❌ Don't create a new driver per request — it is important that you create one Driver
   at startup and share it across goroutines for performance reasons.
✅ Create one Driver at startup; share across goroutines. Never create per-request.
```

Every instruction must be answerable with "done" or "not done". Terse is correct; verbose is noise that costs tokens and buries the rule.

**The 500-line budget is for content, not ceremony.** Frontmatter + When to Use/NOT + Entry + Checklist take ~40 lines. Spend the rest on actionable rules, decision tables, and code examples — not on explanatory paragraphs the agent doesn't need.

## Six Design Principles

Every skill must satisfy all six:

| Principle | Test |
|---|---|
| **Triggerable** | Does the description clearly state when to activate? |
| **Procedural** | Do the instructions define a deterministic workflow? |
| **Scoped** | Does the skill do exactly one coherent thing? |
| **Composable** | Does it work alongside sibling skills without conflict or overlap? |
| **Verifiable** | Are outputs and success criteria explicit? |
| **Context-efficient** | Is metadata concise? Does detail load only when needed? |

---

## SKILL.md Spec (agentskills.io)

Each skill is a directory with a `SKILL.md` file at the root:

```
neo4j-my-skill/
├── SKILL.md          # Required
├── references/       # Optional — detailed docs, loaded on demand
├── scripts/          # Optional — executable code
└── assets/           # Optional — templates, data files
```

### Frontmatter

```yaml
---
name: neo4j-my-skill          # Required. Must exactly match the parent directory name.
description: What it does and when to use it. Include keywords and negative triggers.
  Does NOT handle X — use neo4j-other-skill.
compatibility: Claude Code    # Optional. Max 500 chars. Only if env requirements exist.
allowed-tools: Bash WebFetch  # Optional. Space-separated pre-approved tools.
version: 1.0.0                # Optional.
---
```

### Hard rules enforced by the linter (`scripts/lint_skills.py`)

- `name` must exactly match the parent directory name — linter hard-fails on mismatch
- `name`: lowercase letters, numbers, and hyphens only; no consecutive hyphens; no leading/trailing hyphen; max 64 chars
- `description`: **80–1024 characters** — linter hard-fails outside this range
- `compatibility`: max 500 characters if present
- No unknown top-level frontmatter fields — linter rejects them
- **Never use `description: >` (YAML block scalar)**. Parsers read the raw `>` character as the description value → 1-char string → linter fail. Use inline continuation instead:

```yaml
# WRONG — block scalar, linter fails:
description: >
  Comprehensive guide to...

# RIGHT — inline with indented continuation:
description: Comprehensive guide to the Neo4j Go Driver v6 — covering driver lifecycle,
  ExecuteQuery, managed and explicit transactions, error handling, and data type mapping.
  Use when writing Go code that connects to Neo4j. Does NOT handle Cypher — use neo4j-cypher-skill.
```

---

## The Description Field — The Routing Signal

The `description` is how the agent decides which skill to load. Get it wrong and the skill never triggers, or triggers on the wrong task.

**Anatomy**: `[what it does] + [positive triggers] + [Does NOT handle X — use Y-skill]`

### Positive triggers — pack these in

- Canonical product name and version: `Neo4j Go Driver v6`, `graphdatascience v1.21`
- Common entry-point symbols: `NewDriver`, `ExecuteQuery`, `GdsSessions`, `gds.pageRank`
- Natural-language task phrases: `"Use when writing Go code that connects to Neo4j"`
- Synonyms: both `GDS` and `Graph Data Science`; both `AGA` and `Aura Graph Analytics`

### Negative triggers — always name the sibling skill

```yaml
Does NOT handle Cypher query authoring — use neo4j-cypher-skill.
Does NOT cover Aura Graph Analytics serverless sessions — use neo4j-aura-graph-analytics-skill.
```

Never a bare "Don't" without naming where to go instead. With 20+ skills in this repo, tight routing is critical. The MongoDB `mongodb-natural-language-querying` pattern is the gold standard:
> "Does NOT handle Atlas Search ($search operator) — use search-and-ai for those. Does NOT analyze or optimize queries — use mongodb-query-optimizer for that."

---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [neo4j-contrib/neo4j-skills](https://github.com/neo4j-contrib/neo4j-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
