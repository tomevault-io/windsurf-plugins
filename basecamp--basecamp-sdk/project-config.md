---
trigger: always_on
description: | Component | Status | Details |
---

# Basecamp SDK Agent Guidelines

## Current Status

| Component | Status | Details |
|-----------|--------|---------|
| **Smithy Spec** | 175 operations | Single source of truth for all APIs |
| **Go SDK** | Production-ready | Full generated client + service wrappers |
| **TypeScript SDK** | Production-ready | 37 generated services, openapi-fetch based |
| **Ruby SDK** | Production-ready | 37 generated services |
| **Swift SDK** | Production-ready | 38 generated services, URLSession-based |
| **Kotlin SDK** | Production-ready | 38 generated services, Ktor/KMP-based |
| **Python SDK** | Production-ready | 40 generated services, httpx-based |

All six SDKs share the same architecture: **Smithy spec -> OpenAPI -> Generated services**. No hand-written API methods exist in any SDK runtime.

---

## Architecture

```
Smithy Spec → OpenAPI → Generated Client → Service Layer → User
```

| SDK | Generated Client | Service Layer |
|-----|-----------------|---------------|
| **Go** | `pkg/generated/client.gen.go` | `pkg/basecamp/*.go` (wraps generated client) |
| **TypeScript** | `openapi-fetch` + `schema.d.ts` | `src/generated/services/*.ts` |
| **Ruby** | HTTP client | `lib/basecamp/generated/services/*.rb` |
| **Swift** | `URLSession` via `Transport` protocol | `Sources/Basecamp/Generated/Services/*.swift` |
| **Kotlin** | Ktor via `BaseService` | `sdk/src/commonMain/kotlin/.../generated/services/*.kt` |
| **Python** | httpx via `HttpClient` | `src/basecamp/generated/services/*.py` |

All 175 operations across 38+ services are generated. Hand-written code is limited to infrastructure:

| Purpose | TypeScript | Ruby | Swift | Kotlin | Python |
|---------|-----------|------|-------|--------|--------|
| HTTP helpers, pagination, hooks | `src/services/base.ts` | `lib/basecamp/services/base_service.rb` | `Sources/Basecamp/Services/BaseService.swift` | `sdk/.../services/BaseService.kt` | `src/basecamp/generated/services/_base.py` |
| OAuth flows (not in OpenAPI spec) | `src/services/authorization.ts` | `lib/basecamp/services/authorization_service.rb` | — | `sdk/.../oauth/*.kt` | `src/basecamp/services/authorization.py` |

Other hand-written service files in `src/services/` (TS) and `lib/basecamp/services/` (Ruby) are NOT loaded at runtime. They exist only as reference implementations.

### Smithy Spec vs Actual API Responses

Smithy wrapper structures are a spec convention, not the API shape. The spec uses wrapper structures for list responses:

```smithy
structure ListAssignablePeopleOutput {
  people: PersonList
}
```

But the actual API returns top-level arrays. The Go code generator unwraps these:

```go
ListAssignablePeopleResponseContent = []Person
```

When verifying API response shapes, check Go generated code in `go/pkg/generated/client.gen.go` — look for `*ResponseContent` type definitions. Don't assume Smithy wrapper structures match the wire format.

**Why the wrappers exist:** Smithy's AWS restJson1 protocol requires list outputs to be wrapped structures because `@httpPayload` only supports string, blob, structure, union, and document types — not arrays directly. See the ARCHITECTURAL NOTE in `spec/basecamp.smithy`.

---

## Hard Rules

### Never Do These

1. **NEVER edit files under `*/generated/`** — they get overwritten by generators
2. **NEVER add hand-written service methods for API operations** — all API ops come from generators
3. **NEVER skip running `make smithy-build` after Smithy changes** — keeps OpenAPI in sync
4. **NEVER construct API paths manually** — use the generated client methods
5. **NEVER bypass the SDK** — no raw `client.Get()`, string-concatenated URLs, or internal method calls

If you're writing `fmt.Sprintf` with an API path, you're doing it wrong. If the generated client lacks functionality, fix the spec and regenerate — don't work around it.

### Anti-patterns

```go
// WRONG - Manual path construction
path := fmt.Sprintf("/buckets/%d/todolists/%d/todos.json", bucketID, todolistID)

// WRONG - Query parameter hacks
path := generatedPath + "?status=active"

// WRONG - "Just this once" shortcuts
path := fmt.Sprintf("/projects/%d/people.json", projectID)
```

### Correct Patterns

```go
// Single-resource: use generated client directly
resp, err := client.gen.GetTodoWithResponse(ctx, accountID, bucketID, todoID)

// Paginated: generated client for first page, Link headers for subsequent
resp, err := client.gen.ListTodosWithResponse(ctx, accountID, bucketID, todolistID, params)
nextURL := parseNextLink(resp.HTTPResponse.Header.Get("Link"))
for nextURL != "" {
    resp, err := client.Get(ctx, nextURL)  // URL from API, not constructed
    nextURL = parseNextLink(resp.Headers.Get("Link"))
}
```

```python
# Python — single resource
todo = account.todos.get(todo_id=123)

# Python — paginated (automatic)
todos = account.todos.list(todolist_id=456, status="active")

# WRONG — manual path construction
url = f"/buckets/{project_id}/todolists/{todolist_id}/todos.json"

# WRONG — bypassing the SDK
response = account.http.get(f"/{account_id}/buckets/{project_id}/todos.json")
```

### Andon Cord — Stop and Fix Immediately

Pull the andon cord when you see:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [basecamp/basecamp-sdk](https://github.com/basecamp/basecamp-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
