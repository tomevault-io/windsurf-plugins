---
trigger: always_on
description: **ALWAYS ask clarifying questions when requirements are ambiguous or unclear.**
---

# Agent Guidelines for WHATWG Specifications Monorepo in Zig

## ⚠️ CRITICAL: Ask Clarifying Questions When Unclear

**ALWAYS ask clarifying questions when requirements are ambiguous or unclear.**

### Question-Asking Protocol

When you receive a request that is:
- Ambiguous or has multiple interpretations
- Missing key details needed for implementation
- Unclear about expected behavior or scope
- Could be understood in different ways

**YOU MUST**:
1. ✅ **Ask ONE clarifying question at a time**
2. ✅ **Wait for the answer before proceeding**
3. ✅ **Continue asking questions until you have complete understanding**
4. ✅ **Never make assumptions when you can ask**

### Examples of When to Ask

❓ **Ambiguous request**: "Implement URL parsing"
- **Ask**: "Should this implement the basic URL parser, the URL parser with base, or just host parsing?"

❓ **Missing details**: "Add encoding support"
- **Ask**: "Which encoding should be supported? Just UTF-8, or should this include legacy encodings like ISO-8859-1?"

❓ **Unclear scope**: "Optimize parser performance"
- **Ask**: "Which part should be prioritized? Character validation, state machine transitions, or memory allocation?"

❓ **Multiple interpretations**: "Handle parsing errors"
- **Ask**: "Should this throw validation errors, collect them for reporting, or fail silently according to the spec?"

### What NOT to Do

❌ **Don't make assumptions and implement something that might be wrong**
❌ **Don't ask multiple questions in one message** (ask one, wait for answer, then ask next)
❌ **Don't proceed with unclear requirements** hoping you guessed correctly
❌ **Don't over-explain options** in the question (keep questions concise)

### Good Question Pattern

```
"I want to make sure I understand correctly: [restate what you think they mean].

Is that correct, or did you mean [alternative interpretation]?"
```

**Remember**: It's better to ask and get it right than to implement the wrong thing quickly.

---

## ⚠️ CRITICAL: No Mid-Task Summaries

**NEVER provide summaries or progress reports in the middle of active work.**

### Summary Rules

**When to Provide Summaries**:
- ✅ **ONLY at the end of completed work** - After all tasks are done and committed
- ✅ **When explicitly asked** - User requests "What did we do?" or similar
- ✅ **After oneshot completion** - Final summary after entire epic is complete

**When NOT to Provide Summaries**:
- ❌ **During active work** - While implementing features, fixing bugs, or writing code
- ❌ **Between logical steps** - After completing one part of a multi-part task
- ❌ **After individual commits** - Just commit and continue to the next step
- ❌ **To update on progress** - Work silently, report only when done

### Correct Behavior

```
User: "Implement feature X"
Agent: [Works on feature X implementation]
Agent: [Commits code]
Agent: [Continues to next logical step]
Agent: [Commits more code]
Agent: [Completes all work]
Agent: "✅ Feature X is complete. [Brief completion note]"
```

### Incorrect Behavior

```
User: "Implement feature X"
Agent: [Works on part 1]
Agent: "I've completed part 1. Here's what I did: [long summary]" ❌ WRONG
Agent: [Works on part 2]
Agent: "Now I've finished part 2. Summary so far: [long summary]" ❌ WRONG
```

### Why This Matters

- **Efficiency**: Summaries break flow and waste time
- **Focus**: Stay focused on completing the work, not reporting on it
- **Clarity**: Final summaries are more valuable than incremental updates
- **User experience**: Users want completed work, not progress reports

**WORK FIRST, SUMMARIZE LAST. If you're not done, keep working.**

---

## ⚠️ CRITICAL: Spec-Compliant Implementation

**THIS IS A WHATWG SPECIFICATIONS MONOREPO** providing Zig implementations of multiple WHATWG standards for web platform compatibility.

### What This Monorepo IS

This project implements multiple WHATWG specifications in idiomatic Zig:

**Currently Implemented**:
- **URL** (`src/url/`) - URL parsing, serialization, and manipulation
- **Encoding** (`src/encoding/`) - Text encoding and decoding (UTF-8, legacy encodings)
- **Console** (`src/console/`) - Console logging and debugging APIs
- **MIME Sniff** (`src/mimesniff/`) - MIME type detection and sniffing
- **WebIDL** (`src/webidl/`) - WebIDL type system and conversions
- **Infra** (`src/infra/`) - Common infrastructure primitives (lists, strings, bytes)
- **Streams** (`src/streams/`) - Streaming data APIs (ReadableStream, WritableStream)

**Available Specs** (in `specs/`): URL, Encoding, Console, Fetch, DOM, Streams, WebIDL, Infra, MIME Sniff, and many more

**IDL Reference Files** (in `idl/`): Symlink to `/Users/bcardarella/projects/webref/ed/idl/` containing all official WHATWG WebIDL definitions used as the source for code generation.

### Context Awareness

**When working on a spec, the system detects context from:**
- File paths (e.g., `src/url/parser.zig` → URL Standard)
- Import statements (e.g., `@import("encoding")` → Encoding Standard)
- Current working directory

**The skills adapt based on detected context** to provide spec-specific guidance.

### Test Guidelines

- Use realistic examples from the target spec
- Test edge cases defined in the specification

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zig-whatwg/crane](https://github.com/zig-whatwg/crane) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
