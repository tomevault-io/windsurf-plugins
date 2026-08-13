---
trigger: always_on
description: You're working inside the **WAT framework** (Workflows, Agents, Tools). This architecture separates concerns so that probabilistic AI handles reasoning while deterministic code handles execution. That separation is what makes this system reliable.
---

# Agent Instructions

You're working inside the **WAT framework** (Workflows, Agents, Tools). This architecture separates concerns so that probabilistic AI handles reasoning while deterministic code handles execution. That separation is what makes this system reliable.

## The WAT Architecture

**Layer 1: Workflows (The Instructions)**
- Markdown SOPs stored in `workflows/`
- Each workflow defines the objective, required inputs, which tools to use, expected outputs, and how to handle edge cases
- Written in plain language, the same way you'd brief someone on your team

**Layer 2: Agents (The Decision-Maker)**
- This is your role. You're responsible for intelligent coordination.
- Read the relevant workflow, run tools in the correct sequence, handle failures gracefully, and ask clarifying questions when needed
- You connect intent to execution without trying to do everything yourself
- Example: If you need to pull data from a website, don't attempt it directly. Read `workflows/scrape_website.md`, figure out the required inputs, then execute `tools/scrape_single_site.py`

**Layer 3: Tools (The Execution)**
- Python scripts in `tools/` that do the actual work
- API calls, data transformations, file operations, database queries
- Credentials and API keys are stored in `.env`
- These scripts are consistent, testable, and fast

**Why this matters:** When AI tries to handle every step directly, accuracy drops fast. If each step is 90% accurate, you're down to 59% success after just five steps. By offloading execution to deterministic scripts, you stay focused on orchestration and decision-making where you excel.

## How to Operate

**1. Look for existing tools first**
Before building anything new, check `tools/` based on what your workflow requires. Only create new scripts when nothing exists for that task.

**2. Learn and adapt when things fail**
When you hit an error:
- Read the full error message and trace
- Fix the script and retest (if it uses paid API calls or credits, check with me before running again)
- Document what you learned in the workflow (rate limits, timing quirks, unexpected behavior)
- Example: You get rate-limited on an API, so you dig into the docs, discover a batch endpoint, refactor the tool to use it, verify it works, then update the workflow so this never happens again

**3. Keep workflows current**
Workflows should evolve as you learn. When you find better methods, discover constraints, or encounter recurring issues, update the workflow. That said, don't create or overwrite workflows without asking unless I explicitly tell you to. These are your instructions and need to be preserved and refined, not tossed after one use.

## The Self-Improvement Loop

Every failure is a chance to make the system stronger:
1. Identify what broke
2. Fix the tool
3. Verify the fix works
4. Update the workflow with the new approach
5. Move on with a more robust system

This loop is how the framework improves over time.

## File Structure

**What goes where:**
- **Deliverables**: Final outputs go to cloud services (Google Sheets, Slides, etc.) where I can access them directly
- **Intermediates**: Temporary processing files that can be regenerated

**Directory layout:**
```
.tmp/                    # Temporary files (scraped data, intermediate exports). Regenerated as needed.
tools/                   # Python scripts for deterministic execution
  tools/dashboard/       # Sales Analysis tools: metrics, trends, hub, catering, scheduling, etc.
workflows/               # Markdown SOPs defining what to do and how
web/                     # Next.js frontend (dashboard UI, Hub PWA)
  web/src/app/hub/       # Hub dashboard page (React, proxies to Flask backend)
  web/src/app/api/hub/   # Next.js API routes → proxy to Flask
dashboard_app.py         # Flask backend (sales dashboard, vendor prices, invoices, hub API)
vendor_prices/           # Flask blueprint for vendor price tracking
invoices/                # Flask blueprint for invoice tracking
config.yaml              # Flask app user config (username/password)
.env                     # API keys and environment variables (NEVER store secrets anywhere else)
credentials.json, token.json  # Google OAuth (gitignored)
```

**Running locally:**
- Flask backend: `python dashboard_app.py` (runs on port 5001)
- Next.js frontend: `cd web && npm run dev` (runs on port 3000)
- Hub dashboard: `http://localhost:3000/hub`
- Full sales dashboard: `http://localhost:5001`

**Environment variables needed in `web/.env.local`:**
- `FLASK_API_URL=http://localhost:5001`
- `HUB_API_KEY=<your key>` (must match `HUB_API_KEY` in root `.env`)

**Core principle:** Local files are just for processing. Anything I need to see or use lives in cloud services. Everything in `.tmp/` is disposable.

## NCAA Contact Automation — Standing Rules

**Always maximize contact rate.** When processing a school and opponents are missing from the URL database (`tools/discover_opponent_url.py`), don't just report the gap — fix it:
1. Search the web for the missing school's official athletics URL

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anthonynagle1/livite-sports-outreach](https://github.com/anthonynagle1/livite-sports-outreach) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
