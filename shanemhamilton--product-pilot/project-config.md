---
trigger: always_on
description: >
---


# Product Pilot — Setup Skill

<!-- Source: https://github.com/shanemhamilton/product-pilot -->
<!-- Product Pilot package version: 2.3 -->

This skill creates and maintains a Product Pilot — a standalone file that gives AI coding agents product awareness: what phase you're in, what milestone is active, what's blocking ship, what metrics matter, what just shipped, what decisions are pending, and what docs to update when done.

**Honest scope note:** This skill does not enforce session-start reads or session-end doc updates — those are the agent's voluntary responsibility. See `TOOLING_GAPS.md` for the harness gaps and what tenants can do to close them.

---

## Mode Detection

Before doing anything, determine which mode to run:

| Signal | Mode | What you'll do |
|--------|------|---------------|
| Pilot exists AND user wants context / is starting a session | **Context** | Read pilot → produce brief orientation |
| Pilot exists AND user mentions changes, completion, or updates | **Update** | Ask what changed → update relevant docs |
| No pilot anywhere in the project | **Setup** | Interview → generate docs |
| User explicitly says "set up" or "bootstrap" | **Setup** | Interview → generate docs (even if pilot exists) |

**Context mode is the most common.** When the pilot exists and the user says "start of session" or asks what to work on, run Context mode — not Setup, not Update.

---

## Context Mode (session start — fastest path)

Run when `PRODUCT_PILOT.md` exists and no setup or update is requested.

If there is no pilot yet, fall through to **Setup mode** instead.

### Step 1: Read and locate
Read `PRODUCT_PILOT.md`. Find the `← ACTIVE` or `<- ACTIVE` milestone.

### Step 2: Synthesize a brief orientation
Report (under 150 words):
- **Phase** — current phase name and goal
- **Active milestone** — name, unchecked `[ ]` tasks, completion signal
- **Top blocker** — from the bulletized "What's Blocking Ship" section (name the blocker, what unblocks it, owner)
- **Recent shipped count** — number of items in the Recent Shipped section since its heading date
- **Metrics at risk** — any metric where Current is significantly below Target (flag these; skip metrics that are "Pre-launch")
- **Pending decisions** — count of rows in Decision Pending; flag any older than 30 days

### Step 3: Freshness checks (run in order, stop at first applicable)
1. Compare `<!-- Last commit captured: HASH -->` against `git rev-parse --short HEAD`. If they differ by more than 10 commits → flag: "The Pilot's last commit anchor is X commits behind HEAD — Recent Shipped is stale. Run the regenerate command before proceeding."
2. If `<!-- Last updated: -->` is more than 30 days old → flag: "The Pilot is X days stale — consider a review after this session."
3. If every task in `← ACTIVE` is checked `[x]` → prompt: "Milestone [X] looks complete. Should we advance `← ACTIVE` to the next milestone?"
4. If PRODUCT_PILOT.md is over 140 lines → note: "The Pilot has grown past 140 lines — operational detail may need moving to a supporting doc."
5. If the Product Docs Index has any `Last reviewed` date more than 30 days old → flag: "[DOC] hasn't been reviewed in over 30 days. Consider updating it this session or scheduling a review."

### Step 4: Connect to current task
If the user's task aligns with the active milestone, say so. If it diverges, note the divergence briefly and offer to log it at session end via Update mode.

**That's it.** No questions, no doc generation. Switch to Update mode if the user wants to log changes.

---

## Update Mode

If a `PRODUCT_PILOT.md` file exists anywhere in the project (check `{PRODUCT_DOCS}`, or search for it), run this section. Use its location as `{PRODUCT_DOCS}`.

**Shortcut:** If this is a periodic review with no specific changes → skip to the Periodic Review checklist at the bottom of this section.

### What changed? (ask the user)

1. Did you complete any milestones or tasks since the last update?
2. Are you moving to a new phase, or staying in the current one?
3. Do you have updated metric values (actual numbers replacing Pre-launch / TBD)?
4. New milestones, features, or priorities to add?
5. Anything new in the competitive landscape?

If nothing changed and the user wants a periodic review → skip to the Periodic Review checklist below.

### What to update

| Change type | Files to update | What to do |
|-------------|----------------|------------|
| Milestone completed | PRODUCT_PILOT.md + ROADMAP.md | Check off items, advance `← ACTIVE` to next milestone. In PRODUCT_PILOT.md, move the completed milestone summary into the Recent Shipped section if Pilot length is above 140 — full detail lives in ROADMAP.md. |
| Phase transition | PRODUCT_PILOT.md + ROADMAP.md | Move `← ACTIVE` to first milestone of new phase, update "Current phase" and the next-transition row |
| Metrics refresh | PRODUCT_PILOT.md + METRICS_AND_OKRS.md | Replace Pre-launch/TBD with actual values in both files; verify both agree |
| New milestone | ROADMAP.md first, then PRODUCT_PILOT.md | Add to correct phase with checklist + completion signal |
| Competitive update | COMPETITIVE_LANDSCAPE.md | Add/update competitor entry; refresh differentiator if positioning shifted |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shanemhamilton/product-pilot](https://github.com/shanemhamilton/product-pilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
