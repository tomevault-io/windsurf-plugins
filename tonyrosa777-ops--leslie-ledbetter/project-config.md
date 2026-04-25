---
trigger: always_on
description: Before writing ANY code — before touching a single file — enter Plan Mode.
---

# CLAUDE.md — Collaborative Insights Project Rules
#
# VARIABLES TO FILL (Task 1 of /prime will complete these):
#   Collaborative Insights     → e.g. "The Enchanted Collective"
#   CollaborativeInsights.com            → e.g. "enchantedmadison.com"
#   spiritual consulting and ascension guidance     → e.g. "luxury glamping and romantic experience property"
#   Texas (online delivery via Zoom)          → e.g. "Madison, Indiana"
#   Demo review: April 10, 2026 — launch date pending sale     → e.g. "June 2026"
#   people experiencing spiritual awakenings who feel confused or overwhelmed by emerging abilities  → e.g. "romantic couples aged 27–45 within 90 min drive"
#   60-minute spiritual consult via Zoom ($100/session)        → e.g. "glamping tents, cottage, day-use hot tub escapes, proposal packages"
#   launch a live, Google-indexed website that generates bookings for spiritual consult sessions          → e.g. "drive direct bookings and own the romantic getaway category in southern Indiana"
#   Calendly (embedded widget, max 8 hrs/day)    → e.g. "Lodgify (launch) / OwnerRez (scale)"
#   ProfessionalService       → e.g. "LodgingBusiness" or "LocalBusiness" or "ProfessionalService"

## Plan Mode Rule
Before writing ANY code — before touching a single file — enter Plan Mode.
Use EnterPlanMode and present a full build plan: what will be built, what files will
be created or modified, what design tokens will be used, what data will flow where.
Get alignment on the plan before the first keystroke. This is non-negotiable.

A wrong plan costs 5 minutes. A wrong build costs 5 hours.

## Subagent Delegation Rule
Any build phase with 3 or more discrete tasks MUST be broken into individual tasks
and delegated to subagents. One subagent per task. Run independent tasks in parallel.

Never execute a multi-task phase as a single monolithic session. This produces slower,
lower-quality output and exhausts context. The correct pattern:
1. Write a task list for the phase
2. Identify which tasks are independent (can run in parallel)
3. Spawn subagents for each task with complete context (file paths, design tokens, exact spec)
4. Collect outputs, verify, integrate

If a task is trivial (under 5 minutes, 1 file), do it inline. If it's substantive, delegate.

## Skill Creation Rule
When you solve a problem in a new way that works — a component pattern, an integration
flow, an animation approach, a build sequence — immediately:
1. Document the exact steps you took (not a summary — the actual implementation)
2. Create a skill file for it using /skill-creator
3. Reference the skill in future builds instead of reinventing

The goal: every non-trivial build decision becomes a reusable skill. Build once, reuse forever.
If a pattern is already in build-log.md, also create a skill so it can be invoked directly.

## Core Law: Research-Backed Decisions Only
Every design decision, copy choice, UX pattern, or technical recommendation
MUST be traceable to market-intelligence.md or initial-business-data.md.
If you cannot cite the research that backs a decision, do not make the
decision — surface it for review.

## Mandatory Pre-Read Protocol
At the start of EVERY session, read in order:
1. CLAUDE.md (this file)
2. progress.md
3. C:\Projects\Optimus Assets\knowledge\build-log.md  ← Cross-project errors + patterns. Check before starting any phase.
4. initial-business-data.md
5. market-intelligence.md
6. design-system.md
7. frontend-design.md
8. website-build-template.md

Never skip this sequence. Never rely on context from a previous session.
Treat each session as if it is your first.

EXCEPTION FOR SUBAGENTS: This protocol applies to the main orchestrator session only.
Subagents spawned via the Agent tool must NOT follow the full 8-file pre-read sequence —
they load only the files listed in their agent file's Required Reading section.
Loading all 8 files in every subagent wastes context before any work begins.

## Agent System Rules
These rules apply whenever the Subagent Delegation Rule triggers agent spawning.

**Agents never spawn agents.** Only orchestrators (workflow commands) spawn agents.
If a subagent needs help, it reports back to the orchestrator — it does not spawn
its own subagents. One level of hierarchy only. This is non-negotiable.

**Agents read files, not summaries.** Every agent gets context by reading known output
files directly (market-intelligence.md, design-system.md, /data/site.ts). The orchestrator
does NOT pass summaries or briefings — it passes file paths. The agent reads the file.

**Agents own their output files exclusively.** No two parallel agents may write to the
same file. Each agent owns exactly one output file or directory. If two tasks share an
output file, they run sequentially — not in parallel.

**Agents checkpoint progress.** After completing each discrete unit of work, the agent
writes a progress note to progress.md. If an agent fails mid-task, the orchestrator
can re-invoke it with "continue from [last checkpoint]" rather than starting over.

**Agent status lifecycle:** Every agent file has a status field: DRAFT → TESTED → VALIDATED.
Only VALIDATED agents run without human review of the output. DRAFT agents always get
output reviewed before proceeding.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tonyrosa777-ops) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
