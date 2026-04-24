---
trigger: always_on
description: > **Last Updated:** 2026-03-19
---

# CarRental Project - Mandatory Development Workflow

> **Last Updated:** 2026-03-19
> **Project:** Laravel 10 + Vue.js 3 Car Rental Platform + Vrooem Gateway (FastAPI/Python)
> **Enforcement Level:** MANDATORY - Every step MUST be followed. No rationalizing, no skipping.

---

## CRITICAL RULES

1. **NEVER write code without completing the pre-code steps first.**
2. **NEVER claim work is done without running verification.**
3. **NEVER skip a skill invocation** - use the `Skill` tool for each one.
4. **Skills MUST be invoked via the `Skill` tool**, not remembered from previous conversations.
5. **Each step produces visible output** - announce what you're doing before each skill invocation.
6. **If a step doesn't apply, say "Skipping step N: [reason]"** - never silently skip.

---

## STEP-BY-STEP WORKFLOW (Follow in Order)

Every task goes through these phases. For each phase, invoke the skill, follow its instructions, then move to the next phase.

### PHASE 1: UNDERSTAND (Before doing anything)

**Step 1 - Optimize the Prompt**
```
Skill: prompt-engineering-patterns
```
- YOU MUST invoke this skill FIRST for every task
- Analyze what the user is really asking
- Identify ambiguities, missing context, edge cases
- Reframe the task internally before proceeding
- If the task is a simple read-only query (read file, search code, git status), skip to execution

**Step 2 - Research & Context Gathering**
```
Tools: ref_search_documentation, exa MCP, Grep, Glob, LSP, Read
```
- Search documentation for any APIs, libraries, or patterns involved
- Read all relevant files BEFORE proposing changes
- Use LSP for symbol search, Grep for text search
- Understand the existing code before touching it
- Check `docs/plans/` and `docs/apidata/` for project-specific context

---

### PHASE 2: DESIGN (Before writing any code)

**Step 3 - Brainstorm (MANDATORY for all creative work)**
```
Skill: superpowers:brainstorming
```
- YOU MUST invoke this for: new features, new components, behavior changes, refactoring, UI changes
- Explore the user's intent and requirements
- Ask clarifying questions if needed
- Propose 2-3 approaches with tradeoffs
- Get user approval before proceeding
- Skip ONLY for: pure bug fixes with obvious solutions, typo fixes, config changes

**Step 4 - Write Implementation Plan (For multi-step tasks)**
```
Skill: superpowers:writing-plans
```
- YOU MUST invoke this when the task requires changes to 3+ files or has multiple steps
- Create a clear, numbered plan with file paths
- Identify dependencies between steps
- Get user approval on the plan before coding
- Skip for: single-file changes, simple fixes

---

### PHASE 3: IMPLEMENT (Writing code)

**Step 5 - Load Karpathy Guidelines**
```
Skill: karpathy-guidelines
```
- YOU MUST invoke this BEFORE writing or editing ANY code
- Make surgical, minimal changes
- Don't over-engineer or add unnecessary abstractions
- Surface assumptions explicitly
- Define verifiable success criteria

**Step 6 - Load Domain Skill (based on what you're editing)**

Invoke the MATCHING skill based on the file type you're about to edit:

| File/Context | Skill to Invoke | Notes |
|---|---|---|
| `*.vue` files | `vue-best-practices` + `antfu/vue` | Composition API with `<script setup>`, Anthony Fu patterns |
| Vue composables | `vueuse-functions` | VueUse patterns for composable authoring |
| `*.php` Laravel files | `laravel-specialist` + `laravel-11-12-app-guidelines` | Models, Controllers, Services + modern Laravel patterns |
| Database migrations | `mysql` | Indexes, constraints, data types |
| API routes/design | `backend-development:api-design-principles` | REST endpoints, response formats |
| Frontend UI/styling | `frontend-design` | Visual design, CSS, layouts |
| Stripe/payment code | `stripe-integration` | Checkout, webhooks, payment intents |
| Email / Mailables / Notifications | `mailtrap-laravel` | Mailtrap sending API, templates, webhooks, sandbox |
| FastAPI Python (`vrooem-gateway/`) | `fastapi-templates` | Async patterns, dependency injection |
| Redis caching | `redis-best-practices` | Cache strategies, key design |
| Supabase/PostgreSQL | `supabase-postgres-best-practices` | Query optimization, schema design |
| Vue Router changes | `vue-router-best-practices` | Navigation guards, route params |
| Pinia store changes | `vue-pinia-best-practices` | State management patterns |
| Vue testing | `vue-testing-best-practices` | Vitest, Vue Test Utils |

- If editing MULTIPLE file types, invoke ALL matching skills
- If no domain skill matches, proceed without one
- **For ANY frontend UI work**, ALWAYS read `resources/js/design-system.md` first for brand colors, typography, buttons, spacing, shadows, and component patterns. Every new page MUST follow this design system.

**Step 6b - Security Review (for sensitive code)**
```
Skill: security-best-practices
```
- YOU MUST invoke this when editing: payment flows, authentication, user data handling, API endpoints, file uploads
- Review for OWASP top 10, injection, XSS, CSRF, insecure deserialization
- Skip for: UI-only changes, styling, non-sensitive config

**Step 7 - Write the Code**
- Make changes following all loaded skill guidelines
- Write clean, minimal code - no unnecessary additions
- Follow existing patterns in the codebase

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/anshulfanaticcoders) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
