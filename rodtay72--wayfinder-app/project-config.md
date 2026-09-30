---
trigger: always_on
description: Wayfinder is a privacy-safe, modular parent-child reflection and parent emotional development app.
---

# AGENTS.md - Wayfinder App Operating Rules

## Mission

Wayfinder is a privacy-safe, modular parent-child reflection and parent emotional development app.

The journal is a mechanism, not the whole product. Wayfinder helps parents recognise where their Cognition, Affect, and Behaviour (CAB) may not yet be aligned with the emerging needs of the child, then guides them toward awareness, growth, repair, and next action.

The app must always remain deployable and working. Partners may improve visuals, content, UX, and functional features, but every change must preserve authentication, privacy, dashboard data loading, journal save/read behaviour, and existing user records.

## Instruction Priority

If instructions appear to conflict, use this priority order:

1. Preserve privacy, security, authentication, RLS, profile integrity, parent/child identity, journal save/read behaviour, and deployability.
2. Preserve the Wayfinder ALIGN/CAB product canon.
3. Improve UX, UI, content, and functional features within the above boundaries.

Do not silently resolve conflicts. If a requested change would weaken privacy, auth, RLS, identity, existing records, or the ALIGN/CAB philosophy, stop and report the conflict.

## Must Read Before Editing

Before making product, UX, UI, copy, or implementation changes, read:

- docs/WAYFINDER_ALIGN_PRODUCT_CANON.md
- docs/app-architecture-navigation-review.md
- docs/auth-profile-flow.md
- docs/partner-collaboration-and-deployment-rules.md
- docs/profile-cleanup.sql

If `docs/WAYFINDER_ALIGN_PRODUCT_CANON.md` is missing, stop and ask for it to be added before making product/UX/copy changes.

## Product Canon: ALIGN/CAB

Wayfinder is not a child-diagnosis app, behaviour-labelling app, or generic Behaviour -> Advice parenting app.

Wayfinder helps parents recognise where their Cognition, Affect, and Behaviour are not yet aligned with the child's emerging need.

Core pathway:

Behaviour
-> Need
-> Parent CAB
-> Alignment Check
-> Awareness
-> Growth
-> Navigate / Next Action

Official ALIGN mapping:

- A = Awareness
- L = Locate
- I = Integrate
- G = Growth
- N = Navigate

CAB means:

- Cognition: what the parent thinks, assumes, expects, or interprets.
- Affect: what the parent feels emotionally and in the body.
- Behaviour: what the parent does in response.

The phrase "My child did something and I don't know why" must never mean "the child is the problem."

It means:

"The child's behaviour may be revealing a moment where the parent's CAB is not aligned with the child's emerging need."

## Product Framing Rules

Required framing:

- Behaviour is a signal, not a diagnosis.
- The child is not framed as the problem.
- The parent is not blamed or shamed.
- The product guides the parent to notice the possible child need, identify their own CAB, locate misalignment, integrate insight, grow capacity, and navigate a next action or repair step.
- Use cautious language: "may", "might", "possible", "may have", "let's explore".

Do not write or generate language such as:

- "Your child is avoidant."
- "Your child is controlling."
- "Your child is the problem."
- "This behaviour means X" as a certainty.
- "Fix this behaviour by..." as the primary flow.

Prefer language such as:

- "This behaviour may have been trying to solve something."
- "A possible need underneath this behaviour is..."
- "The behaviour may be a signal of..."
- "Let's get curious about what need was emerging."
- "Where might your CAB have moved out of alignment with this need?"

## Parent Development Profile Rules

Do not create fixed parent types, parent diagnoses, or child diagnoses.

If building profile or dashboard insights, create a living developmental reflection profile based on:

- recurring CAB patterns
- recurring child needs noticed
- recurring misalignments
- current growth edge
- emerging strengths
- current practice
- repair habits
- next action patterns

Example acceptable insight:

"Current Alignment Challenge: urgency vs child need for predictability."

Example unacceptable insight:

"You are a controlling parent" or "Your child is oppositional."

## Decode a Moment Build Rules

The planned entry point is:

Title: Decode a Moment
Subtitle: "My child did something and I don't know why."
Description: Explore what the behaviour may have been signalling, what happened in you, and how to respond with more alignment.
Button: Start Decode

When implementing this feature, preserve this flow:

1. Awareness - name what happened without judging the child.
2. Locate - locate the possible child need and the parent's internal response.
3. Integrate - connect child need with parent Cognition, Affect, and Behaviour.
4. Growth - identify the parent capacity being developed.
5. Navigate - choose one next action or repair step.

Suggested saved entry type:

entry_type = "behaviour_decode"

Use existing journal storage in Phase 1 if safe. Do not change Supabase schema unless explicitly approved.

## Core Architecture Rules

Stable identity chain:

Supabase auth user -> profiles.parent_id / Wayfinder ID -> dyads.child_id -> journal_entries

Use:

- Supabase user_id only as technical auth identity.
- parent_id / Wayfinder ID as the app identity.
- child_id as child identity.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rodtay72/wayfinder-app](https://github.com/rodtay72/wayfinder-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
