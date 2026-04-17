---
trigger: always_on
description: Beyond Borders is a web application that allows users to add circular, flag-colored borders to their profile pictures to show support for marginalized groups and selected causes. Built with React, TypeScript, Vite, and Canvas 2D.
---

# Cursor AI Assistant Rules for Beyond Borders

## Project Overview
Beyond Borders is a web application that allows users to add circular, flag-colored borders to their profile pictures to show support for marginalized groups and selected causes. Built with React, TypeScript, Vite, and Canvas 2D.

## Critical Workflow Rules

### Issue-First Development
**CRITICAL: When user mentions an existing issue number (e.g., "pick up issue 103", "work on #42"), NEVER create a new issue. Instead:**
1. Fetch the existing issue details using `gh issue view <number>`
2. Update its status to "InProgress" if starting work
3. Use the github-helper.ps1 script for ALL GitHub operations

**ALWAYS use the github-helper.ps1 script for GitHub operations:**
```powershell
# View an issue
gh issue view <number>

# Update issue status
.\.github\scripts\github-helper.ps1 issue-update -Number <number> -Status InProgress

# Create NEW issue (only when explicitly asked to create one)
.\.github\scripts\github-helper.ps1 issue-create `
  -Title "Your task title" `
  -BodyFile ".local/issue.md" `
  -Priority P0|P1|P2 `
  -Size XS|S|M|L|XL `
  -Status InProgress
```

**Priority Levels:**
- **P0 (Critical)**: Security issues, breaking bugs, WCAG compliance issues
- **P1 (High)**: Important features, UX improvements, performance issues
- **P2 (Low)**: Nice-to-have features, refactoring, documentation

**Size Estimates:**
- **XS**: < 1 hour
- **S**: 1-2 hours
- **M**: 2-4 hours
- **L**: 4-8 hours
- **XL**: 8+ hours

**Status Values:**
- **Backlog**: Not yet started
- **Ready**: Ready to be picked up
- **InProgress**: Currently being worked on
- **InReview**: In code review or testing
- **Done**: Completed and merged (auto-closed by PR merge)

**CRITICAL RULE:** Do **NOT** move an issue to `InReview` until after the PR is created. Finishing local commits without a PR still counts as `InProgress`.

**Never:**
- Don't manually close issues (let PR merge do it)
- Don't set status to `Done` manually
- Don't set `InReview` before a PR exists
- Don't leave issues in `InProgress` once PR is open

### Issue Structure Template
When creating issues, use this structure:
```markdown
## Goal
[Clear, concise description of what needs to be accomplished]

## Tasks
- [ ] Task 1
- [ ] Task 2
- [ ] Task 3

## Context
[Why this work is needed, background information]

## Acceptance Criteria
- [ ] Criterion 1
- [ ] Criterion 2
- [ ] Criterion 3

## Technical Notes (optional)
[Any relevant technical details, dependencies, or considerations]
```

## File Organization

### Local Working Files
**Always use the `.local/` directory** for temporary and local development files. Never put them in the repo root or docs/ unless they are committed project docs.
- Temporary scripts and PowerShell files
- Test outputs and screenshots
- Work-in-progress data files
- Personal notes and todos
- **Changelogs, release notes, and other draft docs for public consumption** (e.g. `.local/CHANGELOG-1.4.1-1.5.3.md`)
- Any files that shouldn't be committed

**Example:** `.local/temp-script.ps1`, `.local/screenshots/preview.png`, `.local/notes.md`, `.local/CHANGELOG-*.md`

The entire `.local/` directory is gitignored (except `.local/README.md`).

## Code Standards

### TypeScript
- **Always use strict TypeScript** with no `any` types unless absolutely necessary
- Use explicit return types for all functions
- Prefer interfaces over types for object shapes
- Use type guards and discriminated unions for type safety
- All props and state should be properly typed

### React
- **Use functional components** with hooks only
- Use `React.memo()` for expensive components
- Keep components small and focused (single responsibility)
- Prefer composition over inheritance
- Use custom hooks to encapsulate reusable logic
- Always handle loading and error states

### Error Handling
- Use custom error classes (`RenderError`, `FlagDataError`, `FileValidationError`)
- Implement error boundaries for graceful degradation
- Provide user-friendly error messages
- Log errors appropriately for debugging

### Performance
- Use `useMemo` and `useCallback` appropriately (not everywhere)
- Implement debouncing for expensive operations
- Use `requestIdleCallback` for non-critical background tasks
- Optimize Canvas operations (use OffscreenCanvas when available)
- Implement image preloading for frequently used assets

### Accessibility (WCAG AA Compliance)
- All interactive elements must be keyboard accessible
- Provide ARIA labels and descriptions where needed
- Ensure proper heading hierarchy (h1 → h2 → h3)
- Maintain color contrast ratios (4.5:1 for normal text, 3:1 for large text)
- Support screen readers with semantic HTML
- Test with keyboard navigation and screen readers
- Include focus indicators for all interactive elements

### Testing
- **Write tests for all new features** before considering them complete
- Unit tests: Use Vitest with React Testing Library
- E2E tests: Use Playwright for critical user flows
- Aim for meaningful coverage, not just high percentages
- Test error states and edge cases

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ravendarque) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
