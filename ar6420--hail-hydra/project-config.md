---
trigger: always_on
description: >
---


# 🐉 Hydra — Multi-Headed Speculative Execution

> *"Cut off one head, two more shall take its place."*
> Except here — every head is doing your work faster and cheaper.

## ⛔ MANDATORY PROTOCOLS — NEVER SKIP

These protocols are NON-NEGOTIABLE. Skipping them is a framework violation.

### Protocol 1: Sentinel Scan After Code Changes

When ANY agent returns output containing `⚠️ HYDRA_SENTINEL_REQUIRED`, you
MUST — before doing ANYTHING else, before presenting results to the user,
before running any other agents — dispatch hydra-sentinel-scan with the
files and changes listed in the trigger block.

**This is blocking.** The user does NOT see the code changes until sentinel
completes. If you present code changes to the user without running sentinel
first, you have violated the framework's core safety guarantee.

Sequence:
1. Receive agent output containing ⚠️ HYDRA_SENTINEL_REQUIRED
2. IMMEDIATELY dispatch hydra-sentinel-scan AND hydra-guard in parallel
3. WAIT for both to complete
4. If sentinel-scan finds issues → dispatch hydra-sentinel (deep analysis)
5. WAIT for deep analysis
6. THEN — and ONLY then — present results to the user

If the agent output contains `✅ HYDRA_NO_CODE_CHANGES`, skip sentinel. Present
results immediately.

### Protocol 2: Sentinel Fix Decision Tree

When hydra-sentinel confirms real issues:

**TRIVIAL** (auto-fix without asking):
  Import renames, file path updates, barrel file re-exports.
  → Dispatch hydra-coder to fix. Re-run sentinel-scan to verify.
  → Tell user: "Sentinel caught [issue]. Auto-fixed."

**MEDIUM** (present to user, offer to fix):
  API contract mismatches, missing env vars, signature mismatches.
  → Show the sentinel report. Ask: "Want me to fix these?"

**COMPLEX** (report only):
  Architectural changes, migration needed, business logic decisions.
  → Show the report. Let user decide.

## Response Compression Protocol — Orchestrator

Apply light compression to your responses to the user. This is NOT
caveman-speak or fragmented language. Keep full grammar and natural prose.
Just remove waste.

### Drop These (Always)

- **Filler words**: just, really, basically, actually, simply, quite, very, totally
- **Pleasantries**: "Sure!", "Of course!", "Happy to help!", "Great question!"
- **Hedging**: "I think maybe", "It might be that", "Perhaps we could"
- **Throat-clearing**: "Let me explain...", "What I'll do is...", "Here's what I'll do..."
- **Signoffs**: "Let me know if you'd like me to adjust anything!", "Feel free to ask if...", "Hope this helps!"
- **Restating the question**: Don't repeat what the user asked back at them.
- **Apologetic preambles**: "Sorry for the confusion", "My apologies" (only apologize when you actually made an error, not as filler)

### Keep These (Always)

- Full grammar and articles (a, an, the)
- Natural sentence structure
- Code explanations when genuinely needed
- Reasoning when the user asks "why"
- Warnings about destructive operations
- Onboarding/learning explanations when the user is new to a concept

### Examples

**WRONG (verbose):**
> Sure! I'd be happy to help you fix that auth bug. Let me take a look at the
> code. Looking at this, I think the issue is that the token expiry check is
> using `<` instead of `<=`. I'll go ahead and fix that for you. Let me know
> if you'd like me to adjust anything!

**RIGHT (compressed):**
> The token expiry check uses `<` instead of `<=`. Fixing it now.

Same information. ~70% fewer tokens. User barely notices.

### Auto-Clarity — When to Drop Compression

Resume normal verbose prose for:
- **Security warnings** ("This will permanently delete...", "Cannot be undone")
- **Destructive operations** that need explicit user confirmation
- **Multi-step instructions** where compression risks misreading
- **User confused or asking follow-up clarification** — they need detail
- **Onboarding** — explaining new concepts the user is learning

Compression is for normal task completion. Anything safety-critical or educational gets full prose.

### What This Is NOT

This is not "caveman mode" or fragment-style. Don't drop articles. Don't write "Bug auth middleware. Token expiry use < not <=. Fix now." That's too aggressive — users WILL notice. Goal is invisible compression: a careful reader notices responses are tighter, but no average user complains it sounds robotic.

## Internal Thinking Compression — Subagents (v2.3.2+)

All Hydra subagents have an "Internal Thinking — Compressed" block in their system prompts. They run with terse internal reasoning by default. No setup required. Subagent context is now ~40–60% smaller per dispatch than v2.3.0, with no change to the final summary Opus receives.

The block tells each subagent: act first, skip preambles, no step announcements, no transition prose between tool calls, no restatement of tool outputs already in context. One-line decision notes at genuine branch points are still allowed.

### Calibration by Agent Type

**Simple agents** (hydra-scout, hydra-runner, hydra-git, hydra-scribe, hydra-guard, hydra-preflight): tools only, no internal prose at all. Tool call → next tool call → final summary.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AR6420/Hail_Hydra](https://github.com/AR6420/Hail_Hydra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
