---
trigger: always_on
description: If Supermemory, OpenRouter, Trail of Bits APIs, or Playwright MCP timeout or fail:
---

# Global Rules

## Graceful Degradation Policy

If Supermemory, OpenRouter, Trail of Bits APIs, or Playwright MCP timeout or fail:
1. Log error to console
2. Notify user: "[Service] unavailable — proceeding with local context only."
3. Continue without the failed service. Do NOT retry in a loop.
4. On next successful connection, sync any pending state.

---

## Rule 1: Council Governance (MANDATORY)

The council skill (`~/.claude/skills/council/`) provides multi-model second opinions via OpenRouter. All sub-rules below are NON-NEGOTIABLE.

### 1a: Bug Fix Guardrail — SELF-MONITOR REQUIRED
Track your own bug fix attempts within each conversation. A "failed attempt" = the fix was applied but the error persists, a test still fails, or the user says it didn't work.

**After the 2nd failed attempt at the same bug, you MUST STOP and do ALL of this before trying a 3rd fix:**

1. Announce to the user: "Invoking the council — 2 fix attempts have failed. Consulting Codex, Gemini, and Kimi before trying again."
2. Run this command (fill in the bracketed sections with actual context from the conversation):
```bash
python ~/.claude/skills/council/scripts/council.py consult --fan-out --context "BUG: [describe the bug/error]

ATTEMPT 1: [what was changed + what happened]
ATTEMPT 2: [what was changed + what happened]

ERROR OUTPUT: [paste the actual error]

RELEVANT CODE:
[paste the relevant code snippet]

What is the root cause and what fix do you recommend?"
```
3. Present all model responses clearly labeled
4. Synthesize: state where you agree/disagree with each model, then propose a revised approach combining the best insights
5. Only THEN attempt the 3rd fix using the collective insight
6. **Log the consultation** — evaluate each model's response and log the assessment:
```bash
python ~/.claude/skills/council/scripts/council.py log \
  --project-dir "$(pwd)" \
  --type bug_fix \
  --bug-type "[classify: runtime_error|type_error|logic_error|import_error|config_error|api_error|ui_bug|state_bug|async_error|test_failure|build_error|other]" \
  --context "[brief bug description]" \
  --attempt 3 \
  --models '[{"model_id":"openai/gpt-5.3-codex","recommendation_summary":"[1-2 sentence summary]","verdict":"[valid|partial|invalid]","adopted":[true|false],"strengths":["[specific strength]"],"weaknesses":["[specific weakness]"]},{"model_id":"google/gemini-3.1-pro-preview","recommendation_summary":"[1-2 sentence summary]","verdict":"[valid|partial|invalid]","adopted":[true|false],"strengths":["[specific]"],"weaknesses":["[specific]"]},{"model_id":"moonshotai/kimi-k2.5","recommendation_summary":"[1-2 sentence summary]","verdict":"[valid|partial|invalid]","adopted":[true|false],"strengths":["[specific]"],"weaknesses":["[specific]"]}]' \
  --outcome pending
```
Note the returned UUID — you will need it for the outcome update.
7. **After the fix resolves or fails**, update the outcome:
```bash
python ~/.claude/skills/council/scripts/council.py log \
  --project-dir "$(pwd)" \
  --type bug_fix \
  --bug-type "[same as step 6]" \
  --context "outcome update" \
  --update-id "[UUID from step 6]" \
  --models '[]' \
  --outcome [resolved|partially_resolved|unresolved] \
  --outcome-notes "[what happened]"
```

### 1b: Plan Validation — AUTO-CONSULT BEFORE EXITING PLAN MODE
When in plan mode and you have drafted a complete plan, you MUST do ALL of this BEFORE calling ExitPlanMode:

1. Announce: "Validating this plan with the council before finalizing."
2. Run this command (fill in the bracketed sections):
```bash
python ~/.claude/skills/council/scripts/council.py consult --fan-out --context "TASK: [user's request in 1-2 sentences]

CODEBASE CONTEXT: [key findings from exploration]

PROPOSED PLAN:
[the complete plan you drafted]

Review this plan. Is this the best approach? What would you change, add, or do differently? Flag any risks or missed edge cases."
```
3. Present a summary of council feedback (agreements, disagreements, suggestions)
4. Revise the plan incorporating the best feedback
5. **Log the consultation** — evaluate each model's feedback and log:
```bash
python ~/.claude/skills/council/scripts/council.py log \
  --project-dir "$(pwd)" \
  --type plan_validation \
  --context "[brief plan description]" \
  --models '[{"model_id":"openai/gpt-5.3-codex","recommendation_summary":"[1-2 sentence summary]","verdict":"[valid|partial|invalid]","adopted":[true|false],"strengths":["[specific]"],"weaknesses":["[specific]"]},{"model_id":"google/gemini-3.1-pro-preview","recommendation_summary":"[1-2 sentence summary]","verdict":"[valid|partial|invalid]","adopted":[true|false],"strengths":["[specific]"],"weaknesses":["[specific]"]},{"model_id":"moonshotai/kimi-k2.5","recommendation_summary":"[1-2 sentence summary]","verdict":"[valid|partial|invalid]","adopted":[true|false],"strengths":["[specific]"],"weaknesses":["[specific]"]}]' \
  --outcome [plan_improved|plan_unchanged]
```
6. Write the revised plan to the plan file with a "Council Review" section noting what changed
7. THEN call ExitPlanMode

### 1c: Historical Insights — LEARN FROM HISTORY
Before evaluating council responses (in both 1a and 1b), query supermemory for historical model performance insights:

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FC-FUZ/claude-code-governance](https://github.com/FC-FUZ/claude-code-governance) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
