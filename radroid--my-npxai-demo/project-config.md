---
trigger: always_on
description: Rules for agents working in this repo. Only essential, repeatedly-relevant rules belong here. Project context lives in `README.md`.
---

# Agent Rules

Rules for agents working in this repo. Only essential, repeatedly-relevant rules belong here. Project context lives in `README.md`.

## Rules

- Use **Bun** as the package manager. Never run `npm`, `yarn`, or `pnpm`. Use `bun install`, `bun add`, `bun dev`, `bunx`.
- **Assume the dev server is already running.** Do not start, restart, or kill it.
- **Prefer CLIs over dashboards or ad-hoc drivers for external services.** For databases and deployment platforms (Supabase, Cloudflare, Vercel, etc.), use the official CLI (`supabase`, `wrangler`, `vercel`) for schema changes, migrations, env management, and deploys. Do not propose new npm drivers (`postgres.js`, `pg`) or "paste into the dashboard SQL editor" as the primary path. The CLIs give us versioned, replayable, scriptable operations; dashboard-paste is a last resort when no CLI command exists.
- **Every frontend change must work in BOTH light and dark themes.** The app ships both modes (`next-themes` + the shadcn `ThemeToggle`). Whenever you touch colors/backgrounds/borders/text:
  1. **Prefer canonical Tailwind utilities over arbitrary `var()` values.** All semantic tokens are mapped in `app/globals.css` under `@theme inline`, so use the generated utility instead of the escape hatch:
     | Token | Canonical utility | Use instead of |
     |---|---|---|
     | `--bg` | `bg-bg`, `to-bg`, `from-bg`, `via-bg` | `bg-[var(--bg)]` |
     | `--surface` | `bg-surface` | `bg-[var(--surface)]` |
     | `--surface-2` | `bg-surface-2` | `bg-[var(--surface-2)]` |
     | `--text` | `text-fg` | `text-[var(--text)]` |
     | `--text-muted` | `text-fg-muted` | `text-[var(--text-muted)]` |
     | `--accent-brand` | `bg-brand`, `text-brand`, `ring-brand`, `border-brand` (opacity modifiers work: `ring-brand/30`) | `bg-[var(--accent-brand)]` |
     | `--accent-brand-hover` | `bg-brand-hover` (typically under `hover:`) | `bg-[var(--accent-brand-hover)]` |
     | `--requirement` | `bg-requirement`, `text-requirement`, `border-requirement` | `text-[var(--requirement)]` |
     | `--guidance` | `bg-guidance`, `text-guidance`, `border-guidance` | `text-[var(--guidance)]` |
     | `--success` | `bg-success`, `text-success`, `border-success` | `text-[var(--success)]` |
     | `--warning` | `bg-warning`, `text-warning`, `border-warning` | `text-[var(--warning)]` |
     | `--danger` | `bg-danger`, `text-danger`, `border-danger`, `ring-danger` | `text-[var(--danger)]` |
     | `--border` | `border-border` | `border-[var(--border)]` |
     | `--border-strong` | `border-border-strong` | `border-[var(--border-strong)]` |
     | `--accent` | `ring-accent` (shadcn alias; our semantic brand focus ring is `ring-brand`) | `ring-[var(--accent)]` |

     Never hardcoded hex values. Light values live in `:root`, dark values in `.dark` — the canonical utilities resolve through `var(--…)` so dark mode continues to work.
  2. Mentally verify each element in BOTH palettes before shipping. Special danger zone: white/colored text on colored backgrounds — these flip catastrophically if a token doesn't resolve or the contrast doesn't survive the theme swap.
  3. **Tailwind 4 arbitrary-value syntax gotcha (for the rare case you do need an escape hatch).** Write `bg-[var(--foo)]` (explicit `var()`) or `bg-(--foo)` (Tailwind 4 parentheses shorthand). Do NOT use the bracket-only form `bg-[--foo]` — Tailwind 4 treats that as a literal token and the background silently fails to render, leaving white-on-white text in light mode. Same rule for `text-`, `border-`, `ring-`, `fill-`, `stroke-` arbitrary-value utilities.

## OVERNIGHT.md (sprint state, gitignored)

When Raj kicks off a multi-stage overnight or multi-day sprint, agent-side state (the brief verbatim, rule overrides, decisions taken autonomously, branch plan, metrics-in-flight) lives in `OVERNIGHT.md` at the repo root. It is gitignored — safe for orchestration notes that should not pollute the committed history.

On any session start: if `OVERNIGHT.md` exists, read it first. It captures what Raj said, what has been done, what remains, and any autonomous decisions the agent made. When committed files (PLAN.md, TODO.md, REFACTOR_METRICS.md) disagree with OVERNIGHT.md, trust OVERNIGHT for agent-side intent and the committed files for shipped state.

## PLAN.md + TODO.md workflow

This repo uses a shared-context system between the human (Raj) and agents. Two files coordinate work:

- **`PLAN.md`** — single source of truth for mission, current phase, stack, decisions, scope guardrails, and appendices (SQL schemas, scraping targets, question batteries, scripts). Source of alignment between human and agents.
- **`TODO.md`** — actionable checklist split into **👤 For Humans** and **🤖 For Agents** sections, organized by phase. Aligned with `PLAN.md`.

### At the start of every session
1. Read `PLAN.md` to learn the current phase, decisions, and scope guardrails.
2. Read `TODO.md` — specifically the **🤖 For Agents** section for the current phase — to know what's outstanding.
3. If the user's request is already captured as an agent task, proceed with it. If it belongs in the Humans section, surface that to the user instead of doing it yourself.

### While working

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [radroid/my-npxai-demo](https://github.com/radroid/my-npxai-demo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
