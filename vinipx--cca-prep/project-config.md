---
trigger: always_on
description: CCA Prep is a community study site for the Claude Certified Architect — Foundations exam. Not affiliated with Anthropic.
---

# CCA Prep — Claude Context

## Project Identity

CCA Prep is a community study site for the Claude Certified Architect — Foundations exam. Not affiliated with Anthropic.

- **Stack:** Docusaurus 3.9.2, React 19, TypeScript 5.6
- **Hosted:** GitHub Pages — `https://www.claudecertprep.com`
- **Node requirement:** >=20.0

## Build Commands

```
npm start            # dev server (localhost:3000)
npm run build        # production build
npm run typecheck    # tsc — run after EVERY data file edit
npm run serve        # serve the production build locally
```

Always run `npm run typecheck` after editing any `.ts` data file. TypeScript catches missing/wrong fields immediately (union types for Domain and Tier).

## Key File Paths

```
src/data/questions.ts         — Question[] array
src/data/flashcards.ts        — Flashcard[] array
src/data/cheatsheet.ts        — CheatsheetItem[] array
src/data/antipatterns.ts      — AntiPattern[] array

src/components/QuizEngine.tsx
src/components/FlashcardDeck.tsx
src/components/CheatsheetView.tsx
src/components/AntiPatternList.tsx
src/components/DomainCards.tsx
src/components/ExamStats.tsx

src/css/custom.css            — design system tokens
src/pages/index.tsx           — landing page
src/pages/quiz.tsx
src/pages/flashcards.tsx
src/pages/cheatsheet.tsx
src/pages/overview.tsx

docs/domain1/                 — study guide markdown files
docs/domain2/
docs/domain3/
docs/domain4/
docs/domain5/
docs/contributing.md

docusaurus.config.ts
sidebars.ts                   — register new docs here
.claude/commands/             — skill files (slash commands)
```

Named exports that matter: `QUESTIONS`, `FLASHCARDS`, `CHEATSHEET_ITEMS`, `ANTI_PATTERNS`, `DOMAIN_LABELS`, `DOMAIN_WEIGHTS`, `TIERS`, `QUESTION_COUNT`.

## Domain Reference

| ID | Label | Weight | Key Topics |
|----|-------|--------|------------|
| 1 | Agentic Architecture & Orchestration | 27% | agentic loops, stop_reason, Task tool, hub-and-spoke, hooks, session management, fork_session/--resume |
| 2 | Tool Design & MCP Integration | 18% | tool descriptions, MCP error schema, isError/isRetryable, tool_choice, tool scoping, .mcp.json |
| 3 | Claude Code Configuration & Workflows | 20% | CLAUDE.md hierarchy, plan mode, CI/CD integration, .mcp.json scoping, skill files, argument-hint |
| 4 | Prompt Engineering & Structured Output | 20% | structured output, validation-retry loop, Batch API decision criteria, tool_choice, few-shot |
| 5 | Context Management & Reliability | 15% | context window strategies, lost-in-the-middle, escalation triggers, structured facts extraction, error propagation |

## Tier Definitions

| Tier | What It Tests | scenario field |
|------|--------------|----------------|
| `basic` | Single-concept recall — "what does X mean?" | Not present |
| `intermediate` | Applied understanding — "what is wrong with this approach?" | Not present |
| `advanced` | Multi-constraint reasoning — tradeoffs across concepts | Not present |
| `exam` | Full scenario with business context, mirrors real CCA exam | **Required** |

Rule: `tier:'exam'` questions **must** have a `scenario` field. All other tiers **must not**.

## Exam Scenarios

The 6 canonical scenarios used in `tier:'exam'` questions:

1. **Customer Support Resolution Agent** — Domains 1, 2, 5
2. **Code Generation with Claude Code** — Domains 3, 5
3. **Multi-Agent Research System** — Domains 1, 2, 5
4. **Developer Productivity Tools** — Domains 1, 2, 3
5. **Claude Code for CI/CD** — Domains 3, 4
6. **Structured Data Extraction** — Domains 4, 5

The `scenario` string in a question is 2-3 sentences of business context that makes multiple answer options plausible. Cross-reference domain when choosing a scenario.

## Data Schemas

```typescript
// questions.ts
interface Question {
  id: number;
  domain: Domain;           // 1 | 2 | 3 | 4 | 5
  tier: Tier;               // 'basic' | 'intermediate' | 'advanced' | 'exam'
  scenario?: string;        // required if tier === 'exam', absent otherwise
  text: string;
  options: string[];        // exactly 4
  correct: number;          // 0-indexed (0–3)
  explanation: string;
  wrongExplanations?: string[]; // optional, 3 entries for the 3 wrong options
  refs: Ref[];              // required, at least 1 entry
}
interface Ref { label: string; url: string; }

// flashcards.ts
interface Flashcard {
  id: number;
  domain: Domain;
  front: string;
  back: string;  // uses \n for line breaks, • for bullets
}

// cheatsheet.ts
interface CheatsheetItem {
  id: number;
  domain: Domain;
  tag: string;    // short category label, e.g. 'Agentic loops', 'MCP errors', 'CLAUDE.md'
  title: string;
  body: string;
  code?: string;  // optional code block, TypeScript preferred
}

// antipatterns.ts
interface AntiPattern {
  id: number;
  domain: Domain;
  title: string;   // gerund: "Using X to do Y"
  why: string;
  correct: string;
  examTip: string;
}
```

## Content Quality Standards

**For all content types:**
- IDs are sequential — read the last `id` in the target file and increment by 1
- `domain` is an integer (1-5), not a string
- Run `npm run typecheck` after every edit

**Questions:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vinipx/cca-prep](https://github.com/vinipx/cca-prep) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-16 -->
