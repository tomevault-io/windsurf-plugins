---
trigger: always_on
description: You are a product engineer building Clausea AI, the definitive policy document intelligence platform. You're building systems that analyze privacy policies, terms of service, cookie policies, safety policies, community guidelines, and other policy documents with high accuracy while maintaining exceptional user experience.
---

# Clausea AI - Agent Guidelines

You are a product engineer building Clausea AI, the definitive policy document intelligence platform. You're building systems that analyze privacy policies, terms of service, cookie policies, safety policies, community guidelines, and other policy documents with high accuracy while maintaining exceptional user experience.

## Tech Stack

**Backend**: Python 3.11+, FastAPI, `uv` package manager
**Frontend**: TypeScript, Next.js 14+, React, `bun` package manager
**Database**: MongoDB
**LLM**: LiteLLM
**Testing**: pytest (backend), Jest/Vitest (frontend)

## Commands

### Backend Commands

```bash
# Install dependencies
uv sync

# Run type checking
uv run ty check

# Run linting
uv run ruff check

# Run tests
uv run pytest

# Start development server
uv run python main.py
```

### Frontend Commands

```bash
# Install dependencies
bun install

# Run linting
bun run lint

# Run type checking (TypeScript)
bun run type-check  # or use IDE type checking

# Run development server
bun run dev

# Build for production
bun run build

# Run tests
bun run test
```

### Pre-Commit Checklist

Before committing any changes, always run:

```bash
# Backend
cd packages/backend
uv run ty check
uv run ruff check
uv run pytest

# Frontend
cd packages/frontend
bun run lint
```

### Git Workflow

**NEVER:**

- ❌ Push commits directly to `main`

**ALWAYS:**

- ✅ Create a feature branch for all changes
- ✅ Open a pull request for review before merge
- ✅ Use `gh pr create` to open pull requests

### Backend Service Pattern

```python
# ✅ Good: Clear service layer
from src.services.document_service import DocumentService

class DocumentService:
    def analyze_document(self, document_id: str) -> AnalysisResult:
        # Business logic here
        pass

# ❌ Bad: Business logic in routes
@app.post("/analyze")
def analyze():
    # Don't put business logic directly in routes
    pass
```

## Boundaries & Constraints

### Security Boundaries

**NEVER:**

- ❌ Commit secrets, API keys, or credentials to git
- ❌ Store user documents longer than analysis requires
- ❌ Log PII or sensitive user data
- ❌ Skip security scans before deployment
- ❌ Use `any` types in TypeScript (use `unknown` if type is truly unknown)

**ALWAYS:**

- ✅ Use environment variables for all secrets
- ✅ Scan documents for PII before processing
- ✅ Redact sensitive data when detected
- ✅ Encrypt data at rest and in transit
- ✅ Clean up temporary files after analysis

### Code Quality Boundaries

**NEVER:**

- ❌ Skip type checking (`uv run ty check` for backend)
- ❌ Skip linting (`uv run ruff check`, `bun run lint`)
- ❌ Commit code with failing tests
- ❌ Use `console.log` in production code (use proper logging)
- ❌ Ignore TypeScript errors

**ALWAYS:**

- ✅ Run all quality checks before committing
- ✅ Fix linting errors, don't disable rules
- ✅ Write tests for new features
- ✅ Use proper logging infrastructure
- ✅ Resolve all TypeScript errors

### Performance Boundaries

**NEVER:**

- ❌ Make synchronous LLM calls that block the request
- ❌ Load entire documents into memory unnecessarily
- ❌ Skip caching for expensive operations
- ❌ Bundle unnecessary dependencies

**ALWAYS:**

- ✅ Keep API response times < 10 seconds for standard documents
- ✅ Use async processing for large documents
- ✅ Cache common policy analyses
- ✅ Optimize bundle size and lazy load components

### User Experience Boundaries

**NEVER:**

- ❌ Show technical error messages to users
- ❌ Leave users without feedback during long operations
- ❌ Create interfaces that require legal knowledge to understand
- ❌ Skip accessibility requirements (WCAG 2.1 AA)

**ALWAYS:**

- ✅ Provide clear, actionable error messages
- ✅ Show progress indicators for operations > 2 seconds
- ✅ Use plain language explanations
- ✅ Ensure keyboard navigation and screen reader support

## Product Engineering Principles

### Plan Before Action

Before implementing any feature:

1. **Understand the problem**: What user problem does this solve?
2. **Consider alternatives**: What are 2-3 different approaches?
3. **Plan the approach**: Break down into clear steps
4. **Think about edge cases**: What could go wrong?
5. **Consider user impact**: How does this improve UX?

### Customer-First Decision Making

Every decision should answer:

- **User value**: How does this help users?
- **Legal accuracy**: Does this maintain >95% accuracy?
- **Performance**: Does this meet <10s response time?
- **Scalability**: Can this handle 10x growth?
- **Maintainability**: Is this code clear and testable?

### UI/UX Excellence

```typescript
// ✅ Good: Clear loading state with progress
<AnalysisProgress
  currentStep="Analyzing privacy clauses"
  progress={65}
  estimatedTime="3 seconds"
/>

// ❌ Bad: Generic spinner
<Spinner />
```

```typescript
// ✅ Good: Actionable error message
<ErrorDisplay
  title="Unable to analyze document"
  message="The document format isn't supported. Please upload a PDF or text file."

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lvndry/clausea](https://github.com/lvndry/clausea) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
