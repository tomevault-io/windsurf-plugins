---
trigger: always_on
description: You are a **principal-level full-stack engineer and AI implementation agent** working on **SKEW**, a production-style AI-powered news analysis website.
---

# AGENTS.md

You are a **principal-level full-stack engineer and AI implementation agent** working on **SKEW**, a production-style AI-powered news analysis website.

Your job is to understand the request, use the right project skills, create a clear implementation prompt, ask for approval, then implement.

<!-- BEGIN:nextjs-agent-rules -->

# This is NOT the Next.js you know

This version has breaking changes APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.

<!-- END:nextjs-agent-rules -->

---

# 1. Product

SKEW collects real news articles from configured sources, analyzes them with AI, stores them in Supabase, and displays reader-friendly sentiment and framing insights.

Build only:

- home page with news cards
- news details page with full article analysis
- Clerk authentication
- Supabase persistence
- Oxylabs scraping
- Oxylabs Scheduler
- AI article analysis
- logs
- pgvector similarity search for related articles
- Vercel Cron for automatic scheduling
- minimal responsive UI

Do not overbuild.

---

# 2. Workflow

For every implementation request:

1. Read `AGENTS.md`.
2. Read the skills explicitly mentioned by the user.
3. Read clearly needed supporting skills from the approved skill list.
4. Inspect relevant code.
5. Ask a focused question only if the task has meaningful ambiguity.
6. Create a detailed prompt file in `prompts/`.
7. Ask: `I prepared the implementation prompt at prompts/<file-name>.md. Is this good to execute?`
8. On approval, re-read the approved prompt file in prompts/ and implement it strictly. Implement only after user approval.
9. Run available checks.
10. Share exact steps to test or run the completed feature.

Do not code before creating the prompt unless the user explicitly says to skip prompt creation.

---

# 3. Skills

Use only these skills:

- `.agents/skills/clerk`
- `.agents/skills/supabase`
- `.agents/skills/oxylabs-web-scraper`
- `.agents/skills/ai-sdk`

Use them for:

- `node_modules/next/dist/docs/`: Next.js, routing, server/client boundaries, API routes, UI patterns
- `clerk`: authentication and protected routes
- `supabase`: schema, migrations, queries, service role usage, dedupe, logs, pgvector
- `oxylabs-web-scraper`: Oxylabs Web Scraper API, Scheduler, scheduled jobs, scraping behavior
- `ai-sdk`: Vercel AI SDK and OpenAI provider usage, model calls, AI analysis output handling

Do not invent new skills.

For Cheerio, Zod, Tailwind, and shadcn/ui, use existing project patterns, package docs, and `node_modules/next/dist/docs/`.

---

# 4. Prompt files

Prompt files live in the `prompts/` directory. Use names like:

- `prompts/oxylabs-scraping.md`
- `prompts/oxylabs-scheduler.md`
- `prompts/ai-analysis.md`
- `prompts/news-details-page-ui.md`

Each prompt must include:

- goal
- skills read
- existing code inspected
- decisions or assumptions
- files likely to change
- implementation requirements
- security requirements
- acceptance criteria
- checks to run
- exact manual test steps expected after implementation

For UI tasks, also include visual interpretation, layout, typography, spacing, colors, responsiveness, and pixel-perfect expectations.

---

# 5. Architecture

Keep these layers separate:

- Website: pages, cards, details UI, auth UI
- API: thin route handlers only
- Database: Supabase reads/writes
- Scraping: Oxylabs calls and Scheduler integration
- Parsing: article link extraction, cleanup, article validation
- AI: article analysis and output validation
- Pipeline: scrape and analysis orchestration, log tracking
- Vector: pgvector similarity queries and article embedding storage

UI must display stored data only.

UI must not scrape, analyze, or mutate pipeline state.

---

# 6. Tech stack

Use:

- Next.js
- Clerk
- Supabase
- Oxylabs Web Scraper API
- Oxylabs Scheduler
- Cheerio
- Vercel AI SDK
- OpenAI provider
- Zod
- Tailwind CSS
- shadcn/ui
- pgvector (via Supabase Extensions)
- Vercel Cron

Do not use:

- Supabase Auth
- local JSON app storage
- a separate backend framework

---

# 7. Supabase source of truth

Supabase is the source of truth for app data.

Core tables:

- `sources`
- `articles`
- `article_analyses`
- `logs`
- `oxylabs_schedules`
- `oxylabs_schedule_runs`

Scraping must load active sources from the `sources` table.

Do not hardcode source URLs inside scraping logic or `AGENTS.md`.

Each source should store the fields needed by the scraper:

- name
- homepage URL (listing_url)
- parser strategy if needed
- active status
- optional logo URL

Only active sources should be used for scraping and scheduling.

Each article should store:

- source reference
- original URL (unique, used for dedupe)
- canonical URL
- title
- image URL (required before saving)
- published date (required before saving)
- raw article text
- scraped timestamp
- analyzed timestamp (null until analysis is saved)

Each article analysis should store:

- article reference
- neutral summary
- sentiment score (âˆ’1 to 1) and sentiment label (positive / neutral / negative)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [adrianhajdin/skew_news](https://github.com/adrianhajdin/skew_news) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-17 -->
