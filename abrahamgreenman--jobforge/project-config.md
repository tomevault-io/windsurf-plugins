---
trigger: always_on
description: AI-powered job search automation built on opencode: pipeline tracking, offer evaluation, CV generation, portal scanning, batch processing.
---

# JobForge -- AI Job Search Pipeline

## What is JobForge

AI-powered job search automation built on opencode: pipeline tracking, offer evaluation, CV generation, portal scanning, batch processing.

**It will work out of the box, but it's designed to be made yours.** If the archetypes don't match your career, the modes are in the wrong language, or the scoring doesn't fit your priorities -- just ask. You (opencode) can edit any file in this system. The user says "change the archetypes to data engineering roles" and you do it. That's the whole point.

### Main Files

| File | Function |
|------|----------|
| `data/applications.md` | Application tracker |
| `data/pipeline.md` | Inbox of pending URLs |
| `data/scan-history.tsv` | Scanner dedup history |
| `portals.yml` | Query and company config |
| `templates/cv-template.html` | HTML template for CVs |
| `generate-pdf.mjs` | Geometra MCP (`geometra_generate_pdf`): HTML to PDF |
| `article-digest.md` | Compact proof points from portfolio (optional) |
| `interview-prep/story-bank.md` | Accumulated STAR+R stories across evaluations |
| `reports/` | Evaluation reports (format: `{###}-{company-slug}-{YYYY-MM-DD}.md`) |

### First Run — Onboarding (IMPORTANT)

**Before doing ANYTHING else, check if the system is set up.** Run these checks silently every time a session starts:

1. Does `cv.md` exist?
2. Does `config/profile.yml` exist (not just profile.example.yml)?
3. Does `portals.yml` exist (not just templates/portals.example.yml)?

**If ANY of these is missing, enter onboarding mode.** Do NOT proceed with evaluations, scans, or any other mode until the basics are in place. Guide the user step by step:

#### Step 1: CV (required)
If `cv.md` is missing, ask:
> "I don't have your CV yet. You can either:
> 1. Paste your CV here and I'll convert it to markdown
> 2. Paste your LinkedIn URL and I'll extract the key info
> 3. Tell me about your experience and I'll draft a CV for you
>
> Which do you prefer?"

Create `cv.md` from whatever they provide. Make it clean markdown with standard sections (Summary, Experience, Projects, Education, Skills).

#### Step 2: Profile (required)
If `config/profile.yml` is missing, copy from `config/profile.example.yml` and then ask:
> "I need a few details to personalize the system:
> - Your full name and email
> - Your location and timezone
> - What roles are you targeting? (e.g., 'Senior Backend Engineer', 'AI Product Manager')
> - Your salary target range
>
> I'll set everything up for you."

Fill in `config/profile.yml` with their answers. For archetypes, map their target roles to the closest matches and update `modes/_shared.md` if needed.

#### Step 3: Portals (recommended)
If `portals.yml` is missing:
> "I'll set up the job scanner with 45+ pre-configured companies. Want me to customize the search keywords for your target roles?"

Copy `templates/portals.example.yml` → `portals.yml`. If they gave target roles in Step 2, update `title_filter.positive` to match.

#### Step 4: Tracker
If `data/applications.md` doesn't exist, create it:
```markdown
# Applications Tracker

| # | Date | Company | Role | Score | Status | PDF | Report | Notes |
|---|------|---------|------|-------|--------|-----|--------|-------|
```

#### Step 5: Ready
Once all files exist, confirm:
> "You're all set! You can now:
> - Paste a job URL to evaluate it
> - Run `/job-forge scan` to search portals
> - Run `/job-forge` to see all commands
>
> Everything is customizable — just ask me to change anything.
>
"

Then suggest automation:
> "Want me to scan for new offers automatically? I can set up a recurring scan every few days so you don't miss anything. Just say 'scan every 3 days' and I'll configure it."

If the user accepts, use the `/loop` or `/schedule` skill (if available) to set up a recurring `/job-forge scan`. If those aren't available, suggest adding a cron job or remind them to run `/job-forge scan` periodically.

### Personalization

This system is designed to be customized by YOU (opencode). When the user asks you to change archetypes, translate modes, adjust scoring, add companies, or modify negotiation scripts -- do it directly. You read the same files you use, so you know exactly what to edit.

**Common customization requests:**
- "Change the archetypes to [backend/frontend/data/devops] roles" → edit `modes/_shared.md`
- "Translate the modes to English" → edit all files in `modes/`
- "Add these companies to my portals" → edit `portals.yml`
- "Update my profile" → edit `config/profile.yml`
- "Change the CV template design" → edit `templates/cv-template.html`
- "Adjust the scoring weights" → edit `modes/_shared.md` and `batch/batch-prompt.md`

### Skill Modes

| If the user... | Mode |
|----------------|------|
| Pastes JD or URL | auto-pipeline (evaluate + report + PDF + tracker) |
| Asks to evaluate offer | `offer` |
| Asks to compare offers | `compare` |
| Wants LinkedIn outreach | `contact` |
| Asks for company research | `deep` |
| Wants to generate CV/PDF | `pdf` |
| Evaluates a course/cert | `training` |
| Evaluates portfolio project | `project` |
| Asks about application status | `tracker` |
| Fills out application form | `apply` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AbrahamGreenman/JobForge](https://github.com/AbrahamGreenman/JobForge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
