---
trigger: always_on
description: Find launchable D2C product ideas for the Indian market, tuned to the user's taste, capital ceiling, distribution skills, and category preferences. Use when the user asks for D2C product ideas in India, wants to brainstorm consumer brand opportunities in India, wants to validate an Indian D2C idea, or wants to change their preferences (categories, capital, distribution channels, hard excludes) mid-conversation. Requires a one-time setup (~3 minutes) the first time it runs.
---


# D2C Idea Finder

Runs a multi-stage agentic pipeline that surfaces 5 launchable Indian D2C product ideas, ranked and tagged (passed/flagged) against the user's taste profile. Five collectors pull signals: Indian marketplaces (Amazon.in, Flipkart, Nykaa, Myntra, Lenskart, FirstCry, and ~12 other specialist sites including quick commerce), per-category exploration queries (Reddit subs like r/IndianTech, r/AsianBeauty, r/IndianFood — plus YouTube reviews and Quora for categories where those communities are strong), 4 rotating US D2C trade publications (Modern Retail, Retail Brew, The Fascination, Exploding Topics), cross-platform Reddit + Amazon US for arbitrage signal, and Google Trends for US-vs-India interest gaps. Instagram is used only weakly in competitor enrichment (not in collection).

## How this skill works

The skill has three entry points:

1. **First-time setup** — captures the user's taste profile (categories, distribution channels, hard excludes, capital ceiling, Exa + Anthropic API keys). Run only once per user.
2. **Find ideas** — runs the full pipeline and returns 5 ranked ideas as markdown (some may be flagged with reasons like "over budget" or "competition too high" — flagged ideas still show, just below the passing ones). Takes 12-18 minutes (signal enrichment and idea generation are the dominant stages; competitor enrichment runs Exa searches in parallel with a concurrency cap).
3. **Update profile** — when the user wants to change capital ceiling, categories, excludes, or distribution channels mid-conversation, applies the change to their profile.

The skill folder is at `~/.claude/skills/india-d2c/`. User data (profile, API key, ratings, state DB) lives in `<skill-folder>/user_data/` and is gitignored.

## When to invoke

**Invoke this skill when the user:**
- Asks "find me d2c ideas in India," "what should I build in Indian D2C," "any consumer brand opportunities in India," "d2c product ideas for India" or similar
- Says "skip pet forever," "bump my capital to 15L," "I'm great at paid ads" (these are profile changes — see Step 3b)
- Asks to "set up" or "configure" the India D2C idea finder

**Do NOT invoke for** general taste comments like "I like #3" or "the wellness angle is interesting" — just acknowledge those conversationally and ask if they want it translated into a profile change.

**Do not invoke for:**
- General market research that isn't about launching a D2C brand
- SaaS or B2B idea generation (different skill)
- Country-specific D2C outside India (this skill is India-focused)

## Step 1 — Check for first-time setup (MANDATORY)

**You MUST run this Bash check before doing ANYTHING else when the skill is invoked.** Do not skip it. Do not assume setup is needed without verifying. Do not tell the user "you don't have settings yet" without first running this command and getting `SETUP_NEEDED`:

```bash
test -f ~/.claude/skills/india-d2c/user_data/profile.yaml && echo "SETUP_DONE" || echo "SETUP_NEEDED"
```

**Read the output literally:**
- Output is `SETUP_DONE` → profile exists, skip to Step 3 (Find ideas) OR Step 3b (Update profile / show settings) depending on what the user asked for. Do NOT trigger onboarding.
- Output is `SETUP_NEEDED` → profile.yaml is missing, walk the user through Step 2 (onboarding).

If you skip this check and improvise an answer based on assumption, you'll incorrectly tell returning users that their settings are gone — which they aren't. This is a confidence-failure pattern we've seen and need to prevent.

### Slash-command note

The skill is invoked when the user's intent matches the skill description (e.g. "find me d2c ideas", "show me my d2c settings"). There is **no `/india-d2c` slash command** for users to type. Do not tell users to "run `/india-d2c`" — that command doesn't exist. If they want to trigger setup explicitly, the right phrasing is *"set up the india-d2c skill"* or *"configure my d2c idea finder"*.

## Step 2 — Setup conversation (first run only)

Tell the user: *"This is your first time running the India D2C Idea Finder. Quick 3-minute setup to tune it to your taste, then we'll find ideas."*

**IMPORTANT — Do NOT use the AskUserQuestion tool for any onboarding question.** Several of these questions have more than 4 options, which AskUserQuestion does not support, and it will throw a visible error. Always present questions as numbered chat text and ask the user to reply with numbers (e.g. "1, 3, 5") or names (e.g. "beauty, food, fitness"). Map their reply back to canonical keys yourself.

Then ask these questions, **one at a time, in order**. Wait for each answer before moving on.

### Q1. Categories of interest

> Which D2C categories are you interested in exploring? Pick as many as you like — recommend 2-5 for good idea diversity. I'll only surface ideas in these.

Present these 16 options:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sanketn/india-d2c](https://github.com/sanketn/india-d2c) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
