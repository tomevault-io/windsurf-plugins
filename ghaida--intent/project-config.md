---
trigger: always_on
description: Intent reference: HEART framework, Goal-Signal-Metric mapping, A/B test design, statistical literacy for designers, ethical measurement. Load when defining metrics, designing experiments, or building measurement plans.
---


# Measurement Frameworks

## HEART Framework

Developed by Kerry Rodden, Hilary Hutchinson, and Xin Fu at Google, the HEART framework provides a structured way to define user-centered metrics at any scale — from a single feature to an entire product.

### The Five Dimensions

**Happiness** — Subjective user satisfaction, attitudes, and perceived ease of use. Measured through surveys (CSAT, SUS, NPS), in-app satisfaction prompts, and qualitative feedback.

What it catches that other metrics miss: A product can have high task completion rates but low happiness if the process feels tedious, patronizing, or stressful. Happiness metrics capture the emotional quality of the experience.

What it misses: Happy users aren't necessarily successful users. A product can feel pleasant while failing to deliver actual value. Happiness without task success is entertainment, not utility.

**Engagement** — The depth and frequency of user interaction with the product. Measured through session frequency, session duration, feature usage, actions per session, content consumption.

What it catches: Whether users find the product valuable enough to return to and invest time in. Engagement distinguishes "signed up but never came back" from "uses it daily."

What to watch for: Engagement can be gamed with addictive patterns (infinite scroll, notification spam, variable ratio reinforcement). High engagement driven by manipulation is not success — it's exploitation. Always pair engagement metrics with happiness and task success to distinguish healthy engagement from compulsive engagement.

**Adoption** — New users of a product or feature. Measured through sign-ups, feature activation (first meaningful use, not just account creation), upgrade conversions, new feature discovery.

What it catches: Whether growth is happening and whether new features are being discovered and used. Adoption metrics answer: are we reaching new people, and are they finding value?

What it misses: Adoption without retention is a leaky bucket. High sign-up rates with low day-7 retention mean the acquisition is working but the product isn't.

**Retention** — Users who return over time. Measured through day-1/7/30 retention curves, churn rates, reactivation rates, subscription renewals.

What it catches: Whether the product delivers sustained value. Retention is the strongest proxy for product-market fit — people who come back found something worth returning for.

What to watch for: Retention can be artificially inflated by switching costs, data lock-in, or sunk-cost fallacy rather than genuine value. A user who keeps their subscription because cancellation is difficult is retained but not satisfied. Pair retention with happiness metrics to distinguish healthy retention from captive retention.

**Task Success** — The ability of users to complete their intended tasks efficiently and completely. Measured through task completion rate, time on task, error rate, task abandonment rate, support ticket volume.

What it catches: Whether the product actually works for what users need it to do. Task success is the most direct measure of UX quality — it answers the question "can people use this thing to accomplish their goals?"

What it misses: Efficiency without satisfaction. A user who completes a task but feels frustrated, confused, or disrespected along the way has a different experience than one who completes it smoothly.

### Applying HEART by Feature Type

| Feature type | Primary dimension | Secondary dimensions |
|-------------|------------------|---------------------|
| Core workflow (checkout, file creation, messaging) | Task success | Happiness, Engagement |
| Onboarding | Adoption | Task success, Retention (day-1) |
| Social features | Engagement | Retention, Happiness |
| Content / discovery | Engagement | Happiness, Adoption (new content areas) |
| Settings / configuration | Task success | Happiness |
| Monetization | Adoption (conversion) | Retention (renewal), Happiness |
| Support / help | Task success (resolution) | Happiness (satisfaction) |

---

## Goal-Signal-Metric (GSM) Mapping

HEART tells you what dimensions to measure. GSM tells you how to operationalize each dimension into specific, trackable metrics. Developed as part of the HEART framework at Google.

### The Three Layers

**Goal:** What do you want to accomplish? Articulate in terms of user outcomes, not business outputs. "Users can quickly find relevant content" not "increase pageviews."

**Signal:** What user behavior would indicate the goal is being met (or not met)? Signals are observable behaviors that correlate with goal achievement. "Users successfully find what they're looking for on the first search" is a signal. Signals can be positive (success indicators) or negative (failure indicators).

**Metric:** How do you measure the signal at scale? The metric is the specific, quantifiable measurement that tracks the signal. "Percentage of search sessions where the user clicks a result on the first results page" is a metric.

### GSM Example: Search Feature

| Layer | Content |
|-------|---------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ghaida/intent](https://github.com/ghaida/intent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
