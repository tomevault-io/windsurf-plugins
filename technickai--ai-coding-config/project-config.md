---
trigger: always_on
description: When we are creating prompts that will be used by an LLM for agents
---


# Prompt Engineering Best Practices for LLM-to-LLM Communication

When creating prompts that will be read and executed by other LLMs (commands, workflows,
agent prompts), follow these practices. These guidelines are for prompts that LLMs write
for other LLMs to consume - not for human-to-LLM interaction.

## Why This Document Minimizes Formatting

This document is designed to be read by LLMs, not humans. Therefore:

- Minimal markdown formatting: No excessive bold, italics, or decorative symbols. These
  waste tokens and add no semantic value for LLM comprehension.
- Minimal "bad" examples: LLMs encode patterns from what they see, regardless of labels
  like "wrong" or "don't do this." Showing anti-patterns teaches the LLM to reproduce
  them.
- Simple structure: Headings for organization, code blocks for actual patterns, plain
  text for instructions.
- Clear over clever: Direct language that LLMs can parse literally, not stylistic
  variations.

When you read "avoid X" or see a counterexample in this document, understand that we're
violating our own principle for teaching purposes - but minimize this pattern in prompts
you create for LLM consumption.

## Key Principles for LLM-Readable Prompts

- Assume the executing model is smarter: The model executing your prompt is likely more
  capable than the model that created it. Trust its abilities rather than
  over-prescribing implementation details.
- Front-load critical information: LLMs give more weight to early content
- Be explicit: LLMs can't infer context the way humans do
- Maintain consistency: Use the same terminology throughout
- Structure matters: Clear boundaries (especially XML tags) help LLMs parse complex
  prompts
- Examples teach patterns: What you show is what the LLM will do
- Clarity over brevity: Never sacrifice unambiguous interpretation for token savings
- Explain motivation: Tell the LLM _why_ a constraint exists - it generalizes from
  reasoning better than from bare rules
- Descriptive over directive: "Use this tool when modifying files" works better than
  "CRITICAL: You MUST use this tool" - aggressive language can cause over-triggering
- Positive framing: "Write in flowing prose" is clearer than "Don't use markdown" -
  positive instructions are unambiguous, negative ones require constructing then
  negating

## Pattern Reinforcement Through Examples

When creating prompts for other LLMs to execute, pattern teaching becomes critical. LLMs
learn from what you show them, not from what you tell them to avoid.

### How LLMs Process Examples in Prompts

When an LLM reads a prompt file with examples, it encodes those patterns for
reproduction:

1. Pattern Matching Over Labels: LLMs reproduce structural patterns. Code structure
   creates strong activation. Text labels like "wrong" or "don't do this" are weak
   signals that don't override pattern encoding.

2. Direct Teaching Through Examples: When writing command workflows or agent prompts,
   you're teaching the executing LLM. Every example shown is a lesson it will follow.

3. Consistency is Critical: In LLM-to-LLM communication, inconsistent examples cause
   unpredictable behavior. The executing LLM can't resolve ambiguity the way humans do.

4. Attention Weighting: All tokens in the prompt receive attention. Structural patterns
   activate high attention regardless of surrounding text saying "avoid this."

### Writing Effective Instructions for LLM Execution

For command workflows and agent prompts:

1. Flood with correct patterns: Show 5+ examples of the standard approach
2. Never show anti-patterns: Don't include "wrong" examples - the LLM will reproduce
   them
3. Describe exceptions in prose: If there are edge cases, describe them in words, not
   code
4. Maintain pattern consistency: All examples should follow the same structure

Example of good pattern teaching in a prompt file:

```xml
<task>
Update all API endpoints to use consistent error handling
</task>

<examples>
// Pattern to follow:
async function getUser(id) {
  try {
    const user = await db.users.findById(id);
    if (!user) {
      throw new NotFoundError('User not found');
    }
    return user;
  } catch (error) {
    logger.error('Failed to get user', { id, error });
    throw error;
  }
}

async function updateUser(id, data) {
  try {
    const user = await db.users.update(id, data);
    if (!user) {
      throw new NotFoundError('User not found');
    }
    return user;
  } catch (error) {
    logger.error('Failed to update user', { id, error });
    throw error;
  }
}

async function deleteUser(id) {
  try {
    const result = await db.users.delete(id);
    if (!result) {
      throw new NotFoundError('User not found');
    }
    return result;
  } catch (error) {
    logger.error('Failed to delete user', { id, error });
    throw error;
  }
}
</examples>

<instructions>
Apply this exact error handling pattern to all endpoints. If an endpoint doesn't interact with the database, omit the NotFoundError check but keep the try-catch structure and logging.
</instructions>
```

### The Mechanism in LLM-to-LLM Communication

When one LLM writes a prompt for another to execute:

- The executing LLM encodes ALL patterns shown, regardless of labels

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TechNickAI/ai-coding-config](https://github.com/TechNickAI/ai-coding-config) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
