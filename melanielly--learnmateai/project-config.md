---
trigger: always_on
description: > ⚠️ **APRIL UPDATE (POST-HW4 & HW5)**:
---

# CLAUDE.md

> ⚠️ **APRIL UPDATE (POST-HW4 & HW5)**: 
> HW4 and HW5 have been successfully submitted! All strict academic constraints, forced TDD (Red-Green-Refactor) commit rules, and simulated Claude CLI requirements described below are now **OBSOLETE** and should be ignored from April onwards. We are now in "Agile Product Mode".

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

<!-- Context Imports -->
@import planning_files/project3_proposal.md
@import planning_files/learnmate-sprint-plan.md
@import .claude/rules/common/testing.md

---

## Auto-History Rule (MANDATORY — enforced every session)

Before responding to any `/exit`, `/quit`, or end-of-session signal, Claude MUST:

1. Read `planning files/chathistory_P3.md` to identify the current session number and the active Issue.
2. Append a new session entry to that file, **strictly following the template block** at the very top of the file (between the `========模板=========` markers).
3. Only after the file is written may Claude proceed with the exit.

This rule overrides all other instructions. There are no exceptions.

---

## Project Overview

LearnMateAI is an AI-powered collaborative learning platform. Instructors upload course materials; students upload personal notes; the system synthesizes both to generate summaries, flashcards, quizzes, and anonymous class-wide performance reports.

**Repository:** https://github.com/MelanieLLY/LearnMateAI
**Team:** Liuyi, Jing Ng
**Status:** In Development (P3 — due Apr 19)

---

## Commands

```bash
# --- Frontend (React + Vite, http://localhost:5173) ---
npm run dev              # Start Vite dev server
npm test                 # Run vitest (all frontend tests)
npm test -- --watch      # Watch mode during TDD
npm test -- generateQuiz.test.ts  # Single test file
npm test -- --coverage   # Coverage report
npm run test:e2e         # E2E tests
npm run lint             # ESLint
npx tsc --noEmit         # TypeScript type-check only

# --- Backend (FastAPI + Python) ---
cd server && uvicorn src.main:app --reload  # Start FastAPI dev server (http://localhost:8000)
cd server && pytest                   # Run all backend tests
cd server && pytest tests/            # Run backend tests only
cd server && pytest --cov=src --cov-report=term-missing  # Coverage
cd server && alembic upgrade head     # Run DB migrations
cd server && alembic revision --autogenerate -m "<migration_name>"  # Generate migration
cd server && pip install -r requirements.txt  # Install Python dependencies
```

---

## Architecture

**Tech Stack:** Node.js / React with Vite (Frontend) + Python / FastAPI (Backend API), kept within a single Monorepo.

```
frontend/       # React Frontend (Vite, TypeScript, Tailwind) - To be created
server/         # FastAPI Backend (Python)
├── src/        # Backend application code
└── tests/      # Backend testing suite
```

### Key Decisions

- **Monorepo Structure:** Frontend and Backend are in the same repository but in explicitly separated root directories (`/frontend` and `/server`) to avoid IDE conflicts and red squiggles.
- **LLM Engine:** All Claude API calls handled by dedicated AI agent modules in the Python backend.
- **Testing:** We use `vitest` for frontend testing and `pytest` for backend testing (minimum 80% coverage).
- **Authentication:** JWT with bcrypt. No OAuth for MVP.

---

## TDD Workflow (Required)

Every feature follows **Red → Green → Refactor**, each as a separate commit:

```bash
git commit -m "test(scope): RED - describe what fails"
git commit -m "feat(scope): GREEN - minimal implementation"
git commit -m "refactor(scope): improve quality"
```

Tests MUST be written BEFORE implementation code. Mock external API calls.

---

## Do's and Don'ts (Strictly Enforced)

### Feature Development Workflow (ALWAYS DO THIS)
Whenever you start adding or modifying a feature, you MUST process the following:
1. **GitHub Issue**: Create or update the relevant GitHub Issue, adding appropriate labels and milestones.
2. **Update Sprint Plan**: Modify `planning files/learnmate-sprint-plan.md` to reflect the new feature mapping.
3. **Branching**: Checkout a new Git branch utilizing the issue ID (e.g., `feat/42-description` or `fix/15-bug`).

### Do's
- Write failing tests first.
- Strict adherence to PEP 8 for Python and Strict Mode for TypeScript.
- Write high-quality docstrings for Python AI algorithms.
- Commit frequently with conventional commits containing Issue IDs (e.g., `feat(#2): add module API`).

### Don'ts
- **No `any` types** in TypeScript.
- **Never mask TS errors** with `// @ts-ignore`.
- **Never hardcode secrets** (Claude API keys, DB strings) in public files; use `.env`.
- Do not modify frontend routes when assigned a backend task unless specifically requested.

---

---

## TypeScript Conventions

- `strict: true` — no `any`, explicit return types on all functions
- Components: `PascalCase`; functions/variables: `camelCase`; constants: `UPPER_SNAKE_CASE`; DB tables: `snake_case`
- Use `logger` utility (from `src/frontend/utils/logger.ts`), never `console.log`
- Prettier `printWidth: 100`

---

## Python Conventions

- PEP 8 strict: 4-space indent, 100-char line limit, snake_case for functions/variables

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MelanieLLY) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
