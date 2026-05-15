---
trigger: always_on
description: > **⚠️ IMPORTANT: This is a PROOF OF CONCEPT. All outputs MUST be validated by qualified security professionals. See [DISCLAIMER.md](DISCLAIMER.md) for critical safety information.**
---

# LimaCharlie MCP Usage Guide

> **⚠️ IMPORTANT: This is a PROOF OF CONCEPT. All outputs MUST be validated by qualified security professionals. See [DISCLAIMER.md](DISCLAIMER.md) for critical safety information.**

This document contains critical instructions for working with the LimaCharlie MCP integration.

## Additional Resources

### Core References
@instructions/LCQL_EXAMPLES.md - LCQL query patterns
@instructions/SAMPLE_EVENTS.md - Event structure examples  
@instructions/CLAUDE-REFERENCE.md - Complete function reference
@instructions/CLAUDE-WORKFLOWS.md - Example workflows and best practices

### Practical Examples
@examples/incident-response-playbook.md - Step-by-step IR procedures
@examples/threat-hunting-queries.md - Proactive threat hunting patterns

### Investigation Workflows (Confidence-Rated)
@workflows/investigation/CLAUDE.md - Guidelines for creating confidence-rated workflows
@workflows/investigation/execution.md - Detecting renamed binaries & suspicious execution (CODE_IDENTITY focus)
@workflows/investigation/lateral-movement.md - Systematic lateral movement detection with 5 PsExec methods
@workflows/investigation/persistence.md - Comprehensive persistence detection using efficient registry-based techniques

### Sigma Detection Rules Reference
@references/sigma-limacharlie/ - Community-maintained Sigma rules converted to LimaCharlie format
- Contains 3,200+ detection rules from the Sigma project (on `rules` branch)
- Organized by platform (Windows, Linux, macOS) and detection maturity
- Use as inspiration for crafting queries and understanding suspicious behaviors
- Update regularly with: `git submodule update --remote --merge` in project root

## 📝 Continuous Documentation Improvement

**IMPORTANT**: As you work with LimaCharlie MCP, continuously validate and improve these instructions:

1. **Track Issues**: When queries fail or behave unexpectedly, note the root cause
2. **Verify Examples**: Test that LCQL examples actually work with real data
3. **Document Discoveries**: When you learn new patterns or limitations, suggest updates
4. **Propose Changes**: Always suggest documentation improvements when you identify:
   - Incorrect LCQL syntax or examples
   - Missing guidance on timestamp handling
   - Undocumented field structures or array notations
   - New troubleshooting techniques
   - Better investigation workflows

**When suggesting updates**: 
- Clearly explain what's wrong with current instructions
- Provide the corrected version with examples
- Ask user to confirm before making changes
- Only update instructions you're 100% confident are correct

Example: "I discovered that NETWORK_ACTIVITY requires array notation (*/). Should I update the documentation to reflect this?"

## ⚠️ Critical Notes

### 🛑 HIGH-IMPACT COMMAND SAFETY

**NEVER automatically execute high-impact commands without explicit user confirmation:**

#### Commands Requiring Explicit Confirmation:
- **`isolate_network(sid)`** - Quarantines endpoint from network
  - ⚠️ IMPACT: Disconnects system from all network access
  - ⚠️ IMPACT: May disrupt business operations
  - ⚠️ IMPACT: Requires `rejoin_network()` to restore connectivity
  - **ALWAYS ASK:** "This will isolate the endpoint from the network. Are you sure you want to proceed?"

- **`add_tag()` / `remove_tag()`** - Modifies sensor tags
  - ⚠️ IMPACT: May trigger automated response rules
  - ⚠️ IMPACT: Changes how system is treated by policies
  - **ALWAYS ASK:** "This will modify sensor tags which may trigger automated actions. Proceed?"

- **YARA scanning operations** - Memory/file scanning
  - ⚠️ IMPACT: Can cause performance degradation
  - ⚠️ IMPACT: May trigger false positives
  - **ALWAYS WARN:** About potential performance impact before scanning

#### Required Safety Protocol:
1. **EXPLAIN the impact** - Clearly describe what will happen
2. **WARN about risks** - Outline potential disruptions
3. **ASK for confirmation** - Get explicit "yes" from user
4. **PROVIDE rollback steps** - Explain how to undo if needed
5. **NEVER assume intent** - Even if user seems to imply action

#### Example Safe Interaction:
```
User: "The system looks compromised"
Assistant: "I can see suspicious activity. To contain this threat, I could isolate the endpoint from the network. 

⚠️ WARNING: Network isolation will:
- Disconnect ALL network access immediately
- Potentially disrupt user work and services
- Require manual intervention to restore connectivity

To proceed with isolation, please explicitly confirm: 'Yes, isolate the system'
To continue investigation without isolation: 'Continue without isolation'"
```

### Time and Timestamps

- **ALWAYS use UTC timestamps** when querying LimaCharlie APIs
- **CRITICAL: Always get current time with `date -u +%s` FIRST** - Never use hardcoded timestamps
- LimaCharlie expects all time parameters in Unix epoch format
- **🚨 CRITICAL TIMESTAMP CONVERSION:**
  - LimaCharlie event timestamps are in **milliseconds** (e.g., `1754346325456`)
  - For Unix timestamp conversion, **divide by 1000** to get seconds: `1754346325`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Digital-Defense-Institute/lc-claude-workbench](https://github.com/Digital-Defense-Institute/lc-claude-workbench) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
