---
trigger: always_on
description: Expert Django guidance covering clean code, DRF, testing, security, and performance. Focus on code organization, early return patterns, avoiding bool trap, decoupling business logic from views, and production-ready patterns. For Django 4.2+, DRF, and TestCase. Use when building Django APIs, optimizing queries, implementing authentication, writing tests, or refactoring Django code.
---


# Django General Expert

Expert guidance for Django development with emphasis on clean code, maintainable architecture, and production-ready patterns.

## Objective

Deliver production-ready code that maintains consistency with the project's architecture, meets quality expectations, integrates with the team's workflow, and adheres to security, testing, and documentation standards so that code reviews are efficient and the codebase remains maintainable and scalable.

## Overview

This skill provides comprehensive Django guidance for advanced developers, focusing on:

- **Clean Code Patterns**: Early return, avoiding bool trap, meaningful naming
- **Architecture**: Service layer, decoupled business logic, modular design
- **Django REST Framework**: Serializers, viewsets, permissions, performance
- **Testing**: TestCase-based testing strategies (no pytest)
- **Performance**: Query optimization, caching, database patterns
- **Security**: OWASP Top 10, authentication, authorization

**Target Audience**: Advanced Django developers working on production applications.

**Django Version**: 4.2 LTS and later

## When to Use

Invoke this skill when:

**Code Quality & Organization:**
- "Refactor this view to be cleaner"
- "This code has too many nested ifs"
- "How to organize this Django app?"
- "Avoid bool trap in this code"
- "Make this more readable"

**Models & ORM:**
- "Design models for..."
- "Fix N+1 query problem"
- "Optimize this queryset"
- "Create migrations for..."

**Views & APIs:**
- "Create an API endpoint for..."
- "Build a view that handles..."
- "Decouple logic from this view"
- "Implement permissions for..."

**Testing:**
- "Write tests for this feature"
- "Test this DRF endpoint"
- "Mock external service in tests"

**Security:**
- "Secure this endpoint"
- "Implement authentication"
- "Review for security issues"

**Performance:**
- "This view is slow"
- "Optimize database queries"
- "Add caching to..."

## Instructions

### Step 1: Classify the Request

Identify the task category, then **use the Read tool** to load the corresponding reference file:

| Task Type | Action |
|-----------|--------|
| Clean code, organization, early return | Read `references/code-organization.md` |
| Code quality standards | Read `references/code-quality.md` |
| Models, ORM | Read `references/models-and-orm.md` |
| Views, URLs | Read `references/views-and-urls.md` |
| DRF (serializers, viewsets, errors) | Read `references/drf-guidelines.md` |
| Filters (django-filters) | Read `references/filters.md` |
| Testing | Read `references/testing-strategies.md` |
| Security | Read `references/security-checklist.md` |
| Performance | Read `references/performance-optimization.md` |
| Advanced patterns (services, repositories) | Read `references/advanced-patterns.md` |
| Constants and choices | Read `references/constants.md` |
| Celery tasks | Read `references/celery-patterns.md` |
| Django signals | Read `references/signals-guide.md` |
| Django Admin | Read `references/admin-guide.md` |
| Forms & ModelForms | Read `references/forms-modelforms.md` |
| Authentication | Read `references/authentication.md` |
| Middleware | Read `references/middleware.md` |
| Database migrations | Read `references/migrations.md` |
| Logging | Read `references/logging.md` |
| Static & media files | Read `references/static-media-files.md` |
| Project structure | Read `references/project-structure.md` |
| Third-party packages | Read `references/third-party-packages.md` |

For multi-category requests, read all relevant references.

### Step 2: Read Reference File(s)

**CRITICAL**: Always use the Read tool to load the relevant reference file(s) before implementing.

Example:
```
Read references/code-organization.md
```

Do NOT proceed without reading the appropriate documentation.

### Step 3: Implement

Apply patterns from references. Before presenting solutions, verify:

**Code Quality Checklist:**
- [ ] Uses early return pattern (no nested ifs)
- [ ] Avoids bool trap (uses enums/choices)
- [ ] Business logic in services, NOT in views
- [ ] Meaningful names for variables/functions
- [ ] No docstrings that just repeat function names
- [ ] Constants in separate file (not hardcoded)
- [ ] No imports inside functions/classes
- [ ] Follows Boy Scout Rule, DRY, KISS, YAGNI

**Django/DRF Checklist:**
- [ ] No N+1 queries (select_related/prefetch_related)
- [ ] Serializers explicitly list fields (no `__all__`)
- [ ] Permissions in settings, NOT in every view
- [ ] Status choices in constants.py
- [ ] UUID for public IDs, id for internal
- [ ] Database indexes on search fields
- [ ] Filters only when needed (else in settings)

**Security Checklist:**
- [ ] User input validated/sanitized
- [ ] Always use ORM (if RawSQL needed, sanitize)
- [ ] Minimum permissions
- [ ] No secrets in code

**Testing Checklist:**
- [ ] Unit tests for services
- [ ] FactoryBoy for test data
- [ ] Mock external services (requests.get/post)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aztrock/django-general-expert](https://github.com/aztrock/django-general-expert) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
