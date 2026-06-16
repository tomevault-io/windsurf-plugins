---
trigger: always_on
description: Open-source LMS. Backend: Django (async) + Django Ninja. Frontend: SolidJS + TanStack Router.
---

# Minima LMS

Open-source LMS. Backend: Django (async) + Django Ninja. Frontend: SolidJS + TanStack Router.
Multi-tenant by subdomain: `student`, `studio`, `tutor`, `desk`, `preview`.

## Dev

```bash
sh dev.sh up          # start all services (Docker Compose)
uv run dev.py lint    # ruff check + format + pyrefly type check
cd web && npm run lint # biome
```

Django :8000 · MinIO :9001 · Mailpit :8025 · nginx subdomain routing

## Architecture

**API auto-discovery:** `minima/api.py` scans `apps/{app}/api/v1.py` for `router` and mounts at `/api/v1/{app}`.

**Auth:** `request.auth` = user ID string · `request.roles` = list[str] · realm mismatch = 401

**Roles:** `editor` (studio) · `grader` · `partner_staff`

**Content `context`** (on `Watch` and attempt models) — isolates activity per session. Format `model::model_id::session_id` (e.g. `course::<course_id>::<engagement_id>`, built by `issue_active_context()`); empty `""` = standalone access (watched/attempted directly, outside any course session). `normalize_context()` drops `session_id` (`course::id::sess` → `course=id`) to dedup across sessions of the same parent. `(user, media, context)` is unique, so **filtering `context=""` already yields one row per (user, media) — no `DISTINCT ON` / normalization / dedup needed.** Only reach for those when a query legitimately spans non-empty contexts.

**Global exception handlers** (`minima/api.py`) — never handle these manually in views or models:

| Exception | Status |
|-----------|--------|
| `ValueError(MessageCode.X)` | 400 |
| `ValidationError` | 400 |
| `ObjectDoesNotExist` / `Http404` | 404 |
| `AuthenticationError` | 401 |
| `Throttled` | 429 |

---

## Before Writing Any Code

1. Read the relevant model files in full
2. Map every DB relationship that the feature touches
3. Write the queryset shape (what `select_related` / `prefetch_related` you'll need) before writing any logic
4. Only then write the implementation

This order eliminates N+1s and structural rewrites after the fact.

---

## Naming

**Names must read like natural English** — a noun, not a noun decorated with metadata.

**DateTime fields** — no `_at` suffix. The name itself is the event:
```python
published   # not published_at
completed   # not completed_at
confirmed   # not confirmed_at
deleted     # not deleted_at
started     # not started_at
lock        # a deadline point in time, concise
```

**Boolean fields** — no `is_` prefix unless the word would be ambiguous without it:
```python
active      # not is_active
featured    # not is_featured
passed      # not is_passed
verified    # not is_verified
public      # not is_public
```
Exception: Django's own `User.is_active`, `User.is_staff`, `User.is_superuser` — leave these alone.

**Variables** — full words, never abbreviations:
```python
exam          # not e, ex, exam_obj, exam_instance
template_channel  # not tc
channel_impl      # not ch, ci
content_type      # not ct
enrollment        # not enr, enrl
```

**FK identifier args/fields** — `_id` suffix is Django convention, keep it:
```python
learner_id, owner_id, exam_id, context_id  # fine
```

**Local variables** — short but complete. One word is ideal, two is fine, three is a smell:
```python
attempt = await Attempt.objects.aget(...)   # good
existing_attempt = ...                       # acceptable when disambiguating
existing_active_attempt_for_user = ...       # too long — restructure instead
```

---

## Imports

Top-level only. Never import inside a function body unless the import is circular and unavoidable.

**Circular imports** — comment must show the full cycle:
```python
from apps.partner.models import Member  # circular: account → partner → account
```

**Type-only imports** — put in `TYPE_CHECKING` block:
```python
from typing import TYPE_CHECKING
if TYPE_CHECKING:
    from apps.learning.models import Enrollment
```

Never use a function-level import to work around a circular dependency that could be resolved with `TYPE_CHECKING` or `apps.get_model()`.

---

## Types

**Annotate for the reader, not the type checker.** If pyrefly already knows the type, don't repeat it.

```python
# wrong — checker knows this is list[str]
result: list[str] = []

# wrong — pyrefly infers return type of cls.objects.aget() chain
async def get_for_edit(cls, *, id: str, owner_id: str) -> "Exam": ...

# right — TypedDict fields always annotated (that's their purpose)
class SessionDict(TypedDict):
    step: LearningSessionStep
    attempt: NotRequired["Attempt"]

# right — annotate where the return type isn't obvious from the body
async def get_session(cls, ...) -> SessionDict:
    ...

# right — annotate when narrowing from a broad return type
exam: Exam = await cls.objects.aget(id=exam_id)
```

Rules:
- `TypedDict` fields: always annotate
- Function signatures: annotate params and return when not obvious
- `cls.objects.aget()` / `cls.objects.select_related(...).aget()` return types: **never annotate** — pyrefly fully infers
- Local variables: only when the type is not immediately obvious from the RHS
- No `Any` without a comment explaining why
- `TypedDict` + `NotRequired[T]` for optional keys in complex return shapes
- `TYPE_CHECKING` guard for forward refs and FK `_id` field annotations on models


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cobel1024/minima](https://github.com/cobel1024/minima) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
