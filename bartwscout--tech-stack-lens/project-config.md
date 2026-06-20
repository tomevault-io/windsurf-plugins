---
trigger: always_on
description: TechStackLens — Full-stack AI tooling audit. Audits Claude Code setup, SaaS providers, databases, deployment, cross-tool configs, dependencies, and environment variables. Generates PDF/Markdown/HTML reports. Bidirectional sync with scouting-llm-wiki. Use for periodic health checks, discovering new tools, stack maintenance, or whenever the user wants to audit their AI/dev tooling — including loose phrasings like "check my stack", "what's broken", "audit my tools", or "run tech-stack-lens".
---


<!--
Versioning: bump `version` on every commit that changes skill behavior.
- 0.1.x–0.5.x: phases 1–5 (initial build, wiki read/write, dashboards, docs)
- 0.6.0: recursive wiki path resolution, research fallback ladder, Step 5 dry-run,
         profile persistence, extracted scripts, test fixtures (2026-04-21)
Stamp the version into the header of every generated report.
-->


# TechStackLens — Full-Stack AI Tooling Audit

Periodic audit pass for a user's full technology stack. Ten sections:

0. Profile the user
1. Audit current setup (7 categories, deep)
2. Research what's new (parallel, budget-capped)
3. Filter, diff, compare
3.5. Trust & safety screen
4. Ask by number
5. Install / configure approved tools
6. Generate reports (PDF + Markdown + HTML)
7. Wiki sync (write-path) — auto-generate/update provider notes in scouting-llm-wiki

Never change anything without the user's explicit approval (see Step 4 for the two-confirmation flow). Match language to the user's tech level throughout.

---

## Output Style & Interaction

Every message the skill sends during a run should be scannable at a glance. Structured text only — **do not emit ANSI escape codes or terminal animations** (they render as gibberish in Claude Code's markdown output).

**Section dividers between steps:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  STEP 1/7 · AUDIT
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**Status lines while working** (one per action, short):
```
→ Running: claude mcp list
→ Reading: ~/.claude/settings.json
→ Checking: APOLLO_API_KEY presence
→ Fetching: carnetscout.app/healthz
```

**Findings with a checkmark, bolded numbers:**
```
✓ Found **14 plugins**, **6 skills**, **3 MCP servers** (1 erroring)
✓ **22/28** API keys present · **3** stale · **2** missing
✓ Researched 6 sources in parallel — **23 candidates** collected
```

**Questions — always use `AskUserQuestion` for multi-choice.** Whenever you need input (profile questions in Step 0, install picks in Step 4, REVIEW overrides, confirmations), invoke the `AskUserQuestion` tool with explicit options. Use free text only when the answer can't be listed.

---

## Step 0 — Verify Context, Then Profile

**First, surface what you already think you know about the user.** Never treat memory files, CLAUDE.md, directory names, or prior session summaries as ground truth — they're guesses that need confirmation.

### Persisted profile (fast path)

Before asking anything, check for `config/user-profile.yaml` in the skill directory. If it exists, read the saved profile and show it compactly:

> **Your saved profile (from `config/user-profile.yaml`, saved YYYY-MM-DD):**
> - Use case: `<use_case>`
> - Tech level: `<tech_level>`
> - Default goal: `<default_goal>`
> - PDF style: `<pdf_style>`
> - Audit scope: `<audit_scope>`
> - Notes: `<notes, if any>`

Then ask via `AskUserQuestion`:
- **Options:** `still accurate — run with this` · `change something` · `ignore and ask fresh`

Branch:
- *still accurate* → skip the 5 profile questions entirely, proceed to Step 1 with the saved profile.
- *change something* → ask which field(s), update the file, continue.
- *ignore and ask fresh* → discard the saved profile for this run only (don't delete the file), fall through to the full question flow below.

If `config/user-profile.yaml` does **not** exist, skip the fast path and go straight to the context + 5 questions below. After the questions are answered, offer via `AskUserQuestion`:
- **Options:** `save profile for next time` · `don't save`

When the user says save, write the answers to `config/user-profile.yaml` (the file is git-ignored; see `config/user-profile.yaml.example` for the schema).

### Inferred context (shown on fresh runs OR when the user picks *ignore and ask fresh*)

Show the user a compact list of what you inferred:

> **Here's what I have on file for you. Is this still accurate?**
>
> - Main projects: [list from memory/directory]
> - Role / use case: [inferred]
> - Last audit run: [date if `~/Desktop/tech-stack-lens-*.pdf` exists]

Then ask via `AskUserQuestion`:

- **Options:** `all accurate` · `some wrong — let me edit` · `ignore all of that, start fresh`

Branch:
- *all accurate* → proceed using the inferred context
- *some wrong* → ask which items to correct, one at a time. Update memory files after
- *ignore all* → discard the inferred context entirely, treat as clean slate

**Then ask 5 profile questions — each via `AskUserQuestion` with explicit options:**

1. **Use case** → `coding` · `writing` · `running a business` · `research` · `creative work` · `learning` · `other`
2. **Tech level** → `brand new` · `basics` · `comfortable dev` · `power user`
3. **This session's goal** → `show me what's new` · `health check` · `fix broken things` · `optimize my stack` · `other` (free-form only if other)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BartWScout/tech-stack-lens](https://github.com/BartWScout/tech-stack-lens) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
