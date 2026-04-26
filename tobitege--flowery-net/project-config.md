---
trigger: always_on
description: **THIS IS NON-NEGOTIABLE. VIOLATION OF THIS RULE IS UNACCEPTABLE.**
---

# Agent Rules

## Agent Behavior

### CRITICAL: THE 3-ITERATION RULE

**THIS IS NON-NEGOTIABLE. VIOLATION OF THIS RULE IS UNACCEPTABLE.**

If an issue cannot be fixed within **3 consecutive attempts using the SAME approach**, the AI assistant MUST:

1. **STOP making further changes immediately**
2. **Explicitly state**: "I have attempted this 3 times and cannot solve it with this approach."
3. **Ask the user** how they want to proceed before making ANY more modifications
4. **NOT switch to a different approach** without user approval or new user directions

**Rationale**: Endless iteration on a broken approach wastes time, introduces regressions, and frustrates the user. It is BETTER to admit failure early than to keep flailing with random changes that make things worse.

**What counts as an "attempt"**:

- Each code modification targeting the same issue = 1 attempt
- Switching fundamental approach (e.g., Canvas -> Grid, Margin -> Padding) ONLY IF USER APPROVES!
- Minor tweaks to the same approach (e.g., changing values from 8 to 10) do NOT reset the counter

To not hinder iterations completely: a user's indication to continue inactivates this rule for the rest of the session!

### CRITICAL: NEVER REVERT OR SWITCH APPROACHES WITHOUT ASKING

**THIS IS NON-NEGOTIABLE. VIOLATION OF THIS RULE IS UNACCEPTABLE.**

When a solution is not working as expected:

1. **DO NOT revert code** without explicit user approval
2. **DO NOT switch to a different approach** without explicit user approval
3. **DO NOT "try another thing"** - ASK FIRST what the user wants to do
4. **STOP and present options** - Let the user decide the path forward

**Examples of FORBIDDEN behavior:**

- ? "The PNGs aren't loading, let me switch back to SVGs" (without asking)
- ? "This approach has issues, I'll try a different one" (without asking)
- ? "Let me revert this change since it's not working" (without asking)

**Required behavior:**

- ? "The PNGs aren't loading. Options: A) Switch to SVGs, B) Fix PNG deployment, C) Something else. What do you prefer?"
- ? "This isn't working as expected. Should I revert or try to fix it?"
- ? "I see Issue X. Before I change anything, what's your preference?"

**Rationale**: The user may have additional context, may prefer to debug the current approach, or may want to test on other platforms first. Unilateral reversions waste time and destroy progress.

### "No Change" Is a Valid Outcome

When the user reports an issue or asks for investigation **without specifying a required change**:

1. **Investigate first** - Understand the current behavior and why it exists
2. **Assess whether a change is needed** - Sometimes the behavior is correct/intentional
3. **If no obvious solution exists, ASK** - Do NOT go on a "coding spree" trying random fixes
4. **Report findings and recommend** - Present options including "no change" if appropriate

**Rationale**: It is better to confirm with the user before making changes than to introduce unnecessary code churn, potential regressions, or deviations from established design patterns. The user may have context that makes "no change" the correct answer.

**Example scenarios where "no change" may be correct**:

- Behavior follows an established design system (e.g., DaisyUI patterns)
- The "issue" is theme-specific and other themes work correctly
- A workaround already exists (e.g., using a different variant/property)
- The behavior is intentional and documented

- **Think Before You Code**:

Before making any code change, trace through the FULL execution flow. For control/UI changes specifically:

  1. When are properties set? (Construction time vs. after construction)
  2. Do property changes need callbacks to propagate to child elements?
  3. What is the initialization order? (Constructor -> property setters -> Loaded event)
  4. Are there async/deferred events that fire after synchronous code completes?
  
  **Do NOT make partial fixes and iterate.** Analyze the complete picture first, then implement the full solution in one pass.

- **No Flip-Flopping on Approaches**: When the user asks to try a specific approach, commit to that approach fully and let them test it. Don't second-guess midway and switch to an alternative solution. If the first approach doesn't work, the user will provide feedback and we can discuss alternatives together.

### Tool Usage Selection

- **Favor Filesystem MCP Tools**: Use tools like `mcp_filesystem_list_directory`, `mcp_filesystem_search_files`, and `mcp_filesystem_read_file` instead of terminal commands (`ls`, `dir`, `find`, `cat`) whenever possible. These tools provide better interactivity, structured metadata, and safer operation handling.

- **Use Terminal ripgrep for Complex Search**: While `mcp_filesystem_search_files` is preferred for simple glob-based file finding, use terminal `rg` (ripgrep) for complex content searching across the codebase. The internal `grep_search` tool frequently fails. To avoid premature output truncation in long results, interleave with `clear`:

  ```bash
  clear && rg "pattern" path/to/search
  ```

  Examples:
  - Search for function: `clear && rg "function_name" .`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tobitege/Flowery.NET](https://github.com/tobitege/Flowery.NET) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
