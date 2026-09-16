---
trigger: always_on
description: Heimly is a property listing platform with verify-by-default model. We verify owner identity (KYC), property documents, and all related documents before listings go live. Target market: Nigeria/Africa.
---

# Heimly Property Listing Platform - Cursor AI Rules

## Project Context
Heimly is a property listing platform with verify-by-default model. We verify owner identity (KYC), property documents, and all related documents before listings go live. Target market: Nigeria/Africa.

## Core Philosophy: Django-First, External-Last

**CRITICAL RULE**: Use Django's built-in batteries before suggesting external services. any installation that is needed, output the instructions and the command.
DO NOT INSTALL ANYTHING YOURSELF 

### What Django Provides (Use These First):
- ✅ Built-in authentication system (NOT Supabase Auth)
- ✅ Built-in admin interface (NOT custom admin panels)
- ✅ Built-in forms system (NOT raw HTML forms)
- ✅ Built-in template engine
- ✅ Built-in messages framework
- ✅ Built-in file upload handling (FileField/ImageField)
- ✅ Built-in email backend (console for dev, SMTP for prod)

### External Services (ONLY When Forced):
- Supabase Postgres (only when deploying - Railway doesn't persist SQLite)
- Supabase Storage (only when deploying - Railway doesn't persist files)
- SMTP provider (Gmail/other - only when need real emails)

### What We DON'T Need (MVP):
❌ Supabase Auth, Realtime, Edge Functions
❌ Redis, Celery, Task queues
❌ React/Vue - Django templates are fine
❌ Docker, Kubernetes
❌ GraphQL, JWT tokens, OAuth
❌ AI/ML services for face verification

## Engineering Principles (MANDATORY)

### 1. O(1) Performance Mindset
- **Database operations MUST be O(1) or O(log n) at worst**
- Always add `db_index=True` to frequently queried fields: `owner_id`, `status`, `city`, `id_number`
- Use `select_related()` and `prefetch_related()` to eliminate N+1 queries
- Example: Dashboard views must load all data in ≤3 queries using prefetch
- Core operations (save draft, toggle status, fetch dashboard) must be constant time

### 2. World-Class Code Quality
- **Type hints**: Use them for all function signatures, especially service methods
- **Docstrings**: Every class, function, and complex method must have clear docstrings
- **Linting**: Code must pass flake8/black/isort checks
- **Tests**: Critical verification flows MUST have tests (even basic smoke tests)
- **Meaningful names**: `submit_for_review()` not `do_stuff()`
- Code quality standards apply from day one, even in local development

### 3. Deterministic Processes
- Use explicit state machines/services, NOT ad-hoc conditionals
- Status transitions must go through service methods (e.g., `VerificationService.submit_listing()`)
- All state changes must be auditable (use AuditEntry model)
- Avoid magic strings - use model enums/choices
- Example: `listing.status = ListingStatus.VERIFIED` not `listing.status = 'verified'`

## Code Style & Structure

### Models
- Use `Meta.indexes` for composite indexes on commonly filtered fields
- Use `Meta.constraints` for unique constraints (e.g., one primary photo per listing)
- Always include `created_at` and `updated_at` timestamps
- Use `db_index=True` on foreign keys used in filters
- Use TextChoices for status fields (e.g., `class ListingStatus(models.TextChoices)`)

### Views
- Prefer class-based views (CreateView, UpdateView, DetailView) over function views
- Use `@login_required` or `LoginRequiredMixin`
- Use `get_queryset()` to filter by user ownership
- Always use `select_related()` and `prefetch_related()` in querysets
- Return structured errors, not generic messages

### Forms
- Use `ModelForm` when possible - Django generates 90% of form code
- Custom validation in `clean_<field>()` methods
- Use `forms.Textarea`, `forms.CheckboxSelectMultiple` for UX
- File uploads: validate file type/size in form, not just model

### Templates
- Use template inheritance (`{% extends 'base.html' %}`)
- Use Django's `{% csrf_token %}` (never skip CSRF)
- Use Django messages framework (`{% if messages %}`)
- Keep templates DRY - extract repeated blocks

### Services
- Create service classes for complex business logic (e.g., `VerificationService`)
- Services handle state transitions, validation, notifications
- Keep views thin - delegate to services
- Services must be testable in isolation

## Verification Flow Requirements

### Owner KYC (MVP - Simplified)
- Government ID (NIN, Passport, Driver's License) - front photo only
- Email verification OR Phone verification (at least one)
- Full name matching ID
- NO selfie/face verification (too complex for MVP)
- NO bank statements (too stiff for MVP)
- NO handwritten codes (too much friction)

### Property Verification
- Required documents: C of O, Deed, Utility Bill, Tax Receipt (upload what you have)
- Photos: Minimum 3-5 photos (one primary)
- Location: Address required, GPS optional but preferred
- Manual review by staff via Django admin

### Listing Status Flow
1. `draft` - User creating, can save incomplete
2. `pending_identity` - Draft saved, owner KYC incomplete
3. `pending_documents` - Identity OK, documents incomplete
4. `in_review` - All requirements met, under staff review
5. `verified` - Approved, goes live with full visibility
6. `rejected` - Rejected (with reason)
7. `archived` - User/admin archived

## File Storage


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Talktodre-ops/django-for-property](https://github.com/Talktodre-ops/django-for-property) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
