---
trigger: always_on
description: Production nightly MEDDICC analysis agent. Analyzes sales calls
---

# RevOps MEDDICC Agent — Claude Code Instructions

## What this repo is

Production nightly MEDDICC analysis agent. Analyzes sales calls
for active HubSpot deals, scores them on 7 MEDDICC components,
writes scores back to HubSpot, and self-improves via a PR-based
learning loop. Built for RevOps School at Pavilion.

Scores every active deal on your qualification methodology —
MEDDICC by default; MEDDPIC, SPICED, and BANT supported via
config/client.yaml.

---

## Auto-setup for fresh forks

When this project is opened in Claude Code, check immediately:
- Does config/client.yaml exist?
- Does config/context.yaml exist?

If either is missing, this is a fresh fork. Say proactively:

"I can see this is a new deployment — config files are missing.
I'll guide you through setup. Takes about 20 minutes. Ready?"

Then run setup in this order:

### Step 1: Credentials interview

Read skills/revops-agent-setup/SKILL.md and run that interview
inline. Collect every credential one at a time. Write the
results to .env in the repo root (it's gitignored). Print the
GitHub Secrets checklist at the end.

### Step 2: Context interview

Read skills/revops-client-context/SKILL.md and run that
interview inline. Ask about product, ICP, competitors,
objections, feature gaps, HubSpot stages, and learning
preferences. Write the four output files directly:
- config/client.yaml
- config/context.yaml
- prompts/CLAUDE.md  (overwrites existing)
- prompts/evaluator_rubric.md  (overwrites existing)

Also read and use these reference files during the interview:
- skills/revops-client-context/references/context-schema.md
- skills/revops-client-context/references/claude-md-template.md
- skills/revops-client-context/references/rubric-template.md
- skills/revops-client-context/references/client-yaml-template.md

### Step 3: Stage discovery

Run: python scripts/discover_stages.py

Show the output. discover_stages.py prints a SUGGESTED pipeline:
block with HINT annotations. Walk the student through confirming,
for EACH pipeline:

- order values (HubSpot displayOrder is 0-based — keep it 0-based,
  do not renumber)
- qualified_stage_order (which order counts as 'a real opportunity'
  — drives win rate, cycle time, and the waterfall qualification filter)
- is_won / is_lost flags (a Disqualified-type stage gets BOTH
  is_lost: true AND exclude_from_analysis: true)
- exclude_from_progression: true on administrative/terminal-adjacent
  stages (prevents polluting win-rate denominator)
- analyze: false on renewal/partner pipelines (excluded from deal
  analysis, INCLUDED in analytics)
- stage_probability per open stage (used by stage-weighted forecast)

Then write the confirmed block into config/client.yaml.

### Step 4: Supabase setup

Run: python scripts/setup_supabase.py

Requires SUPABASE_DB_URL in the environment (collected in Step 1).
The script executes each migration, VERIFIES a fingerprint object
exists via a scoped read, and only then records it — if it reports
a verification failure, paste the printed SQL into the Supabase
SQL editor and re-run.

After setup, audit any time with:
  python scripts/setup_supabase.py --verify-all

### Step 5: Hand off

Tell the student:
"Add the GitHub Secrets from your .env file, then:

1. Actions → MEDDICC Agent Nightly Run → Run workflow
   (first deal analysis)
2. Actions → Weekly Analytics → Run workflow
   (first snapshot; runs itself every Sunday 3am UTC)

Note: the waterfall needs TWO snapshots before it computes
anything — its first run will correctly print 'insufficient
snapshot history' and skip. That is expected, not an error.
It comes alive on week two.

Optional: 52+ weeks of historical waterfall can be reconstructed
from HubSpot stage history — see the 'Historical Backfill' section
of docs/data-schema.md."

If config files already exist, skip setup and help with
whatever the student needs.

---

## Architecture

```
2am UTC: GitHub Actions (nightly.yml)
  → Load deals from memory/deals/index.json
  → For each deal:
      Load calls from memory/calls/<slug>.json (cache-first)
      → context_builder.py (Haiku) → cumulative MEDDICC state
      → meddicc_agent.py: Generator (Sonnet) → Evaluator (Haiku)
                          → Reflection gate (Haiku)
      → Write output/*.md
      → Write HubSpot deal properties (hubspot_deals.py)
      → Write Supabase analyses table (supabase_client.py)
      → Write memory/learnings/*.json if learning found
  → Self-improvement: learnings → synthesizer → PR to prompts/CLAUDE.md
```

---

## Key files

| File | Purpose |
|---|---|
| `scripts/run_nightly.py` | Main orchestration |
| `scripts/meddicc_agent.py` | Generator, evaluator, reflection |
| `scripts/context_builder.py` | Haiku cumulative state synthesis |
| `scripts/github_memory.py` | All file read/write operations |
| `scripts/hubspot_deals.py` | HubSpot API + MEDDICC write-back |
| `scripts/etl_deals.py` | CSV → memory/deals/index.json |
| `scripts/etl_calls.py` | CSVs → memory/calls/*.json cache |
| `scripts/supabase_client.py` | Parallel write to Supabase |
| `scripts/token_tracker.py` | Per-role cost tracking |
| `scripts/setup_supabase.py` | One-time DB migration runner |
| `scripts/discover_stages.py` | HubSpot stage ID discovery |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jeff266/revops-meddicc-agent](https://github.com/jeff266/revops-meddicc-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
