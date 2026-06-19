---
trigger: always_on
description: Provides overall methodology, philosophy, and reference material for binary reverse engineering.
---

# Binary Reverse Engineering Plugin

## Overview

Agentic binary reverse engineering for ELF binaries from embedded devices. The LLM drives analysis while humans provide context about the target platform, hardware, and suspected purpose.

## CRITICAL: Skill Invocation Required

**You MUST invoke the `binary-re` skill BEFORE taking any action** when:

### The Core Pattern
**User has a binary + wants to understand something about it**

This covers ANY question about a binary's behavior, purpose, or internals:
- "I have a binary that does X..."
- "What does this binary do?"
- "How does this work?"
- "Can you analyze/figure out/understand this?"
- "This executable [anything]..."

### Why This Matters
The skill provides:
1. **Structured methodology** - Hypothesis-driven analysis prevents wasted effort
2. **Tool selection** - Correct r2/Ghidra/QEMU invocation for the architecture
3. **Human-in-the-loop gates** - Safety checks before execution
4. **Episodic memory integration** - Findings persist across sessions

### What NOT To Do
❌ Run `file`, `strings`, `rabin2`, or `r2` commands before invoking the skill
❌ Start analyzing without checking episodic memory for prior work
❌ "Just quickly check" anything before invoking the skill
❌ Skip triage phase and jump to static/dynamic analysis

### Correct Behavior
```
User: "I have a binary here that checks this key and validates it.
       Can we determine what it's checking?"

Claude:
1. Invoke binary-re skill (FIRST)
2. Follow skill's triage → static → dynamic → synthesis flow
3. Gate execution decisions through human approval
```

The skill handles ALL binary analysis scenarios - don't try to enumerate them.

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    SIMPLIFIED ARCHITECTURE                   │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────┐     ┌──────────────┐     ┌──────────────┐ │
│  │ binary-re    │     │  episodic-   │     │    Bash      │ │
│  │   skill      │────▶│   memory     │     │   (tools)    │ │
│  │ (workflow)   │     │ (persistence)│     │              │ │
│  └──────────────┘     └──────────────┘     └──────────────┘ │
│         │                    │                    ▲         │
│         │                    │                    │         │
│         └────────────────────┴────────────────────┘         │
│                         Claude                               │
│                   (reasoning + orchestration)                │
└─────────────────────────────────────────────────────────────┘
```

**Key insight:** No custom MCP server needed. Episodic memory provides cross-session persistence. Claude orchestrates tools directly via Bash.

## Skills Included

### Main Skill: binary-re

Provides overall methodology, philosophy, and reference material for binary reverse engineering.

### Phase-Specific Skills (Auto-Detect)

| Skill | Purpose | Trigger Keywords |
|-------|---------|------------------|
| `binary-re:triage` | Fast fingerprinting via rabin2 | "what is this binary", "identify", "file type" |
| `binary-re:static-analysis` | r2 + Ghidra deep analysis | "disassemble", "decompile", "functions" |
| `binary-re:dynamic-analysis` | QEMU/GDB/Frida runtime observation | "run", "execute", "debug", "trace" |
| `binary-re:synthesis` | Report generation | "summarize", "report", "document" |
| `binary-re:tool-setup` | Tool installation guides | "install", "setup", "tool not found" |

**Note:** Each skill auto-detects based on keywords in your request. The runtime's semantic matching handles routing automatically.

## Episodic Memory Integration

Knowledge persists across sessions via episodic memory with consistent tagging:

```
[BINARY-RE:{phase}] {artifact_name} (sha256: {hash})
FACT: {observation} (source: {tool})
HYPOTHESIS: {theory} (confidence: {0.0-1.0})
QUESTION: {unknown}
DECISION: {choice} (rationale: {why})
```

### Session Management

**Starting new analysis:**
1. Compute `sha256sum binary`
2. Search episodic memory: `[BINARY-RE] sha256:{hash}`
3. If found: "Previous analysis exists. Resume or start fresh?"

**Resuming analysis:**
1. Search: `[BINARY-RE] {artifact_name}`
2. Load facts/hypotheses from previous session
3. Continue from last phase

**Searching past work:**
- `[BINARY-RE] ARM` - Find all ARM binary analyses
- `[BINARY-RE] FACT: hardcoded` - Find binaries with hardcoded values
- `[BINARY-RE:synthesis]` - Find completed analyses

## Tool Requirements

### Required
- `radare2` (r2) - Static analysis and disassembly
- `qemu-user` - Dynamic emulation for ARM/MIPS/etc.
- `gdb-multiarch` - Cross-architecture debugging

### Recommended
- `ghidra` - Headless decompilation
- `gef` - GDB Enhanced Features
- `frida` - Dynamic instrumentation

### Optional
- `angr` - Symbolic execution
- `unicorn` - CPU emulation
- `yara` - Pattern matching

## Human-in-the-Loop Protocol

The skill gates these operations (requires human approval):

1. **Before any binary execution** - Confirm sandbox is appropriate
2. **Network-capable dynamic analysis** - Prevent unintended phone-home

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [2389-research/binary-re](https://github.com/2389-research/binary-re) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
