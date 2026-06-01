---
trigger: always_on
description: > **This document is the authoritative source of truth for this repository.**
---

# AGENTS.md — Agent Guide

> **This document is the authoritative source of truth for this repository.**
> When any convention here conflicts with another document (including `README.md`,
> inline comments, or code), **this file wins**. If you change how the repository is
> structured, built, or run, you **must update this file in the same change**.

This is a **generic, reusable full-stack monorepo template**. It is meant to be cloned
as the starting point for new, unrelated projects. It ships **scaffolding, tooling,
documentation, and minimal working examples** — never project-specific business logic.
Read this guide fully before making changes, and follow it on every task.

---

## 1. Purpose

This template gives you a typed, conventional starting point for a full-stack project:

- A typed **Python backend** (FastAPI, managed with `uv`) at `apps/api`.
- A typed **TypeScript frontend** (Next.js App Router, managed with `pnpm`) at `apps/web`.
- A reserved **shared package** placeholder at `packages/shared`.

The goal is to let a Developer (human or AI agent) start building immediately with all
static checks passing from the first clone. Everything is typed end to end: Python uses
full type hints checked by `ty`; TypeScript runs in strict mode checked by `tsc --noEmit`.

The template stays **small, flat, explicit, and generic**. It is production-ready without
being enterprise-heavy. Example code uses placeholder names (an `Item` resource with an
`id` and a `name`) and carries no domain meaning, so it can be cloned into any project
without inheriting anything that must be deleted.

> **No testing setup — by design.** This template intentionally has **no testing setup**.
> Tests are **not required**, and **no test runner, test framework, or test scaffolding**
> is provided by default. Do not add one as part of routine work. Correctness is verified
> through the static checks described in [§8](#8-type-checking--linting). See
> [§14](#14-prohibited-actions).

---

## 2. Repository Map

The repository has exactly two top-level code directories — `apps/` and `packages/` —
plus root documentation and configuration. There is **no third top-level application
directory**.

```
.
├── AGENTS.md                    # This guide — authoritative source of truth
├── README.md                   # Small landing page (overview + links to Setup.md and AGENTS.md)
├── Setup.md                    # Setup_Guide: install & run instructions (human-facing)
├── .gitignore                  # Excludes real .env files, caches, node_modules, lockfile noise
├── apps/
│   ├── api/                    # Backend_App — FastAPI, managed with uv
│   │   ├── app/
│   │   │   ├── __init__.py
│   │   │   ├── main.py         # Creates the FastAPI app; registers routers (no route logic)
│   │   │   ├── config.py       # Typed Settings (Pydantic BaseSettings) from env
│   │   │   ├── db.py           # Engine, SessionLocal factory, get_db dependency
│   │   │   ├── models.py       # SQLAlchemy declarative Base + ORM models (autogenerate target)
│   │   │   ├── schemas.py      # Pydantic v2 request/response models
│   │   │   ├── routes.py       # Application API routes (Example_Endpoint: /items)
│   │   │   └── health.py       # Health_Endpoint (GET /health), kept separate from routes
│   │   ├── alembic/            # Migration environment (env.py, script.py.mako, versions/)
│   │   ├── alembic.ini         # Alembic configuration
│   │   ├── pyproject.toml      # Dependencies + [tool.ruff] / [tool.ty] config (uv)
│   │   ├── uv.lock             # Backend lockfile (committed)
│   │   └── .env.example        # Documents DATABASE_URL with a placeholder value
│   └── web/                    # Frontend_App — Next.js App Router, managed with pnpm
│       ├── app/
│       │   ├── layout.tsx      # Root layout; imports globals.css
│       │   ├── page.tsx        # Example_Page (client component: fetch + loading + error states)
│       │   └── globals.css     # Tailwind base/component/utility layers
│       ├── lib/
│       │   ├── api.ts          # Typed backend helpers (single place for fetch + base URL)
│       │   └── utils.ts        # Shared utilities (e.g., shadcn `cn` helper)
│       ├── components/ui/      # shadcn/ui generated primitives live here
│       ├── components.json     # shadcn/ui configuration
│       ├── package.json        # Scripts + packageManager (pnpm)
│       ├── pnpm-lock.yaml       # Frontend lockfile (committed)
│       ├── tsconfig.json       # TypeScript strict mode
│       ├── tailwind.config.ts  # Tailwind configuration
│       ├── postcss.config.mjs  # PostCSS configuration
│       └── .env.local.example  # Documents NEXT_PUBLIC_API_URL with a placeholder value
└── packages/
    └── shared/                 # Shared_Package placeholder (README only; no app code yet)
        └── README.md
```

---

## 3. Core Principles

1. **Typed end to end.** Every Python function has typed parameters and a typed return
   value; TypeScript runs in strict mode. The "tests" for this template are the static
   checks (type checking, linting, format checking), not a test runner.
2. **Small, flat, explicit.** Predictable layout with one clearly-named file per

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aryanraj2713/Agentic-Project-Template](https://github.com/aryanraj2713/Agentic-Project-Template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
