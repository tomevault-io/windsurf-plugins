---
trigger: always_on
description: Bootstrap a NEW project from scratch using Ivan's preferred stack and conventions, then write production-ready first feature code. Triggered when Ivan describes what he wants to build ("napravi mi novi projekat za X", "treba mi nova app", "novi monorepo", "scaffold a Next.js app", "new project for ...", "start a new SaaS", "imam ideju za app"). For fuzzy ideas, enters Phase 0 (discovery) — uses Tavily MCP for competitor research and Context7 MCP for library docs. For web apps, lets user pick Nex
---


# New Project Bootstrap

Use when Ivan wants to **start a brand-new project**. The skill picks a tech stack and folder layout based on what he's building, using his actual habits across past projects (not generic best-practices).

## When NOT to use

- Existing project being prepared for Claude → use `ai-project-init`
- Adding a feature to an existing project → use module/feature-specific skills
- Just exploring ideas, no commitment to scaffold → answer the question, do not run this skill

## Tools to use (and when)

Default to MCP-aware research instead of guessing. Check what's available in the current session:

| Need                                                                                                       | Tool                                                 | When                                                                                                                                                                |
| ---------------------------------------------------------------------------------------------------------- | ---------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Library / framework / SDK docs (current API, version-specific)                                             | `context7` MCP (`resolve-library-id` + `query-docs`) | ALWAYS prefer over WebFetch when the question is about a library, framework, or SDK. Even for well-known ones like React, Next.js, Prisma — training data is stale. |
| Competitive / market / inspiration research, "what apps exist for X", "how do others solve Y", recent news | `tavily` MCP (`tavily_search`, `tavily_research`)    | When user wants idea exploration, competitor analysis, or general web search. Prefer over WebFetch for unknown URLs.                                                |
| Specific known URL the user provided                                                                       | `WebFetch`                                           | Only when user gives a direct URL to read                                                                                                                           |
| Generic web search with no MCP available                                                                   | `WebSearch`                                          | Last resort if neither MCP is available                                                                                                                             |

**Rule**: before any web research, check what MCPs are exposed in the session. If `tavily_*` or `context7` tools appear, use them. Don't fall back to WebFetch/WebSearch if a better-targeted MCP exists.

## Phase 0 — Discovery & brainstorm (when the idea is fuzzy)

If the user's project description is vague ("nešto za fitnes", "neki AI tool", "marketing site za moj startup"), DO NOT jump to Phase 1. First, work with the user to sharpen the idea.

### When to enter Phase 0

Enter Phase 0 if any of the following:

- The pitch is one short sentence with no clear product shape
- User explicitly asks for help thinking through the idea ("pomozi mi da razradim", "nisam siguran šta tačno")
- The target user / core feature / monetization model is unclear
- The user wants to research what already exists before deciding

### What Phase 0 covers

Have a real conversation, not an interrogation. Aim for 5-10 minutes of back-and-forth covering:

1. **Core value** — what does the user actually solve, and for whom? Force a one-sentence answer.
2. **Differentiator** — what makes it not just "another X"? (If there's no answer, that's a flag — say so.)
3. **MVP scope** — what's the smallest thing that could prove the idea? List 3-5 features max.
4. **Out of scope (v1)** — what is explicitly NOT in the first version. This is as important as what's in.
5. **Target users** — who, how many, what device, what context (mobile-first vs desktop-first)
6. **Data shape** — what entities exist, who owns them, what's the privacy model
7. **Monetization (if any)** — free, freemium, B2B, ads, no money yet

### Tools during Phase 0

- **Tavily** — competitor scan ("what apps exist for habit tracking with social features"), market signals, recent funding/news on the space. Use `tavily_research` for deep dives, `tavily_search` for quick lookups.
- **Context7** — only if user mentions a specific library/SDK they want to use ("hoću da koristim X SDK"); confirm it's still alive and check current API.
- **AskUserQuestion** — for branching choices (mobile-first vs web-first, free vs paid).

Use Tavily/Context7 PROACTIVELY when relevant — don't make the user ask. If they say "ne znam šta već postoji", run a Tavily scan and summarize.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ivanmeda993/claude-new-project-bootstrap](https://github.com/ivanmeda993/claude-new-project-bootstrap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
