---
trigger: always_on
description: AI Content Generator SaaS with credit-based payments.
---

# Next.js SaaS - Claude Code Configuration

## Context
AI Content Generator SaaS with credit-based payments.

**Core Principle**: Navigator workflow → Load docs on-demand, not upfront (92% token reduction)

---

## Navigator Workflow (CRITICAL - ENFORCE STRICTLY)

### SESSION START PROTOCOL (MANDATORY)

**🚨 EVERY new conversation/session MUST begin with**:

```bash
/nav:start
```

**What `/nav:start` does**:
1. Loads `.agent/DEVELOPMENT-README.md` (navigator)
2. Sets Navigator workflow context
3. Activates token optimization strategy
4. Reminds about agent usage for complex tasks

**If you don't explicitly run `/nav:start`**:
- Claude MUST proactively run it or ask to run it
- Never proceed with work without loading navigator
- This is NOT optional - it's the foundation of Navigator

---

### Lazy-Load Documentation Based on Task

**Implementing feature?**
```
1. Read .agent/DEVELOPMENT-README.md (2k)
2. Read .agent/tasks/TASK-XX-feature.md (3k)
3. Read relevant .agent/system/ doc (5k)
Total: 10k tokens vs 150k
```

**Debugging issue?**
```
1. Read .agent/DEVELOPMENT-README.md (2k)
2. Check .agent/sops/debugging/ for relevant SOP (2k)
3. Read relevant system doc if needed (5k)
Total: 9k tokens vs 150k
```

**Adding integration?**
```
1. Read .agent/DEVELOPMENT-README.md (2k)
2. Check .agent/sops/integrations/ for similar pattern (2k)
3. Read .agent/system/project-architecture.md (5k)
Total: 9k tokens vs 150k
```

---

### Autonomous Task Completion (CRITICAL)

**When task implementation is complete, execute finish protocol AUTOMATICALLY**:

✅ **DO automatically** (no human prompt needed):
1. **Commit changes** with proper conventional commit message
2. **Archive implementation plan** via `/nav:update-doc feature TASK-XX`
3. **Create completion marker** `TASK-XX-complete`
4. **Suggest compact** to clear context for next task

❌ **DON'T wait for**:
- "Please commit now"
- "Update documentation"
- "Create a marker"

**Exception cases** (ask first):
- Uncommitted files contain secrets (.env, credentials, API keys)
- Multiple unrelated tasks modified (unclear which to close)
- No task context loaded (ambiguous which TASK-XX)
- Tests failing or implementation incomplete

---

### Smart Compact Strategy

**Run `/nav:compact` after**:
- Completing isolated sub-task
- Finishing documentation update
- Creating SOP
- Switching between unrelated tasks

**Don't compact when**:
- In middle of feature implementation
- Context needed for next sub-task
- Debugging complex issue

---

## Code Standards

### Next.js 15 Patterns

**Routing**: App Router (not Pages Router)
- Route groups: `(auth)`, `(dashboard)`
- Layouts for shared UI
- Loading/error states per route

**Components**:
- Server Components by default (no 'use client')
- 'use client' ONLY for: interactivity, hooks, browser APIs
- Async Server Components for data fetching

**Data Fetching**:
- Server Components: Direct Supabase calls
- Client Components: Server Actions or API routes
- No client-side database queries

### TypeScript

- Strict mode enabled
- No `any` without justification
- Props interfaces for all components
- Type imports for better tree-shaking

### Styling

- Tailwind utility classes
- Component-specific styles in same file
- Avoid global styles (except `globals.css`)
- Mobile-first responsive design

### Database

- Supabase client creation: `utils/supabase/server.ts` or `client.ts`
- Always use Row-Level Security (RLS)
- Server-side queries use service role (when needed)
- Client-side queries use anon key

---

## Tech Stack

- **Framework**: Next.js 15 (App Router, SSR)
- **Language**: TypeScript (strict mode)
- **Styling**: Tailwind CSS
- **Database**: Supabase (PostgreSQL)
- **Auth**: Supabase Auth
- **Payments**: Stripe
- **AI**: OpenAI API
- **Hosting**: Vercel

---

## Forbidden Actions

### Navigator Violations (HIGHEST PRIORITY)
- ❌ NEVER wait for explicit commit prompts after task completion (autonomous mode)
- ❌ NEVER skip documentation after completing features (knowledge loss)
- ❌ NEVER load all `.agent/` docs at once (defeats context optimization)
- ❌ NEVER skip reading DEVELOPMENT-README.md (navigator is essential)
- ❌ NEVER create docs outside `.agent/` structure (breaks discovery)

### General Violations
- ❌ No Claude Code mentions in commits/code
- ❌ No package.json modifications without approval
- ❌ Never commit secrets/API keys/.env files
- ❌ Don't use Pages Router (use App Router)
- ❌ Don't use Client Components unnecessarily

---

## Development Workflow

1. **Start Session** → `/nav:start` (loads navigator)
2. **Select Task** → Load task doc (`.agent/tasks/TASK-XX.md`)
3. **Plan** → Use TodoWrite for complex tasks
4. **Implement** → Follow Next.js + Navigator patterns
5. **Verify** → Test locally, check functionality
6. **Complete** → [AUTONOMOUS] Commit, document, create marker
7. **Compact** → Run `/nav:compact` to clear context for next task

---

## Documentation System

### Structure
```
.agent/
├── DEVELOPMENT-README.md      # Navigator (always load first)
├── tasks/                     # Implementation plans
├── system/                    # Living architecture docs
└── sops/                      # Standard Operating Procedures
    ├── integrations/
    ├── debugging/
    └── development/
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [qf-studio/navigator](https://github.com/qf-studio/navigator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
