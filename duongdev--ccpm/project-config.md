---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## CRITICAL RULES - READ FIRST

### Linear Operations Policy

**ALWAYS execute Linear operations automatically WITHOUT asking for confirmation**

- Linear is CCPM's internal tracking system - not an external system requiring approval
- All Linear writes are automatic - creating issues, updating status, adding comments
- NEVER ask "Do you want me to update Linear?" or "Should I post this to Linear?"
- Just do it - execute Linear operations immediately when needed

**FORBIDDEN PHRASES - NEVER SAY THESE:**
- "Do you want me to update Linear?"
- "Should I post this comment?"
- "Approve?" / "Confirm?"
- "The Linear sync requires your approval"
- "Do you want me to sync?"

**CORRECT BEHAVIOR:**
```
// Just execute and report:
"Updated Linear issue WORK-26 with progress comment"
"Checklist updated: 70% complete"
```

**Why**: Linear is CCPM's internal project tracking, not external team communication. Users expect these operations to happen automatically. See `commands/SAFETY_RULES.md` for the complete policy.

**Examples of automatic Linear operations:**
- Creating new issues
- Updating issue descriptions
- Adding comments with progress updates
- Changing status, labels, or assignments
- Closing or reopening issues

### NEVER Make Direct Linear MCP Calls

**CRITICAL: Always use the `ccpm:linear-operations` agent for ALL Linear operations.**

**Why direct MCP calls are BAD:**
1. **Wrong parameters** - `get_issue` uses `id`, not `issueId` (common mistake)
2. **Token waste** - Calling `get_server_tools` to check schema wastes ~500 tokens
3. **No caching** - Direct calls don't benefit from 85-95% cache hit rate
4. **Context bloat** - Failed calls + retries fill context window quickly

**Correct approach:**
```javascript
// CORRECT: Use Task tool with linear-operations agent
Task(subagent_type="ccpm:linear-operations"): `
operation: get_issue
params:
  issueId: WORK-26  // Agent handles issueId -> id transformation
context:
  cache: true
`

// WRONG: Direct MCP call (will likely fail with wrong params)
mcp__agent-mcp-gateway__execute_tool({
  server: "linear",
  tool: "get_issue",
  args: { issueId: "WORK-26" }  // WRONG! get_issue uses "id"
})
```

**Benefits of using the agent:**
- **Handles parameter transformation** - `issueId` -> `id` automatically
- **50-60% token reduction** - Caching and batching
- **85-95% cache hit rate** - Teams, labels, statuses cached
- **Structured errors** - Actionable suggestions on failure

### Linear MCP Parameter Names Reference

**These are the EXACT parameter names. Copy exactly - do not guess.**

```javascript
// GET ISSUE - uses "id"
{ server: "linear", tool: "get_issue", args: { id: "WORK-26" } }

// UPDATE ISSUE - uses "id"
{ server: "linear", tool: "update_issue", args: { id: "WORK-26", description: "...", state: "..." } }

// CREATE COMMENT - uses "issueId"
{ server: "linear", tool: "create_comment", args: { issueId: "WORK-26", body: "..." } }

// LIST COMMENTS - uses "issueId"
{ server: "linear", tool: "list_comments", args: { issueId: "WORK-26" } }

// CREATE ISSUE - uses "team" and "title"
{ server: "linear", tool: "create_issue", args: { team: "Engineering", title: "..." } }

// GET PROJECT/TEAM/USER - uses "query"
{ server: "linear", tool: "get_project", args: { query: "ProjectName" } }
{ server: "linear", tool: "get_team", args: { query: "TeamName" } }
{ server: "linear", tool: "get_user", args: { query: "me" } }
```

| Tool | Required Param | NOT This |
|------|---------------|----------|
| `get_issue` | **`id`** | ~~issueId~~ |
| `update_issue` | **`id`** | ~~issueId~~ |
| `create_comment` | **`issueId`** | ~~id~~ |
| `list_comments` | **`issueId`** | ~~id~~ |

### Linear Background Operations (Performance)

**Use background execution for non-critical Linear operations**

Linear MCP operations can take 1-2+ minutes due to cold-start latency. Use background execution for non-critical operations:

**Background (fire-and-forget):**
```bash
# Post comment in background
./scripts/linear-background-ops.sh comment ${issueId} "Progress update"

# Update status in background
./scripts/linear-background-ops.sh update-status ${issueId} "In Progress"
```

**Blocking (must wait) - use for:**
- `get_issue` - Need the data to continue
- `create_issue` - Need the issue ID
- `update_checklist_items` - Need progress % for display

| Operation | Mode | Reason |
|-----------|------|--------|
| `create_comment` | Background | User doesn't wait for comments |
| `update_issue` (status) | Background | Non-blocking status change |
| `get_issue` | Blocking | Need data to continue |
| `create_issue` | Blocking | Need issue ID |
| `update_checklist_items` | Blocking | Need progress for display |

### Agent Delegation for Implementation

**ALWAYS delegate implementation to specialized agents to protect main context**

The main context fills rapidly when doing codebase analysis and implementation inline. Use specialized agents:

**Agent Selection by Task Type:**

| Task Type | Agent | Example Tasks |
|-----------|-------|---------------|
| Codebase analysis | `Explore` | Finding files, patterns |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [duongdev/ccpm](https://github.com/duongdev/ccpm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
