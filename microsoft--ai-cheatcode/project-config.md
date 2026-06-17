---
trigger: always_on
description: "The Cheat Code" is an internal Microsoft newsletter distributed weekly to the ABS Tech Strategy team (AI Business Solutions). Each issue features a reusable agentic pattern for Copilot Chat, extracted from real customer engagements. The name is a play on "chat" (Copilot Chat) and "cheat" (cheat codes) — the Konami code motif appears subtly in every issue.
---

# Copilot Instructions — The Cheat Code

## What This Is

"The Cheat Code" is an internal Microsoft newsletter distributed weekly to the ABS Tech Strategy team (AI Business Solutions). Each issue features a reusable agentic pattern for Copilot Chat, extracted from real customer engagements. The name is a play on "chat" (Copilot Chat) and "cheat" (cheat codes) — the Konami code motif appears subtly in every issue.

## Architecture

Each issue exists in **four parallel formats**, all produced from the same source content:

1. **HTML email** (`issues/the_cheat_code_issue_NNN.html`) — primary format, 600px max-width, inline CSS, Outlook-compatible
2. **PDF** (`issues/the_cheat_code_issue_NNN.pdf`) — rendered from HTML via Chrome headless, attached to Engage posts
3. **Viva Amplify content** (`viva_amplify/issue_NNN_amplify.md`) — section-by-section content mapped to Amplify web parts
4. **Architecture diagram** (`diagrams/issue_NNN_*_rich.png`) — rendered from companion HTML/CSS files in `diagrams/`
5. **Interactive portal** (`docs/`) — GitHub Pages site with Bento grid portal and step-through diagram walkthroughs. Separate design system from the email newsletter.

The **PRODUCTION_PLAYBOOK.md** is the authoritative reference for all production processes. Read it first when working on anything.

## Hosting & Distribution

- **GitHub Pages archive:** https://aka.ms/the-cheat-code (resolves to `microsoft.github.io/ai-cheatcode`)
- **Repo:** `microsoft/the-cheat-code` (private, GitHub Enterprise)
- **Pages source:** `main` branch, `docs/` folder
- **Workflow:** Edit files → `git push` → GitHub Pages auto-deploys
- The portal (`docs/index.html`) shows all published issues in a Bento grid layout. Interactive walkthroughs live at `docs/interactive/issue-NNN/`.
- Only the `docs/` folder is reader-facing via Pages. Everything else (playbook, series plan, templates) lives in the repo but isn't linked from the portal.
- The `docs/index.html` should only show **published** issues + a "Coming Next" teaser for the next one. Update it as each issue is sent.

## Alternating Cadence: Conceptual → Practical

Starting with Issue #007, the series follows a two-issue arc model:

- **Odd issues (🧠 Conceptual Pattern)**: Names the pattern, explains *why* it exists, frames the architecture, identifies design decisions. Sections: Intro → Pattern Breakdown → Where This Pattern Lands → Quick Tips. No Agent Spotlight.
- **Even issues (🔧 Practical Build)**: Shows *how* to build it in Copilot Studio, step-by-step with components. Sections: Intro (links back to conceptual pair) → Agent Spotlight (the build) → Pattern Breakdown (components in build order) → Try This Now.

The issue info bar carries a type tag: `🧠 CONCEPTUAL PATTERN` or `🔧 PRACTICAL BUILD`.

Issues #001–006 predate this cadence and stand as-is — no rebranding.

## Content Model

Each issue follows a fixed structure defined in `the_cheat_code_template.html`:

- **Header**: Dark navy (#1B1B3A), unique Konami code glyphs per issue (see Symbol Registry in playbook)
- **Intro**: Customer problem → pattern name (2 paragraphs max)
- **Agent Spotlight** (when featuring a specific agent): Attribution, scenario, solution
- **Pattern Breakdown** (always): The core reusable pattern with numbered design decisions
- **Rotating sections**: Quick Tips, Try This Now, Where This Pattern Lands (not all appear in every issue)

### Key Editorial Rules

- Lead with the customer scenario, never write a demo recap
- One agent demo often yields 2–4 standalone patterns — disaggregate them into separate issues
- Every section must answer: "How does this make me faster as a CSA?"
- Full name attribution for the builder in every issue
- Cross-reference related issues when patterns connect — all cross-refs are hyperlinked to the Pages archive

## Current Issue Map (20 issues)

| # | Title | Type | Builder | Date |
|---|-------|------|---------|------|
| 001 | Code-First Agent Delivery | 🔧 | Cristiano Almeida Gonçalves | Mar 23 |
| 002 | Scoped Multi-Source Search | 🧠 | Raghav BN | Mar 31 |
| 003 | Prompt-Chained Triage + Playbooks | 🔧 | Raghav BN | Apr 7 |
| 004 | Secure In-Boundary Processing | 🧠 | Raghav BN | Apr 14 |
| 005 | Human-in-the-Loop Approval Gates | 🧠 | Pete Puustinen | Apr 21 |
| 006 | Meeting-to-Knowledge Pipeline | 🔧 | Pete Puustinen | Apr 28 |
| 007 | Holographic Memory | 🧠 | Tyson Dowd | May 5 |
| 008 | Cross-Project Knowledge Agent | 🔧 | Tyson Dowd | Jun 8 |
| 009 | Adaptive Guardrails | 🧠 | TBD | May 18 |
| 010 | Building Adaptive Guardrails | 🔧 | TBD | May 25 |
| 011 | Multi-Agent Handoff | 🧠 | TBD | Jun 1 |
| 012 | Building Multi-Agent Handoff | 🔧 | TBD | Jun 8 |
| 013 | Persistent Agent Memory | 🧠 | TBD | Jun 15 |
| 014 | Building Persistent Memory | 🔧 | TBD | Jun 22 |
| 015 | Proactive Agent Triggers | 🧠 | TBD | Jun 29 |
| 016 | Building Proactive Agents | 🔧 | TBD | Jul 6 |
| 017 | Agent Evaluation & Trust Signals | 🧠 | TBD | Jul 13 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/ai-cheatcode](https://github.com/microsoft/ai-cheatcode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
