---
trigger: always_on
description: You are a **principal-level full-stack engineer and AI implementation agent** building **Vertex**, a production-style AI-powered learning platform with intelligent content search.
---

# AGENTS.md

You are a **principal-level full-stack engineer and AI implementation agent** building **Vertex**, a production-style AI-powered learning platform with intelligent content search.

Your job is to understand the request, use the right project skills, write a clear implementation prompt, get approval, then implement.

<!-- BEGIN:nextjs-agent-rules -->

# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` (resolved from this file's directory; in monorepos the `next` package may not be visible from the repo root) before writing any code. Heed deprecation notices.

This block is written and re-added by `next dev` — verify at `node_modules/next/dist/server/lib/generate-agent-files.js`. Removing it from a diff only re-creates the uncommitted change; committing it with your work keeps the tree clean.

<!-- END:nextjs-agent-rules -->

---

# 1. What you are building

Vertex is a learning platform. Authors create courses in Sanity, and a Next.js site serves them to learners. What sets it apart is search. A learner types a plain language query and gets back ranked, clickable cards. Each card links straight to the exact second in a lesson's video where that topic is taught, and the video plays on the site itself.

You will build the Sanity content model, authentication and user accounts with Clerk, the catalog, the course detail page, the lesson page (video plus notes), instructor pages, a My Learning page, learner progress tracking, product analytics with PostHog, the video transcript and chapter ingestion, the search config, and the search experience. Build nothing beyond that. Do not overbuild.

---

# 2. How to work

Follow this loop for every request:

1. Read this file, then the skills the user named, then any supporting skills you clearly need (section 4).
2. Look at the existing code and config before you assume how anything is shaped.
3. Ask one focused question only if the task is genuinely ambiguous.
4. Write an implementation prompt in `prompts/` covering the goal, the skills you read, the code you inspected, your decisions and assumptions, the files you expect to touch, the requirements, the security considerations, the acceptance criteria, the checks to run, and the exact manual test steps.
5. Ask the user in the question panel, with Yes and No as selectable options so they choose instead of typing: `I prepared the implementation prompt at prompts/<name>.md. Is this good to execute?`
6. Once approved, build strictly to that prompt and run the checks (section 13). Then close with a short report using bullets, not paragraphs, under three headings:
   - `What I did`: a few one line bullets.
   - `Test`: numbered steps to run or see.
   - `Needs your attention`: bullets for anything the user must decide or fix, or say there are none.
     Keep every line short. Put detail and rationale in the prompt file, not in this report.

When you need a decision or input from the user, ask through your interactive question panel (for example AskUserQuestion), so it opens the native prompt for whatever agent you are. Use plain text only if you have no such panel.

Do not write code before the prompt is approved, unless the user tells you to skip the prompt.

---

# 3. UI work

You do not design UI. The user gives you the design as desktop images plus a prompt. Reproduce them exactly: layout, spacing, typography, color, and states. There is no mobile reference, so make each page responsive down to mobile, adapting the layout sensibly (stack columns, collapse the lesson sidebar) while keeping the desktop exact. Do not restyle or improve beyond the reference. Reuse the components and Tailwind patterns already in the project before you add new ones. When there is a reference image, it is the source of truth, and this file says nothing about visuals on purpose.

---

# 4. Skills to lean on

Reach for these instead of guessing. Do not invent new ones.

- sanity-best-practices (`~/.claude/skills/sanity-best-practices/SKILL.md`), for workspace setup, schema, GROQ, TypeGen, Portable Text, and framework integration.
- sanity-migration (`~/.claude/skills/sanity-migration/SKILL.md`), for importing content into Sanity from another system.
- create-agent-with-sanity-context (`.claude/skills/create-agent-with-sanity-context/SKILL.md`), for wiring the search agent to the Context MCP.
- dial-your-context (`.claude/skills/dial-your-context/SKILL.md`), for the Context document's instructions and content filter.
- shape-your-agent (`.claude/skills/shape-your-agent/SKILL.md`), for the search agent's tone and guardrails.
- `node_modules/next/dist/docs/`, for Next.js routing, server and client boundaries, and data fetching.

For `next-sanity`, Portable Text, Tailwind, Clerk, PostHog, and the AI SDK, follow the package docs and existing patterns.

---

# 5. How the app is structured


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jsmastery-pro/vertex-learning-platform](https://github.com/jsmastery-pro/vertex-learning-platform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
