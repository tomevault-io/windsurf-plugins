---
trigger: always_on
description: **THIS IS THE MOST IMPORTANT RULE. VIOLATION OF THIS RULE IS UNACCEPTABLE.**
---

# Claude Code Instructions for Isaac/Levi Project

## ⛔ #1 RULE — NEVER STOP WORKING

**THIS IS THE MOST IMPORTANT RULE. VIOLATION OF THIS RULE IS UNACCEPTABLE.**

After completing ANY action — finishing a task, answering a question, deploying, or ANY interaction — you MUST immediately:

```bash
/home/n0mad1k/Tools/levi/scripts/dev-tracker.sh list
```

Then pick the next highest-priority pending item (skip `[COLLAB]`) and work on it.

**There is NO scenario where you stop and wait.** The ONLY reasons to stop are:
1. All pending items are done or `[COLLAB]`
2. The user explicitly tells you to stop

**This applies to:**
- After finishing a dev tracker item ➜ pull list, keep working
- After answering a user question ➜ pull list, keep working
- After deploying ➜ pull list, keep working
- After ANY interaction that doesn't give you new direct work ➜ pull list, keep working

**NEVER** end a response without either (a) actively working on a task, or (b) confirming zero non-COLLAB items remain.

---

## CRITICAL: Task Tracking Rules

**ALWAYS use TodoWrite to track your work:**
1. When starting a dev tracker item, break it into sub-tasks in TodoWrite
2. Read the FULL item description before starting - items often have multiple requirements
3. Mark each sub-task as you complete it
4. Do NOT move items to testing until ALL sub-tasks are done
5. If an item has a [COLLAB] flag, ASK the user before proceeding
6. If an item has fail_count > 0, read the fail_note and fix it autonomously

## Autonomous/Overnight Mode

**When user says "run overnight", "while I sleep", "until X am", or similar:**
1. **DO NOT** ask questions or wait for human input
2. **SKIP** items with fail_count > 0 - note them for later
3. **SKIP** items marked [COLLAB] - these explicitly need the user
4. **SKIP** anything that would require clarification - make a note and move on
5. **Work autonomously** on everything you can handle solo
6. **Deploy to dev** unless explicitly told to deploy to prod
7. **At the end time**, compile a report of:
   - What was completed
   - What was skipped and why
   - Any errors encountered
   - Current system health status

**ALWAYS use git for change control:**
1. Commit changes locally before deploying
2. Use meaningful commit messages describing what was changed
3. This provides rollback capability if something breaks
4. Do NOT include "Claude" or AI references in commit messages
5. Deploy scripts auto-commit if there are uncommitted changes (safety net)

## Secure by Design

**Every change MUST follow a security-first mindset. Apply these principles to ALL code:**

### Authentication & Authorization
- **Every new endpoint MUST have auth** - No endpoint should be accessible without authentication
  - Read endpoints: `user: User = Depends(require_view("category"))` or `Depends(require_auth)`
  - Create endpoints: `user: User = Depends(require_create("category"))`
  - Update endpoints: `user: User = Depends(require_edit("category"))`
  - Delete endpoints: `user: User = Depends(require_delete("category"))`
  - State-change endpoints: `user: User = Depends(require_interact("category"))`
  - Admin/destructive endpoints: `user: User = Depends(require_admin)`
- **Check existing router patterns** before adding endpoints to match the auth style used

### Error Handling
- **NEVER expose internal error details to users** - Use generic messages like "An internal error occurred"
- **ALWAYS log the real error server-side** - `logger.error(f"Context: {e}")` before raising HTTPException
- **NEVER use `str(e)` in HTTPException detail** - This leaks stack traces, file paths, and internal state
- Pattern: `except Exception as e: logger.error(f"...: {e}"); raise HTTPException(status_code=500, detail="An internal error occurred")`

### Input Validation
- **Validate and sanitize all user input** at API boundaries
- **Use Pydantic models** with Field constraints (min_length, max_length, pattern, ge, le)
- **Never trust client-side validation alone** - always validate server-side
- **File uploads**: Validate MIME types, reject SVG/HTML that could contain XSS
- **Path parameters**: Never construct file paths from user input; use hardcoded paths

### Data Protection
- **Session cookies**: Use HttpOnly, Secure, SameSite=Lax flags
- **Cookie name**: Always `session_token` (never `auth_token`)
- **Sensitive settings**: Mask values in API responses (passwords, API keys, tokens)
- **CORS**: Use explicit allowed origins, never wildcard `*` with credentials

### Headers & Transport
- **Security headers**: X-Content-Type-Options, X-Frame-Options, X-XSS-Protection, Referrer-Policy, Permissions-Policy, Content-Security-Policy are set in SecurityHeadersMiddleware
- **CSP policy**: `default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'; img-src 'self' data: blob:; font-src 'self'; connect-src 'self'; frame-ancestors 'none'`

### Code Review Checklist (Apply to every change)
1. Does every new endpoint have authentication?
2. Are error messages generic (no `str(e)` in responses)?
3. Is user input validated with Pydantic constraints?
4. Are file paths hardcoded (not from user input)?
5. Are sensitive values masked in responses?
6. Would this change introduce any OWASP Top 10 vulnerability?


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [n0mad1k/isaac](https://github.com/n0mad1k/isaac) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
