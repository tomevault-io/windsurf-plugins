---
trigger: always_on
description: Persistent project-scoped store for deep research on large topics. Use for substantive questions - comparing libraries, evaluating tools, surveying solutions to hard problems. Not for plan notes, not for small facts, not for code-level decisions, not for ideas.
---


# Research

Persistent project-scoped store for deep research findings. You activated this skill because the user asked a substantive research question, or invoked it explicitly with `/research <topic>`.

If invoked with a topic argument (e.g. `/research tailwind-v5`), use it as the seed for Retrieval - start by looking up that topic in `INDEX.md`. Don't research blindly; the lookup may answer immediately.

## When to use

- "what's the latest npm package that does X"
- "compare A vs B vs C for 2026"
- "which engines / frameworks / libraries can clone X fast"
- "research how Y works under the hood"
- "deep dive on Z"
- User pastes a long markdown research dump and asks you to save it

## When NOT to use

- Plan-stage notes
- Small facts or one-line preferences
- Code-level decisions tied to one file
- Casual lookups answerable from a single source with no synthesis
- Recording personal ideas or musings
- As a substitute for a single WebSearch or WebFetch

If a single WebSearch + 1-2 sentences answers the question, you don't need this skill.

## Setup (first use only)

On first activation in a project, do this once:

1. Resolve project root: `git rev-parse --show-toplevel 2>/dev/null || pwd`
2. Create `<root>/.research/` if missing
3. Create `<root>/.research/INDEX.md` with this exact content:

   ```markdown
   # Research index

   | Topic | Path | Last verified | One-liner |
   |---|---|---|---|
   ```
4. Add `.research/` to `<root>/.gitignore`. If `.gitignore` doesn't exist, create it. Research data may contain proprietary insights, default private.

The data lives at `<root>/.research/` (sibling of `.claude/`, not nested inside it). It is a top-level project directory chosen so research data is colocated with the project, gitignored by default, and easy to find by name. Auditing remains intact: every read and write goes through the host's normal permission system.

## Workflow

### 1. Retrieval (the read side - this is how the skill saves your context)

The whole point of this system is **progressive disclosure**: don't load what you don't need. `INDEX.md` is your dispatcher - it lets you decide which entries to load *without paying to load them*. Walk the hierarchy from cheapest to most expensive; only escalate when the previous tier doesn't answer the question.

#### Loading hierarchy (cheapest → most expensive)

| Tier | Load | Approx tokens | When |
|---|---|---|---|
| 1 | `INDEX.md` (always) | ~100-500 | Every retrieval - your routing table |
| 2 | Entry's `## Summary` only | ~50-200 | When the index shows a topic match |
| 3 | Full `FINDINGS.md` body | ~500-3000 | When Summary doesn't cover the question |
| 4 | Specific `raw/<file>` document | varies (often heavy) | When a finding cites it and you need to verify a claim |
| 5 | Cross-referenced entry (`related:`) | repeats tiers 2-3 | When the question spans entries |

#### Lookup procedure

1. **Read `INDEX.md` first** (tier 1). Scan the one-liner summary column against the user's question. This is the dispatcher - same role as `RESOLVER.md` in GBrain.

2. **Match decision:**
   - **Strong match** - one entry's one-liner clearly covers the topic → go to step 3 with that entry.
   - **Multiple plausible matches** - load `## Summary` of each (still cheap at tier 2). Pick the one(s) that actually answer.
   - **Weak / no match** → fall through to Investigation. A new entry will be added.

3. **Read only the matched entry's `## Summary`** (tier 2):
   ```bash
   sed -n '/^## Summary/,/^## /p' <root>/.research/<slug>/FINDINGS.md
   ```
   Usually enough.

4. **Escalate one tier only when needed:**
   - Question needs claims-level detail beyond the Summary → load the full `FINDINGS.md` body (tier 3).
   - Question is "have we tried X before / what was discarded?" → `sed` just that section: `sed -n '/^## Discarded approaches/,/^## /p' <root>/.research/<slug>/FINDINGS.md`. Don't load the rest.
   - Question references a paste-cited claim → open that specific file under `raw/` (tier 4).
   - Question spans topics covered by separate entries → follow `related:`, repeat tiers 2-3 on each.

5. **Fall through to Investigation. Pick the mode:**
   - **No entry exists** in `INDEX.md` → Investigation in **new entry mode**.
   - **Existing entry doesn't actually resolve the question** (problem still unsolved) → Investigation in **merge mode** (pass existing entry content to the subagent).
   - **Existing entry is stale** on a fast-moving topic → Investigation in **merge mode** (refresh, don't quote).

#### What NEVER to do

- **Don't load everything.** The schema exists so you can be selective.
- **Don't load the full body when Summary suffices.** If 3 lines answer it, don't pull 300.
- **Don't load raw documents speculatively.** They're heavy; most questions don't need them.
- **Don't re-read an entry you already loaded this session** - unless it was updated since.

#### `INDEX.md` as dispatcher


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hec-ovi/research-skill](https://github.com/hec-ovi/research-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
