---
trigger: always_on
description: **CONTEXT**: You are Claude using Claude Code to work on the Claude Chrome MCP project - a system that enables AI agents to control Chrome browsers through MCP (Model Context Protocol) tools.
---

# Claude Chrome MCP

**CONTEXT**: You are Claude using Claude Code to work on the Claude Chrome MCP project - a system that enables AI agents to control Chrome browsers through MCP (Model Context Protocol) tools.

**ARCHITECTURE OVERVIEW**: 
```
[AI Agent/Claude Code] → [MCP Server] → [Modular Relay System] → [Chrome Extension] → [Chrome Browser]
                         44 tools         Auto-election         Passive health        Controls Chrome
                                         Port 54321           monitoring
```

**CRITICAL RULES**:

## Document Philosophy

THIS DOCUMENT IS: An operational playbook for daily use
THIS DOCUMENT IS NOT: A development log, architecture guide, or progress tracker

ALWAYS:
- Answer "What do I do?" not "How does it work?"
- Update immediately when workflows change
- Test every command before documenting
- Include expected outputs and timing
- Correct drift the moment it's noticed - no deferrals
- Fill gaps when discovered during use - not "later"
- Formalize patterns after 2+ occurrences

NEVER:
- Document completed work (→ GitHub issues)
- Explain implementations (→ docs/ folder)
- Show code snippets (→ reference file paths)
- Add emoji or decorative glyphs
- Leave "TODO" or "TBD" markers - fix it now
- Accept vague timings like "a while" - measure and specify

## Content Patterns

### Primary Pattern: WHEN/THEN
```
WHEN: [Specific situation]
THEN: [Concrete action with example]
```

### Troubleshooting Pattern: SYMPTOM → DIAGNOSIS → TREATMENT
```
SYMPTOM: MCP tool timeout
DIAGNOSIS: Check connection with `mcp system_health`
TREATMENT: 
  Level 1: `mcp chrome_reload_extension`
  Level 2: Manual reload at chrome://extensions/
  Level 3: Enable debug mode and check logs
```

### Decision Pattern: IF/THEN/ELSE
```
IF: extension.relayConnected = false
THEN: Reload extension
ELSE IF: Still disconnected after reload
THEN: Manual intervention required
ELSE: Continue with operations
```

### Workflow Pattern: CONTEXT → ACTION → EXPECTED
```
CONTEXT: Need to send message to Claude
ACTION: `mcp tab_send_message --tabId 12345 --message "Hello"`
EXPECTED: Returns immediately with operation ID
```

## Information Architecture

PUT THIS HERE | NOT HERE
---|---
Operational procedures | Implementation details
Command examples with output | Code snippets
Troubleshooting steps | Root cause analysis
Active warnings (e.g., Issue #7) | Resolved issues
File path references | File contents

ROUTING RULES:
- Active work → GitHub Issues
- Stable knowledge → docs/ folder  
- What changed → Commit messages
- How to operate → CLAUDE.md
- Why it works → Architecture docs

## Command Documentation Standards

GOOD EXAMPLE:
```bash
# Create tab (2-3 seconds)
mcp tab_create --injectContentScript
# Output: { "success": true, "tabId": 12345 }
```

BAD EXAMPLE:
```bash
# Create a tab
mcp tab_create  # Missing timing, options, output format
```

PATTERN FOR COMMANDS:
1. Purpose comment with timing expectation
2. Full command with common options
3. Expected output format or behavior
4. Common failure modes (if any)

## Critical Operational Rules

ALWAYS:
- Test in correct environment (Extension → MCP tools, Server → CLI tools)
- Check `mcp system_health` before complex operations
- Use `timeout` if you notice commands timing out
- Use file paths not code: "See implementation in `path/to/file.js:123`"
- Document timing: "Returns immediately" vs "Blocks 2-3 seconds"
- Capture error codes/messages when encountered → Add to patterns
- Verify recovery with explicit checklist after any failure
- Document performance baseline on first observation

NEVER:
- Add `sleep` delays (commands handle own timing)
- Test server changes with MCP tools
- Create new files when existing ones can be edited
- Skip documenting a workaround - formalize it immediately
- Use relative performance terms without baseline numbers

## Troubleshooting Decision Tree

```
Problem Detected
├─ Connection Issue?
│  ├─ YES → See "Connection Issues" pattern
│  └─ NO → Continue
├─ Operation Hanging?
│  ├─ YES → See "Operation Failures" pattern
│  └─ NO → Continue
├─ After Chrome Restart?
│  ├─ YES → See "State Recovery" pattern
│  └─ NO → Continue
└─ Need Detailed Diagnosis?
   └─ See "Diagnostic Recipes"
```

## Git Workflow Rules

BEFORE COMMITTING:
1. ✓ All tests pass
2. ✓ Changes are related
3. ✓ Working directory is clean

COMMIT SEQUENCE:
```bash
git status                    # Review all changes
git add <specific-files>      # Stage selectively  
git commit -m "type: description"
git status                    # Verify clean state
```

IF: Multiple unrelated changes
THEN: Split into separate commits
EXAMPLE: 
- Commit 1: "fix: tab operation timeout handling"
- Commit 2: "docs: update troubleshooting guide"

## Testing Patterns

Component → Test Method:
- `extension/*` → `mcp chrome_reload_extension` + MCP tools
- `cli/*` → `npm run build && npm install -g` + CLI commands  
- `mcp-server/*` → `mcp daemon restart` + CLI tools
- `tests/*` → `cd tests && npm test`

COMMON MISTAKE PATTERN:
```
MISTAKE: Using MCP tools after server changes
WHY: MCP tools use Claude Code's server, not your local daemon

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [durapensa/claude-chrome-mcp](https://github.com/durapensa/claude-chrome-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
