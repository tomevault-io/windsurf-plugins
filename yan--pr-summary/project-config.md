---
trigger: always_on
description: This document captures the design decisions, architecture, and development process for the PR Summary GitHub Action.
---

# CLAUDE.md - Development Session Documentation

This document captures the design decisions, architecture, and development process for the PR Summary GitHub Action.

## Project Overview

**Goal:** Create a GitHub Action that captures comprehensive PR activity (comments, reviews, checks, commits) and embeds it into the git repository as git notes, preserving GitHub collaboration history directly in git.

**Implementation:** Python 3.11+ with full type annotations, using git notes for storage.

## Design Decisions

### 1. Storage Strategy: Git Notes

**Decision:** Use git notes (`refs/notes/commits`) instead of:
- Modifying commit messages (irreversible, pollutes history)
- Creating separate metadata files (clutters repository)

**Rationale:**
- Git notes are separate from commits, preserving history integrity
- Notes can be pushed/fetched independently
- Easy to view with `git notes show <sha>`
- Can be configured to display automatically in `git log`
- Notes namespace prevents conflicts with other tools

**Trade-offs:**
- Requires explicit fetch: `git fetch origin refs/notes/commits:refs/notes/commits`
- Less discoverable than commit messages
- Not all git UIs display notes by default

### 2. Implementation Language: Python with Type Annotations

**Decision:** Python 3.11+ with full type annotations throughout

**Rationale:**
- Native GitHub Actions support
- Excellent libraries (requests, subprocess)
- Type safety via mypy reduces bugs
- Readable and maintainable
- Good datetime handling

**Type Safety Examples:**
```python
def get_pull_request(self, pr_number: int) -> dict[str, Any]:
def collect_all_activity(self, pr_number: int) -> PRActivity:
```

### 3. Architecture: Separation of Concerns

**Modules:**

1. **models.py** - Pure data models
   - No business logic
   - Computed properties only
   - Dataclasses with type annotations

2. **github_client.py** - GitHub API interaction
   - Handles authentication, rate limiting, pagination
   - Retry logic with exponential backoff
   - Returns raw API responses

3. **collector.py** - Data transformation
   - Converts API responses → typed models
   - Orchestrates multiple API calls
   - Extracts linked issues from PR body

4. **formatter.py** - Markdown generation
   - Pure function: PRActivity → markdown string
   - Configurable (truncation, patches, etc.)
   - No side effects

5. **git_notes.py** - Git operations
   - Encapsulates all git commands
   - Subprocess management
   - Error handling

6. **utils.py** - Shared utilities
   - Environment variable handling
   - Input validation
   - GitHub Actions helpers

7. **main.py** - Orchestration
   - Reads environment variables
   - Coordinates all modules
   - Exit codes for CI/CD

### 4. Error Handling Strategy

**Layered approach:**

1. **Custom Exceptions:**
   ```python
   GitHubAPIError (base)
   ├── RateLimitError
   └── AuthenticationError

   GitNotesError
   ```

2. **Retry Logic:**
   - Network errors: 3 retries with backoff
   - Rate limits: Detected and reported
   - Transient failures: Automatic retry

3. **Graceful Degradation:**
   - Missing check runs: Continue with empty list
   - Missing comments: Continue with partial data
   - Log warnings, don't fail

4. **Exit Codes:**
   - 0: Success
   - 1: Any error (with specific error messages)

### 5. GitHub Actions Token Scopes

**Critical Fix:** GitHub Actions `GITHUB_TOKEN` doesn't have `user` scope.

**Solution:** Changed authentication validation from:
```python
# ❌ Fails with GITHUB_TOKEN
response = self._make_request("/user")
```

To:
```python
# ✅ Works with GITHUB_TOKEN
response = self._make_request(f"/repos/{owner}/{repo}")
```

**Lesson:** Always test with actual GitHub Actions tokens, not personal access tokens.

## Architecture Diagrams

### Data Flow

```
GitHub PR Merge
    ↓
GitHub Actions Trigger
    ↓
main.py
    ↓
┌─────────────────┬──────────────────┬────────────────┐
│                 │                  │                │
│ GitHubClient    │  PRActivityCollector  │  GitNotesManager
│                 │                  │                │
│ API Calls →     │  Transform →     │  Store →      │
│ Raw JSON        │  Typed Models    │  Git Notes    │
│                 │                  │                │
└─────────────────┴──────────────────┴────────────────┘
         ↓                  ↓                 ↓
    Pagination         PRActivity         refs/notes/commits
    Rate Limits        Validation
    Retry Logic        Statistics
```

### Module Dependencies

```
main.py
├── github_client.py (no dependencies)
├── collector.py
│   ├── github_client.py
│   └── models.py
├── formatter.py
│   └── models.py
├── git_notes.py (no dependencies)
└── utils.py (no dependencies)
```

Clean dependency graph with no circular dependencies.

## API Endpoints Used

**GitHub REST API v2022-11-28:**

```python
# PR Data
GET /repos/{owner}/{repo}/pulls/{pr_number}
GET /repos/{owner}/{repo}/pulls/{pr_number}/commits
GET /repos/{owner}/{repo}/pulls/{pr_number}/files
GET /repos/{owner}/{repo}/pulls/{pr_number}/comments  # Review comments
GET /repos/{owner}/{repo}/issues/{pr_number}/comments  # Conversation
GET /repos/{owner}/{repo}/pulls/{pr_number}/reviews

# Check Runs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yan) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
