---
trigger: always_on
description: Defines and tracks UX success through metrics, measurement frameworks, and experimentation. Part of the Intent design strategy system. Connects design decisions to observable evidence — did the thing we built actually help? Guards against measurement becoming manipulation. Trigger when: defining success metrics, designing A/B tests, building measurement frameworks, analyzing funnels, reviewing metric dashboards, questioning whether the right things are being measured, or when someone says "how d
---



# Measure — Define and Track Success

## Overview

If you can't define success, you can't design for it. And if you measure the wrong thing, you'll optimize for the wrong outcome.

UX measurement connects design decisions to observable evidence — did the thing we built actually help? This skill defines what to measure, how to measure it, and how to make decisions from what you learn. It bridges the gap between "we shipped it" and "it worked."

But measurement is not neutral. Every metric you choose shapes what gets optimized. Measure time-on-site and you'll get infinite scroll. Measure clicks and you'll get clickbait. Measure conversion and you'll get dark patterns — unless you also measure what those metrics cost the user. This skill guards against measurement becoming manipulation, ensuring that metrics incentivize genuine value, not engineered engagement.

**When to activate this skill:** Defining success criteria for a new feature, designing experiments, building measurement frameworks, analyzing funnel performance, reviewing whether existing metrics are measuring the right things, or anytime "the numbers look good" but the experience feels wrong.

---

## Skill family

Measure works alongside the full Intent skill system:

- **`/strategize`**: Their hypotheses need measurable success criteria. Every strategic bet should connect to a metric that tells you whether the bet paid off. `/strategize` defines "we believe X"; `/measure` defines "we'll know X is true when Y." When metrics contradict a strategic assumption, measure loops back to reopen strategy — with guardrails (see "When measurement points back to strategy" below).
- **`/investigate`**: Qualitative research complements quantitative measurement. When the numbers say users drop off at step 3, investigate tells you why. When satisfaction scores drop after a redesign, investigate interviews users to understand the experience behind the number. Never make major design decisions from metrics alone.
- **`/evaluate`**: UX assessment produces scores and findings that inform what to measure. Evaluation identifies usability issues; measurement tracks whether fixes actually resolved them.
- **`/specify`**: Test plans and success metrics go into handoff specs. Every feature spec should include what success looks like and how to measure it, so engineering can instrument accordingly.
- **`/philosopher`**: A cross-cutting cognitive mode for questioning your metrics before they become targets. Invoke when: a metric feels too easy to game, the dashboard looks green but users are complaining, you're not sure whether you're measuring user success or business extraction, or you need the question: "What if measuring this changes the behavior we're trying to measure?"

---

## Core capabilities

### 1. Metric selection: HEART framework

Google's HEART framework provides a structured approach to selecting UX metrics. Apply it per feature, not globally — different features need different metrics.

**Happiness — subjective satisfaction:**
- NPS (Net Promoter Score): likelihood to recommend, 0-10 scale. Blunt but useful for trending.
- CSAT (Customer Satisfaction): satisfaction with specific interaction, usually 1-5 scale. More actionable than NPS for feature-level decisions.
- SUS (System Usability Scale): 10-question standardized usability questionnaire. Good for benchmarking across releases.
- Custom surveys: specific questions tied to specific features. "How easy was it to find what you were looking for?" is more useful than "How satisfied are you?"

**Engagement — behavioral depth:**
- Frequency: how often users return (daily, weekly, monthly active users)
- Intensity: depth of usage per session (features used, content consumed, actions taken)
- Breadth: how many features a user touches (adoption breadth, not just depth)
- Recency: when was the last interaction (early warning for churn)

**Adoption — new usage:**
- New user activation: percentage completing key onboarding milestones
- Feature adoption: percentage of eligible users who try a new feature
- Onboarding completion: funnel through first-use experience
- Time-to-value: how quickly new users reach their first meaningful outcome

**Retention — continued usage:**
- Return rate: D1, D7, D30 retention (percentage returning after 1, 7, 30 days)
- Churn rate: percentage of users who stop using the product in a period
- Reactivation: users who left and came back (what brought them back?)
- Cohort retention: retention curves by signup cohort (are newer users retaining better?)

**Task success — effectiveness:**
- Completion rate: percentage of users who finish the task they started
- Error rate: percentage of attempts that result in errors
- Time-on-task: how long the task takes (shorter is usually better, but not always)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ghaida/intent](https://github.com/ghaida/intent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
