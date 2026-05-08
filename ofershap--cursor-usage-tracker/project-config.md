---
trigger: always_on
description: What Cursor API metrics actually mean, what's reliable, what's misleading, and how to interpret the data correctly
---


# Cursor API Data Interpretation Guide

This file documents non-obvious behaviors, gotchas, and correct interpretations of data from the Cursor Enterprise APIs. Built from real-world analysis of team data and cross-referencing with Cursor docs, forum posts, and community tools.

## The Three APIs

Cursor provides three separate APIs, each with its own API key:

1. **Admin API** - Members, daily usage aggregates, spending, usage events, billing groups
2. **Analytics API** - Team-level and per-user analytics (DAU, models, agent edits, tabs, MCP, etc.)
3. **AI Code Tracking API** - Per-commit breakdown of AI vs human code (NOT currently collected by us)

We currently use Admin API + Analytics API. The AI Code Tracking API would give us per-commit granularity but we haven't integrated it yet.

## Critical: What `totalLinesAdded` Actually Measures

The Admin API's `/teams/daily-usage-data` returns `totalLinesAdded` and `totalLinesDeleted`. These are **NOT commit-based metrics**. They track all lines changed in the editor during a day, regardless of source.

This means `totalLinesAdded` includes:
- Lines written by the agent and applied to files
- Lines from tab completions
- Lines from composer suggestions
- Lines the developer typed manually
- Lines from paste operations, refactoring tools, etc.

**`acceptedLinesAdded`** is a subset - it only counts lines from explicitly accepted AI suggestions (tab accepts, composer accepts). But here's the critical gap:

### Agent Mode Does NOT Reliably Count as "Accepted Lines"

When the agent writes code directly to files (which is the default behavior in agent mode), those lines show up in `totalLinesAdded` but may NOT show up in `acceptedLinesAdded`. The agent applies changes directly to files without going through the explicit "suggest -> accept" flow that tab completions and composer use.

Evidence from our data:
- Users with 100% agent mode (0 composer, 0 tabs) show `acceptedLinesAdded` as a small fraction of `totalLinesAdded`
- Users like arielki: 77,830 lines_added, 0 accepted_lines, 171 agent_requests - impossible if agent lines counted as accepted
- The `totalApplies` and `totalAccepts` fields track the agent diff review flow (apply = agent showed a diff, accept = user accepted it), but the actual lines written by the agent to files are broader than just these diffs

**Bottom line: You CANNOT use `acceptedLinesAdded / totalLinesAdded` as "% of code written by AI" for agent-heavy users. It will massively undercount AI contribution.**

### The AI Code Tracking API Has Better Data

The separate `/analytics/ai-code/commits` endpoint (AI Code Tracking API) provides per-commit breakdown:
- `tabLinesAdded` - lines from tab completions
- `composerLinesAdded` - lines from composer/agent
- `nonAiLinesAdded` - manually written lines

These sum to `totalLinesAdded`. This is the correct way to measure AI vs human code. However:
- Only tracks code committed through Cursor's Source Control panel
- Commits made via external git tools (terminal, VS Code, etc.) are NOT tracked
- We don't currently collect this data

## What `totalApplies`, `totalAccepts`, `totalRejects` Mean

These track the **agent diff review flow**:
- `totalApplies` - Number of times the agent presented a code diff to the user
- `totalAccepts` - Number of times the user accepted the diff
- `totalRejects` - Number of times the user explicitly rejected the diff

The gap between `totalApplies` and `totalAccepts + totalRejects` represents diffs that were dismissed without explicit action (user moved on, started a new request, etc.).

Note: In YOLO/auto-apply mode, the agent writes directly to files without showing diffs. These changes may not increment `totalApplies` at all, which means some users will show low applies despite heavy agent usage.

## Request Types and Billing

### Request Fields in Daily Usage
- `agentRequests` - Number of agent mode requests (the primary interaction mode)
- `composerRequests` - Number of composer requests (older feature, being phased out)
- `chatRequests` - Number of chat/ask mode requests
- `usageBasedReqs` - Requests that consumed usage-based billing (cost real money beyond plan)
- `subscriptionIncludedReqs` - Requests covered by the subscription (we don't store this)
- `cmdkUsages` - Inline edit (Cmd+K) usages (we don't store this)
- `bugbotUsages` - Bugbot code review usages (we don't store this)

### How Billing Works
Cursor uses **token-based billing**, not fixed per-request pricing.
For current plan details and pricing, see: https://cursor.com/pricing

Key concepts that affect how we interpret the data:
- Each plan includes a monthly usage allowance per user
- Usage is calculated based on the model's public API token price plus a Cursor surcharge
- Once the included allowance is exhausted, additional usage is billed at the same rate
- `spend_cents` in the spending API = total cost including the included portion
- `included_spend_cents` = the portion covered by the plan
- Actual overage = `spend_cents - included_spend_cents`

### Model Pricing

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ofershap/cursor-usage-tracker](https://github.com/ofershap/cursor-usage-tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
