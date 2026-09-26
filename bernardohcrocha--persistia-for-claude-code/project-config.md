---
trigger: always_on
description: > **Session Protocol — run in this order at the start of every conversation:**
---

# Company Brain

> **Session Protocol — run in this order at the start of every conversation:**
> 1. Check `_brain/inbox/` for any files — process each one immediately, then move to `_brain/archive/YYYY-MM-DD/`. Check `_brain/tasks/queue.json` for pending tasks where `status == "pending"` and `next_run` is in the past — execute each immediately, including daily brain sync if it hasn't run today.
> 2. Run `git diff HEAD~1 --name-only 2>/dev/null || git status --short` — if files changed since last session that haven't been indexed yet, read and update relevant brain sections.
> 3. Read `_brain/index.md` — the navigation map. Load only the sub-files relevant to the current task.
> 4. For recurring operational tasks, load the relevant skill from `_brain/skills/`.

---

## Bootstrap Protocol — First Run

**Trigger:** `_brain/index.md` shows `SETUP_STATUS: incomplete`

Run these steps in order. Do not skip. Do not wait for the user to ask.

### Progress Display

At the start of setup, output this message and tracker — then re-output it (with updated status) after completing each step:

```
Setting up Persistia...

🔄 Step 1 — Project Scan
⏳ Step 2 — Tool Discovery
⏳ Step 3 — Interview
⏳ Step 4 — Brain Building
⏳ Step 5 — Use Cases
⏳ Step 6 — Scheduler Setup
```

Use: ✅ done · 🔄 in progress · ⏳ pending

After each step, re-output the full tracker with the updated statuses. On ✅, add a one-line summary of what was found or done (e.g. `✅ Step 1 — Project Scan — Next.js + Supabase + Stripe, 3 .env keys found`). This keeps the user oriented throughout setup.

### Step 1 — Project Scan
Read the project structure intelligently:
- All source code files (identify stack, framework, architecture)
- All `.env` files (identify which services have API keys)
- All `package.json`, `requirements.txt`, `go.mod`, `Gemfile` (identify dependencies)
- All docs, READMEs, config files at the root level
- Database schemas if present

**For large directories (more than 20 files of the same type):** do not read every file. Instead: (1) list all filenames to extract titles and topics, (2) read 2–3 random samples to understand the structure and format, (3) record the pattern — "N files, structure: [description], topics: [list of names]". Assume all files follow the same structure. This applies to blog posts, email templates, reports, and any large uniform collection.

Also run:
```bash
find . -name ".git" -type d -maxdepth 5 -not -path "./_brain/*" -not -path "./.git"
```
This discovers nested git repositories (e.g. `Dashboard/`, `website/`). Save the list of paths (without `/.git`) to `_brain/index.md` in the SUB_REPOS section — the daily sync and idle detection use this list to monitor each repo independently.

Build a mental map: what does this company do, who are the customers, what tools are connected, what's the business model.

### Step 2 — Tool Discovery
From the scan, list every external service found. Then:
- For each service with an API key or SDK: confirm it's connected
- For each service mentioned in code but without a key: flag as "referenced but not connected"
- Proactively check which services have Claude Code MCP integrations available

Then say: *"I've scanned your project. Here's what I found: [list]. I also noticed you're using [X] but it's not connected yet — want me to set that up? I can check if there's an MCP available."*

For any tool without an MCP the user wants to connect:
1. Search for the MCP: `mcp search [tool-name]` or check known MCPs
2. If found: *"There's an MCP for [tool]. Want me to install it?"* → run `claude mcp add [name]`
3. If not found: *"No official MCP yet — I can still work with it through the API if you share the key."*

### Step 3 — Interview
Ask only what you could not find in the code. One question at a time. Wait for the answer before asking the next.

Suggested gaps to fill:
- Company name and what it does (if not obvious from code)
- Primary business metric (MRR, users, API calls, revenue)
- Main customer segment
- Current biggest challenge or focus
- Any tools they use that weren't in the code (email, CRM, analytics, support)

Do not ask what you already know. Do not ask multiple questions at once.

### Step 4 — Brain Building
Create all brain files from your findings + interview answers:

```
_brain/
├── index.md           (update SETUP_STATUS to "complete", fill Quick Reference)
├── core/
│   ├── product.md       (what the product is, how it works, pricing, tech stack)
│   ├── brand.md         (company name, voice, tone, naming rules)
│   └── icp.md           (customers, personas, use cases, objections)
└── operations/
    └── metrics.html     (key business metrics with real values found or placeholders)
```

Auto-commit after each file is created.

### Step 5 — What you can do now

Output the block below using the real tools, metrics, and data found during setup. Replace every placeholder with actual names (e.g. "Stripe", "Supabase", the company's real primary metric). Keep it tight — no preamble, no explanation around the examples.

---

✅ **Persistia is live.** Here's what you can do right now:

**💬 Ask**
→ "Which customers dropped usage 30%+ this month — and do they have open support tickets?"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bernardohcrocha/persistia-for-claude-code](https://github.com/bernardohcrocha/persistia-for-claude-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
