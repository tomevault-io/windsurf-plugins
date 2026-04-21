---
trigger: always_on
description: Generates a context-appropriate persona for product or marketing workflows with explicit assumptions, confidence, and evidence trail. Use when shaping artifact perspective, stress-testing decisions, or framing PM/GTM decisions.
---

# PM-Skills

> Open source Product Management skills for AI agents

This repository contains professional PM skills organized by the Triple Diamond framework plus foundation capabilities. Each skill helps AI agents produce high-quality PM artifacts.

## Skills

### Foundation Classification

#### lean-canvas
**Path:** `skills/foundation-lean-canvas/SKILL.md`

Generate a one-page lean canvas across nine interlocking blocks (problem, customer, UVP, solution, channels, revenue, cost, metrics, unfair advantage). Optional visual mode writes a self-contained HTML file rendering the canonical Maurya layout for sharing, printing, or presentation.

#### persona
**Path:** `skills/foundation-persona/SKILL.md`

Generates a context-appropriate persona for product or marketing workflows with explicit assumptions, confidence, and evidence trail. Use when shaping artifact perspective, stress-testing decisions, or framing PM/GTM decisions.

#### meeting-agenda
**Path:** `skills/foundation-meeting-agenda/SKILL.md`

Produces an attendee-facing agenda with time-boxed topics, type tags (Discussion / Decision / Information / Working), owners, and attendee prep. Supports ten meeting-type variants (standup, planning, review, decision-making, brainstorm, 1-on-1, stakeholder-review, project-kickoff, working-session, exec-briefing). Member of the Meeting Skills Family.

#### meeting-brief
**Path:** `skills/foundation-meeting-brief/SKILL.md`

Produces a private strategic prep document with stakes, stakeholder reads, ranked desired outcomes, key messages, anticipated Q&A, risks, and specific asks. Member of the Meeting Skills Family. Distinct from meeting-agenda because the brief is private by default, not shared with attendees.

#### meeting-recap
**Path:** `skills/foundation-meeting-recap/SKILL.md`

Produces a topic-segmented post-meeting summary with decisions bold-flagged inline and actions captured per topic (plus consolidated action view). Auto-populates topic skeleton from a sibling meeting-agenda when available. Accepts transcripts from any tool. Member of the Meeting Skills Family.

#### meeting-synthesize
**Path:** `skills/foundation-meeting-synthesize/SKILL.md`

Cross-meeting archaeology skill. Consumes multiple meeting recaps over a period and surfaces patterns invisible in any single meeting: decision evolution, stakeholder position shifts, stalled threads, contradictions. Produces plain-text timeline, themes, prioritized follow-ups. Member of the Meeting Skills Family.

#### stakeholder-update
**Path:** `skills/foundation-stakeholder-update/SKILL.md`

Produces async communication to stakeholders from a meeting recap. Supports five channel variants (slack, teams, email, notion, exec-memo) and five audience variants (engineering, design, leadership, customer-facing, mixed). Surfaces primary CTA up front, logs technical-to-business translations, detects thread continuation. Member of the Meeting Skills Family.

> Meeting skills share a contract at `docs/reference/skill-families/meeting-skills-contract.md` governing frontmatter, file naming, go-mode behavior, and universal output requirements. Enforced by `scripts/validate-meeting-skills-family.sh`.

---

### Discover Phase

#### competitive-analysis
**Path:** `skills/discover-competitive-analysis/SKILL.md`

Creates a structured competitive analysis comparing features, positioning, and strategy across competitors. Use when entering a market, planning differentiation, or understanding the competitive landscape.

#### interview-synthesis
**Path:** `skills/discover-interview-synthesis/SKILL.md`

Synthesizes user research interviews into actionable insights, patterns, and recommendations. Use after conducting user interviews, customer calls, or usability sessions to extract and communicate findings.

#### stakeholder-summary
**Path:** `skills/discover-stakeholder-summary/SKILL.md`

Documents stakeholder needs, concerns, and influence for a project or initiative. Use when starting projects, managing complex stakeholder relationships, or ensuring alignment across organizational boundaries.

---

### Define Phase

#### hypothesis
**Path:** `skills/define-hypothesis/SKILL.md`

Defines a testable hypothesis with clear success metrics and validation approach. Use when forming assumptions to test, designing experiments, or aligning team on what success looks like.

#### jtbd-canvas
**Path:** `skills/define-jtbd-canvas/SKILL.md`

Creates a Jobs to be Done canvas capturing the functional, emotional, and social dimensions of a customer job. Use when deeply understanding customer motivations, designing for jobs, or reframing product positioning.

#### opportunity-tree
**Path:** `skills/define-opportunity-tree/SKILL.md`

Creates an opportunity solution tree mapping desired outcomes to opportunities and potential solutions. Use for outcome-driven product discovery, prioritization, or communicating product strategy.

#### problem-statement
**Path:** `skills/define-problem-statement/SKILL.md`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [product-on-purpose/pm-skills](https://github.com/product-on-purpose/pm-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
