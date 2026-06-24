---
trigger: always_on
description: Before implementing ANY solution, ALWAYS:
---

# Recursive Learning Cursor Rules for Browser Extension Development

## Core Learning Principle
Before implementing ANY solution, ALWAYS:
1. Check `.cursor/browserextension-bestpractices.mdc` for known solutions
2. Check `.cursor/known-issues.md` for documented problems and their fixes
3. Check `.cursor/architecture-decisions.md` for design patterns and rationale
4. If encountering an error, document it with solution before moving forward

## Dashboard Integration 🆕
The project has a real-time dashboard (`.dashboard/dashboard.html`) for tracking daily issues.

**MANDATORY: Update dashboard when:**
1. **Starting work** on any issue → Update current work + add issue
2. **Encountering errors** → Add to dashboard immediately  
3. **Fixing bugs** → Update to resolved with fix description
4. **Getting blocked** → Update to blocked with reason
5. **Completing task** → Mark resolved, clear current work

**How to Update Dashboard (Automatic):**

### Method 1: Update Current Work (What AI is doing NOW)
Write to `.dashboard/ai-current-work.json`:
```json
{
  "timestamp": "<ISO timestamp>",
  "active": true,
  "tasks": [
    {
      "title": "Task title",
      "description": "What you're doing",
      "status": "in-progress",
      "startTime": "<ISO timestamp>",
      "files": "file1.ts, file2.ts"
    }
  ]
}
```

### Method 2: Queue Issues for Dashboard
Append to `.dashboard/ai-issues-queue.json`:
```json
[
  {
    "timestamp": "<ISO timestamp>",
    "title": "Issue title",
    "description": "Context",
    "status": "in-progress|resolved|blocked",
    "severity": "critical|high|medium|low",
    "fix": "Solution (if resolved)",
    "files": "Comma-separated file paths"
  }
]
```

### Method 3: One Command Sync
After updating JSON files, tell user to run:
```bash
node .dashboard/dashboard-bridge.js
```
This generates a sync page they can use to update the dashboard.

**Important**: ALWAYS update these JSON files when working on issues. Tell user: "Dashboard updated - run sync or refresh dashboard to see changes."

## Self-Correction Protocol

### When Encountering Errors
1. **Identify Root Cause**: Don't just fix symptoms, understand WHY the error occurred
2. **Document Immediately**: Add to `.cursor/known-issues.md` with:
   - Error description and symptoms
   - Root cause analysis
   - Solution implemented
   - Prevention strategy for future
   - Date and context
3. **Update Best Practices**: If it's a pattern, add to `browserextension-bestpractices.mdc`
4. **Verify Fix**: Test thoroughly before considering it resolved

### When Making Architecture Decisions
1. **Document Decision**: Add to `.cursor/architecture-decisions.md` with:
   - Problem being solved
   - Options considered
   - Decision made and rationale
   - Trade-offs accepted
   - Date
2. **Reference in Code**: Add comments linking to the decision document

### Before Implementing Features
1. **Check Learning Docs**: Review all `.cursor/*.md` files for relevant patterns
2. **Apply Known Solutions**: Reuse proven approaches from best practices
3. **Avoid Known Pitfalls**: Check known-issues.md for similar past problems

## Browser Extension Specific Rules

### Always Apply These Patterns
- ✅ Use React-compatible input setters (see browserextension-bestpractices.mdc)
- ✅ Implement page stability checks before DOM manipulation
- ✅ Handle both regular and shadow DOM elements
- ✅ Test across Chrome, Firefox, Edge, Safari if applicable
- ✅ Use message passing for content script ↔ background communication
- ✅ Never use `eval()` or inline scripts (CSP violations)

### Common Browser Extension Pitfalls
1. **Async Timing Issues**
   - Problem: Race conditions with page load
   - Solution: Always use page stability gates before DOM operations
   
2. **React/Vue State Management**
   - Problem: Setting input.value directly doesn't update framework state
   - Solution: Use property descriptor setters + dispatch events
   
3. **Content Security Policy**
   - Problem: Inline scripts/styles blocked
   - Solution: Use external files, hash-based CSP, or nonces

4. **Storage Sync Limits**
   - Problem: chrome.storage.sync has 100KB limit
   - Solution: Use chrome.storage.local for large data, sync only config

5. **Cross-Origin Requests**
   - Problem: Content scripts inherit page CSP, can't make arbitrary requests
   - Solution: Use background script as proxy, declare permissions

## Learning File Structure

### `.cursor/known-issues.md`
Format:
```markdown
# Known Issues & Solutions

## [Issue Title] - [Date]
**Severity**: Critical/High/Medium/Low
**Context**: [When/where this occurs]
**Symptoms**: [What you observe]
**Root Cause**: [Why it happens]
**Solution**: [How to fix]
**Prevention**: [How to avoid in future]
**Related Files**: [List of affected files]

---
```

### `.cursor/architecture-decisions.md`
Format:
```markdown
# Architecture Decision Records (ADR)

## ADR-001: [Decision Title] - [Date]
**Status**: Accepted/Superseded
**Context**: [Problem space]
**Decision**: [What we decided]
**Rationale**: [Why this approach]
**Consequences**: [Trade-offs]
**Alternatives Considered**: [Other options]

---
```

### `.cursor/browserextension-bestpractices.mdc`
Already exists - keep updating with proven patterns

## Mandatory Pre-Implementation Checks


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [offlyn-ai/offlyn-apply](https://github.com/offlyn-ai/offlyn-apply) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
