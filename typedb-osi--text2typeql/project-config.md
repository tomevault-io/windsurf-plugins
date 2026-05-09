---
trigger: always_on
description: When starting a **main interactive session** (NOT when running as a subagent), read these files:
---

# Text2TypeQL - Project Notes

## Important: First Steps (Main Session Only)

When starting a **main interactive session** (NOT when running as a subagent), read these files:
- `README.md` - Dataset card and project overview
- `pipeline/README.md` - Pipeline docs, progress, and remaining tasks
- `plan.md` - Implementation plan (if it exists)
- `dataset/<source>/<database>/README.md` - Query counts for each database being worked on

**Subagents** (e.g., `convert-query-runner`) should NOT read these files — they have self-contained instructions.

## Updating Documentation

When learning new TypeQL patterns or syntax rules, update ALL of these files:
- `CLAUDE.md` - This file (quick reference)
- `.claude/skills/convert-query.md` - Conversion skill (comprehensive reference)
- `pipeline/docs/suggestions.md` - Validated examples of advanced patterns

## Project Overview

Pipeline to convert Neo4j text2cypher datasets to TypeQL format for training text-to-TypeQL models.

**Source**: https://github.com/neo4j-labs/text2cypher

Two source datasets:
- **synthetic-1**: `datasets/synthetic_opus_demodbs/` -- 7 databases, 4,776 valid queries (4,733 converted, 43 failed)
- **synthetic-2**: `datasets/synthetic_gpt4o_demodbs/` -- 15 databases, 9,267 valid queries (9,206 converted, 61 failed)

## Important: Sequential Processing

**DO NOT process queries in parallel.** Multiple agents writing to the same CSV file can cause race conditions and data loss. Always process queries one at a time, waiting for each to complete before starting the next.

## Available Skills

Use this skill for query conversion (requires TypeDB server running):

- `/convert-query <database> <index>` - Convert from synthetic-1 (default)
- `/convert-query <database> <index> --source synthetic-2` - Convert from synthetic-2

## Quick Start

```bash
# Start TypeDB server (must be running for validation)
typedb server --development-mode.enabled=true

# Run pipeline
python pipeline/main.py setup                      # Clone Neo4j dataset
python pipeline/main.py list-schemas               # List schemas (synthetic-1)
python pipeline/main.py list-schemas --source synthetic-2  # List schemas (synthetic-2)
python pipeline/main.py convert-schema movies      # Convert schema

# Use skill for query conversion (agent-based, no API costs)
/convert-query movies 0
/convert-query movies 0 --source synthetic-2
```

## Agent-Based Query Conversion (Main Session Only)

**NOTE: This section is for the MAIN interactive session only. If you are a subagent (e.g., `convert-query-runner`), IGNORE this entire section — do the conversion work directly using your own instructions.**

In the main session, always delegate TypeQL query conversion to the `convert-query-runner` subagent. Do NOT write TypeQL queries directly in the main conversation — use the Task tool to spawn the subagent. This ensures consistent validation, semantic review, and CSV routing.

**IMPORTANT: Process queries SEQUENTIALLY, not in parallel.** Parallel writes to the same CSV file can cause race conditions.

Set `max_turns: 15` to prevent runaway execution:
```
Use Task tool with subagent_type=convert-query-runner, max_turns=15
Prompt: "Convert query <index> from the <database> database"
```

For synthetic-2:
```
Prompt: "Convert query <index> from the <database> database --source synthetic-2"
```

When providing hints or specific patterns to use, include them in the subagent prompt:
```
Prompt: "Convert query <index> from <database>. Hint: use reduce with max() groupby for the aggregation."
```

For re-converting queries from `failed_review.csv`:
```
Prompt: "Convert query <index> from <database> --source failed_review"
```

## TypeDB 3.0 Query Syntax

### Key Rules

1. **Query order**: `match` → `sort` → `limit` → `fetch` (or `reduce`)
2. **Relations**: `relation_type (role: $var, role: $var);` (NOT `(role: $var) isa type`)
3. **Fetch directly**: `fetch { "prop": $entity.prop };` - no need to bind
4. **Double quotes** for strings
5. **Grouped counts**: `reduce $count = count($var) groupby $group_var;`

### Pattern Examples

```typeql
# Entity with attribute
$p isa person, has name "John";

# Relation (TypeDB 3.0 style - preferred)
follows (follower: $a, followed: $b);

# Relation with variable (when accessing relation attributes)
$rel isa follows (follower: $a, followed: $b);
$rel has timestamp $t;

# Fetch directly from entity (preferred)
fetch { "name": $p.name, "age": $p.age };

# Bind only when filtering/sorting
$p has age $a; $a > 25;
sort $a desc;
fetch { "age": $a };

# Multi-cardinality attributes
fetch { "emails": [ $p.email ] };

# Negation
not { follows (follower: $p, followed: $other); };

# Grouped aggregation
match $p isa person; acted_in (actor: $p, film: $m);
reduce $count = count($m) groupby $p;
sort $count desc;
limit 5;
```

### Advanced Features

```typeql
# String length (TypeDB 3.8+)
let $len = len($name);

# String concatenation (TypeDB 3.8+)
let $display = $name + " (" + $city + ")";

# Custom functions - reusable query logic
with fun follower_count($user: user) -> integer:
  match follows (followed: $user);
  return count;
match $u isa user;
let $count = follower_count($u);
fetch { "user": $u.name, "followers": $count };


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [typedb-osi/text2typeql](https://github.com/typedb-osi/text2typeql) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
