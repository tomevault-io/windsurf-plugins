---
trigger: always_on
description: Intent reference: method selection matrix, sample size guidance, interview techniques, usability testing, survey design, synthesis frameworks. Load when planning or conducting user research.
---


# Research Methods

## Method Selection Matrix

Choosing the right research method depends on what you need to learn, how much time and budget you have, and where you are in the design process. There is no universal "best method" — there's the right method for the question you're asking right now.

### Generative Methods (What should we build?)

**Contextual Inquiry**
- What it is: Observe users in their natural environment while they perform real tasks. Ask questions as they work.
- When to use: Early exploration. You don't understand the problem space well enough to ask good survey questions yet.
- Sample size: 4-8 participants per user segment.
- Time/cost: High. 1-2 hours per session, plus travel. Analysis is intensive.
- What it reveals: Workarounds, environmental constraints, unspoken needs, the gap between what people say and what they do.
- Trade-offs: Small samples, not generalizable, observer effect can alter behavior. But the depth of insight is unmatched.

**Semi-Structured Interviews**
- What it is: One-on-one conversations guided by a topic framework, not a rigid script. Follow interesting threads.
- When to use: When you need to understand motivations, mental models, and experiences in depth. Works at any stage.
- Sample size: 5-8 for pattern identification, 12-20 for saturation (Guest, Bunce & Johnson, 2006).
- Time/cost: Moderate. 45-60 minutes per session. Analysis takes roughly 3x the interview time.
- What it reveals: User motivations, pain points, mental models, emotional responses, workarounds.
- Trade-offs: Self-reported behavior differs from actual behavior. Users are not reliable predictors of their own future actions. But interviews surface the "why" that behavioral data can't.

**Diary Studies**
- What it is: Participants record experiences over time (days to weeks), logging entries when specific events occur.
- When to use: When behavior unfolds over time and can't be observed in a single session. Habit formation, recurring tasks, infrequent events.
- Sample size: 10-15 participants minimum (high dropout expected — recruit 20-30% more).
- Time/cost: Study runs 1-4 weeks. Setup is moderate; analysis is substantial.
- What it reveals: Temporal patterns, context shifts, emotional changes over time, frequency and triggers of behavior.
- Trade-offs: High participant burden, significant dropout, entries are self-reported and often incomplete. But nothing else captures real behavior over time.

**Surveys**
- What it is: Structured questionnaires distributed to a sample population. Quantitative or mixed-method.
- When to use: When you have specific hypotheses to validate, when you need quantitative data at scale, or when you need to measure attitudes/preferences across a population.
- Sample size: 30 minimum for basic statistics, 100+ for segmentation, 400+ for population estimates with reasonable confidence intervals.
- Time/cost: Low per-respondent. Design is the hard part — a bad survey produces confidently wrong data.
- What it reveals: Prevalence of behaviors, attitudes, preferences, and demographics across a population.
- Trade-offs: You only learn what you ask about. Question design bias is pervasive and hard to detect. Response rates are falling across all channels. Self-report limitations apply. But surveys are the only practical way to quantify patterns at scale.

### Evaluative Methods (Is this working?)

**Usability Testing (Moderated)**
- What it is: Observe users attempting tasks with a prototype or product while thinking aloud. Facilitator guides the session.
- When to use: Whenever you have something testable — wireframes, prototypes, live products. The single most valuable evaluative method.
- Sample size: 5 participants catch approximately 85% of major usability issues (Nielsen & Landauer, 1993). Run 5 per round, iterate, test again.
- Time/cost: Moderate. 30-60 minutes per session. Can be done remotely to reduce logistics.
- What it reveals: Where users succeed, fail, hesitate, and get confused. Task completion rates, error frequencies, and recovery patterns.
- Trade-offs: Small sample, not statistically generalizable, facilitator skill matters enormously. But 5 users finding the same problem is a signal, not a sample size issue.

**Usability Testing (Unmoderated)**
- What it is: Users complete tasks independently, recorded by software. No facilitator present.
- When to use: When you need more participants, faster turnaround, or lower cost. Good for benchmarking and A/B comparison.
- Sample size: 10-20 for qualitative patterns, 50+ for quantitative benchmarking.
- Time/cost: Low per-session. Platform costs (UserTesting, Maze, Lookback). No facilitator time.
- What it reveals: Task completion rates, time on task, click paths, self-reported satisfaction.
- Trade-offs: No ability to probe "why." Users may abandon without explanation. Technical issues go unnoticed. Thinking aloud without a facilitator is less natural and less revealing.

**A/B Testing**
- What it is: Randomly assign users to different versions of a design and measure behavioral differences.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ghaida/intent](https://github.com/ghaida/intent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
