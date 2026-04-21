---
trigger: always_on
description: DashClaw is a production Next.js 16 governance runtime for AI agents. Copilot should treat the repo's own `CLAUDE.md`, `PROJECT_DETAILS.md`, and `docs/architecture/runtime-api.md` as the source of truth for architecture and API shape. The section below is the canonical design tone — apply it to any UI, marketing, or copy change.
---

# DashClaw — GitHub Copilot Instructions

DashClaw is a production Next.js 16 governance runtime for AI agents. Copilot should treat the repo's own `CLAUDE.md`, `PROJECT_DETAILS.md`, and `docs/architecture/runtime-api.md` as the source of truth for architecture and API shape. The section below is the canonical design tone — apply it to any UI, marketing, or copy change.

## Design Context

### Users

DashClaw has two overlapping audiences who see the same surfaces but read them differently:

- **Primary: AI-agent developers and platform engineers.** They're integrating DashClaw into Claude Code, Claude Managed Agents, LangChain, CrewAI, OpenAI Agents SDK, custom runtimes, or MCP hosts. They live in terminals, read code more naturally than prose, and judge a product in the first 60 seconds by the quality of its README, SDK ergonomics, and error messages. Their job-to-be-done: *"Let my agent act in production without it doing something expensive, irreversible, or embarrassing."*
- **Secondary: governance / compliance / security stakeholders.** They rarely write code but need to audit agent behavior, approve risky actions, and produce evidence. Their job-to-be-done: *"Show me, verifiably, what the agents did, why it was allowed, and who approved anything sensitive."*

The context of use is almost always **professional, focused, and consequential**. Operators open Mission Control because an agent is running in prod. Developers open `/connect` because they're wiring up a live integration. No one is idly browsing. Every pixel should respect that — no decorative filler, no tutorials-for-their-own-sake, no "welcome to your new dashboard" fluff.

### Brand Personality

**Three words: Serious · Precise · Trustworthy.**

DashClaw is a governance runtime. It sits on the critical path between an AI agent's intent and the real world. The visual and verbal tone must match the weight of that position:

- **Voice:** direct, technical, declarative. Short sentences. Verbs like *intercept, enforce, record, verify*. No hype, no exclamation marks, no "unleash your agents."
- **Tone shift:** slightly warmer on marketing pages (landing, `/connect`, docs intros), strictly neutral on operational surfaces (Mission Control, Approvals, Decisions, Policies).
- **Emotional target for operational surfaces:** *quiet confidence — "things are under control."* When an operator opens Mission Control, the room should feel like a calm instrument panel, not an alarm board. Status should be obvious at a glance. Brand orange appears only when attention is actually required, not as decoration.
- **Emotional target for marketing surfaces:** confident competence. We are the adults in the AI-safety room.

### Aesthetic Direction

**Theme:** dark-mode only. Pitch-black canvas (`--color-bg-primary: #0a0a0a`), layered surfaces in near-black (`#111`, `#1a1a1a`, `#222`), with all interaction chrome built from translucent whites. No light-mode variants exist and none should be added without explicit product-level approval.

**Brand color:** `#f97316` (Tailwind `orange-500`). Used as a *signal*, not a fill. Valid uses: the active brand mark, a focused state, a primary CTA, a "this decision needs you" cue, a live-stream pulse. Invalid uses: decorative backgrounds, gradients for their own sake, hero-washes, marketing glow-spam. Accompanying subtle variant `rgba(249, 115, 22, 0.12)` is reserved for hover surfaces and brand-ring glows.

**Typography:** Inter via `next/font/google`, exposed as `var(--font-inter)`. Mono stack is `ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, monospace`. Tabular numerals (`.tabular-nums`) on any data column. Tiny uppercase mono labels with wide tracking are a recurring motif for meta-labels — keep this pattern.

**Component language already established in `app/globals.css` and `tailwind.config.js`:**
- Cards: `var(--color-bg-secondary)` background, 1px `var(--color-border)` stroke, `12px` radius, border-color lift on hover. This is the atom.
- Semantic color tiers are already formalized — *always use tokens, never hardcode hex*. `text-primary / secondary / tertiary / disabled`, `bg-primary / secondary / tertiary / elevated`, `border / border-hover / border-active`, `status-success / warning / error / info` (plus `-subtle` variants).
- Iconography: `lucide-react` only. Size `14–20px` for inline, `24–28px` for hero moments. Never mix icon libraries.
- Elevation via border-color shifts and subtle shadow rings. No chunky drop-shadows.

**Reference direction (what we're aiming at):** the restrained developer-brand register of Vercel, Linear, Resend, Supabase, Datadog, Grafana. Dense-but-readable, technical, calm, branded sparingly.

**Anti-references (what we are explicitly NOT):**
- ❌ **Generic SaaS dashboard** — flat blue accents, Bootstrap-era cards, stock-photo marketing, "empower your team" copy. Undifferentiated and forgettable.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ucsandman/DashClaw](https://github.com/ucsandman/DashClaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
