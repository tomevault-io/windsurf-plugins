---
trigger: always_on
description: All GTM pipeline skills must follow these conventions. Read this file before executing any skill.
---

# GTM Pipeline — Shared Conventions

All GTM pipeline skills must follow these conventions. Read this file before executing any skill.

---

## Working Directory

Every skill run operates inside a `{client-slug}-gtm/` directory. Create it on the first skill invocation for a client. If it already exists, reuse it.

```
{client-slug}-gtm/
├── context/
│   ├── icp.md                         ← ICP definition, voice, proof points
│   └── provider_performance.md        ← Running log of provider results per run
├── prompts/
│   └── message_prompt.md              ← Message generation prompt (demo skill)
├── csv/
│   ├── input/
│   │   ├── companies_raw.csv          ← Input company list
│   │   └── contacts_raw.csv           ← Input contact list (if provided)
│   ├── intermediate/
│   │   ├── companies_enriched.csv     ← After company-enrichment
│   │   ├── companies_scored.csv       ← After ICP scoring
│   │   ├── signals.csv                ← After signal-search
│   │   ├── contacts_found.csv         ← After people-search
│   │   ├── contacts_filtered.csv      ← After contact-filter (ICP-ranked, enrichment-ready)
│   │   └── request_ids.json           ← Saved API task IDs for recovery
│   └── output/
│       ├── contacts_enriched.csv      ← Final enriched contacts
│       └── messages.csv               ← Generated messages (demo only)
└── run_log.md                         ← Chronological log of all skill runs with costs
```

**Client slug:** lowercase, hyphens, no spaces (e.g. `acme-corp-gtm/`, `example-client-gtm/`).

---

## Output Format

Every skill run must end with this summary printed to the user AND appended to `run_log.md`:

```
## Run Summary — {skill name} — {timestamp}
- **Records processed:** {n}
- **Records with results:** {n_found} ({hit_rate}%)
- **Provider used:** {provider_name}
- **Credits consumed:** {credits} (est. {cost_per_record} × {n})
- **Cost:** ~${cost} (if pricing known)
- **Quality assessment:**
  - Hit rate: {hit_rate}%
  - Data completeness: {completeness details}
  - Issues: {issues or "None"}
- **Recommendation:** {proactive suggestion or "None"}

## Not Yet Available
- {list of undocumented tools/endpoints that could improve this run}
- {reference to the skill's "What's Missing" items}
```

**Rules:**
- Produce this summary after every step (test batch, full batch, fallback pass), not just at the end
- Include actual data quality observations, not just numbers — e.g. "2 contacts from wrong country (flagged)"
- The "Not Yet Available" section references items from each skill's own "What's Missing" list

---

## Run Log

Append to `{client-slug}-gtm/run_log.md` after each skill step:

```markdown
### {timestamp} — {skill name} — {step name}
- Provider: {provider}
- Records: {n_processed} → {n_found} ({hit_rate}%)
- Credits: {credits_consumed}
- Cost: ~${cost}
- Duration: {time}
- Notes: {any issues or observations}
```

---

## Provider Performance Log

Append to `{client-slug}-gtm/context/provider_performance.md` after each provider run:

```markdown
### {date} — {provider_name} — {skill}
- Audience: {description of target audience/geography}
- Hit rate: {actual}% (baseline: {expected from guide}%)
- Deviation: {actual - expected}%
- Records: {n}
- Cost: {credits} credits
- Issues: {any}
```

---

## Provider Optimization

After each batch:
1. Compare actual hit rate vs documented baseline (from the skill's provider tables)
2. If deviation > 15 percentage points below baseline → flag and suggest switching providers
3. If multiple providers are available, suggest A/B test on next batch
4. After a full run, produce a provider comparison table if multiple providers were used or could have been used

---

## Demo Mode

When a skill is invoked from the demo flow:
- **No phone enrichment** — email only, skip all phone providers
- **Scope:** ~10 contacts (request 10–15, expect drop-off)
- The demo skill sets this context explicitly when invoking other skills
- People-enrichment must check for this and skip phone providers entirely

---

## Cross-Cutting Rules

These rules apply to every skill in the pipeline:

1. **Plan and test in sandboxes** — validate request/response structure at zero cost before any production call
2. **Test a few leads before the full batch** — 5–15 records, never the full list. If the primary source returns 0 on the probe, switch source before spending the full batch (see People-Source Cadence).
3. **Review results after each run** — inspect actual data rows (names, titles, locations, URLs), not just hit counts
4. **LLM/judgement work is done by the agent** — extraction, scoring, filtering, and message generation run in-context (or via model-routed subagents), never via a third-party LLM API on the default path. See **Model Routing**. Do not silently swap models.
5. **Never re-run before reviewing** — do not waste credits on duplicate runs
6. **Save all API output fields in `intermediate/`** — never drop columns from API responses there. The **lead-facing `output/` view is sanitized** (see Output Sanitization): provenance, statuses, and empty columns are stripped there, not in intermediate.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [keinsaasforever/gtm-pipeline-skills](https://github.com/keinsaasforever/gtm-pipeline-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
