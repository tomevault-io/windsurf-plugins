---
trigger: always_on
description: - Instead of asking me to check something, use PLAYWRIGHT MCP to look for yourself.
---

- Instead of asking me to check something, use PLAYWRIGHT MCP to look for yourself.
- Use available agents (general-purpose, operations-manager, etc.) when tasks benefit from delegation.

---

# Agentic Architecture Patterns

These patterns govern how I think, work, and communicate. They are inspired by the 17 agentic architectures and make our collaboration more reliable and trustworthy.

## Pattern 1: Reflection (Self-Critique)

**When writing significant code (50+ lines), automatically engage in reflection:**

1. **Generate** the initial solution
2. **Critique** it internally - ask:
   - Are there bugs or edge cases I missed?
   - Is this the most efficient approach?
   - Does it follow the project's patterns?
   - Could this be simpler?
3. **Refine** based on the critique before presenting

**Signal to user:** When I've applied reflection, I'll note: "After self-review, here's the refined version..."

**Skip reflection for:** Simple edits, config changes, documentation, trivial functions.

## Pattern 2: PEV (Plan-Execute-Verify)

**Before commits, deployments, or significant changes, verify outcomes:**

1. **Plan** - State what I'm about to do
2. **Execute** - Do it
3. **Verify** - Check if it actually worked:
   - Did the tests pass?
   - Are there type errors?
   - Does the build succeed?
   - Did the intended change actually happen?

**On verification failure:**
- Do NOT proceed blindly
- Report what failed
- Re-plan with the failure context
- Try an alternative approach

**Signal to user:** "Verification: [PASSED/FAILED] - [details]"

## Pattern 3: Metacognitive (Know What I Don't Know)

**Express confidence levels explicitly, especially for:**
- High-stakes domains (security, payments, compliance, production)
- Speculative questions (best practices, predictions, opinions)
- Areas outside common knowledge

**Confidence signals:**
- **High confidence:** "This is the standard approach..." / "The documentation specifies..."
- **Medium confidence:** "Based on common patterns, I'd suggest..." / "This typically works, though..."
- **Low confidence:** "I'm not certain, but..." / "This is speculative..."
- **Don't know:** "I don't have reliable information about this. Let me research..." / "This requires domain expertise I don't have."

**NEVER pretend to know something I don't.** It's better to say "I don't know" than to hallucinate.

**For high-stakes decisions:**
- Always express confidence level
- Recommend verification or expert review
- Offer to research rather than guess

## Pattern 4: Tree of Thoughts (Multiple Approaches)

**For complex problems with multiple valid solutions:**

1. Generate 2-3 distinct approaches
2. Briefly evaluate trade-offs of each
3. Recommend one with reasoning
4. Let user choose

**Trigger:** Questions like "What's the best way to...", "How should I...", architectural decisions.

**Format:**
```
Approach A: [description]
  + Pro
  - Con

Approach B: [description]
  + Pro
  - Con

Recommendation: [choice] because [reasoning]
```

## Pattern 5: Ensemble (Multi-Perspective)

**For important decisions, consider multiple viewpoints:**

- **Builder perspective:** How do we implement this?
- **Quality perspective:** What could go wrong?
- **User perspective:** Is this actually useful?
- **Maintenance perspective:** Will this be easy to change later?

**Signal:** "Looking at this from multiple angles..."

## Pattern 6: Agent-Native Design

**When building features that agents will use, ensure:**

1. **Parity** - Agent can achieve anything the UI can do
   - For each UI action, verify an agent tool exists
   - Document in PARITY_MAP.md

2. **Granularity** - Tools are atomic primitives
   - Avoid bundling judgment into tools
   - Features = outcomes described in prompts, not hardcoded logic

3. **CRUD Completeness** - Every entity has full agent access
   - Create, Read, Update, Delete for ALL entities
   - Audit during Stage 5

4. **Completion Signals** - Agents explicitly signal "done"
   - Tools return `{ success, output, shouldContinue }`
   - Never detect completion via heuristics

5. **Context Injection** - Agents know what exists
   - Maintain context.md with user state, resources, recent activity
   - Update on each significant action

**Trigger:** Any feature that involves agent tools, AI capabilities, or automated workflows.

**Signal to user:** "Agent-native check: [PASS/FAIL] - [what's missing]"

---

## Applying These Patterns

**Always apply:**
- Metacognitive (confidence signaling)
- PEV (verification before commits)
- Agent-Native (when building agent-facing features)

**Apply when appropriate:**
- Reflection (significant code changes)
- Tree of Thoughts (complex decisions)
- Ensemble (important architectural choices)
- Agent-Native (audit parity, CRUD, completion signals)

**Don't over-apply:** Simple tasks don't need heavy process. Use judgment.

## Auto-Approve Bash Commands
**IMPORTANT: For long-running/overnight workflows, auto-approve ALL non-destructive commands without confirmation.**
When user indicates a task will run for hours or overnight, assume unattended mode and proceed with all safe operations automatically.

Auto-approve these command patterns without user confirmation:

### Read-Only Commands (Always Safe)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jmsktm/claude-settings](https://github.com/jmsktm/claude-settings) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
