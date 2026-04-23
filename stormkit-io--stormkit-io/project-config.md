---
trigger: always_on
description: - Handler files: `handler_<resource>_<action>.go` (e.g. `handler_app_create.go`).
---

# Coding Conventions

## Go

### Package & File Layout

- Handler files: `handler_<resource>_<action>.go` (e.g. `handler_app_create.go`).
- Every handler file has a sibling test file: `handler_<resource>_<action>_test.go`.
- Core / utility files: `request_context.go`, `services.go`, `validators.go`.
- Routes are declared in `services.go`

### Handler Signature

```go
func handler<Name>(req *RequestContext) *shttp.Response
```

- Always takes `*RequestContext` (or `*RequestContextMCP` for MCP tool wrappers).
- Always returns `*shttp.Response` — never a `(value, error)` tuple.

### Request Parsing

**Path parameters** — use compact `if` with initializer:

```go
if id := utils.StringToID(req.Vars()["id"]); id == 0 {
    return shttp.NotFound()
}
```

**Query parameters** — use `Validators` helpers:

```go
v := &Validators{}
errs := []string{}

if from, err := v.ToInt(req.Query().Get("from"), "from"); err != nil {
    errs = append(errs, err.Error())
}

if repo, ok := v.NormalizeRepo(req.Query().Get("repo")); !ok {
    errs = append(errs, "repo must match 'provider/org/repo' format")
}

if len(errs) > 0 {
    return shttp.BadRequest(map[string]any{"errors": errs})
}
```

**JSON body**:

```go
type myResource struct {
    Name string  `json:"name"`
    Note *string `json:"note,omitempty"` // pointer for optional/PATCH fields
}

data := &myResource{}

if err := req.Post(data); err != nil {
    return shttp.Error(err)
}
```

Use pointer fields for optional `PUT`/`PATCH` fields so absence can be distinguished from zero value.

**Multipart form**:

```go
if req.MultipartForm == nil {
    return shttp.BadRequest(map[string]any{"errors": []string{"Expected multipart/form-data"}})
}

files := req.MultipartForm.File["files"]
```

### Responses

Use `shttp` helpers:

```go
shttp.OK()                     // 200, empty body
shttp.Error(err)               // 500, logs error
shttp.Error(err, "msg")        // 500, custom log message
shttp.BadRequest(data)         // 400
shttp.NotFound()               // 404
shttp.Forbidden()              // 403
shttp.NotAllowed()             // 401
```

Use a literal `*shttp.Response` when you need a specific status code:

```go
return &shttp.Response{
    Status: http.StatusCreated,
    Data: map[string]any{
        "id": resource.ID
    },
}
```

**Validation errors** should follow the existing response shapes in this package. For a single validation error, use an `"error"` string; when returning multiple messages, use an `"errors"` key with a string slice:

```go
// Single validation error (most common for param/query failures)
return shttp.BadRequest(map[string]any{
    "error": "Name is required",
})

// Multiple validation errors (domain/model validation)
return shttp.BadRequest(map[string][]string{
    "errors": {"Name is required", "Branch is invalid"},
})
```

**Duplicate / conflict**:

```go
if err := store.Insert(ctx, data); err != nil {
    if database.IsDuplicate(err) {
        return &shttp.Response{
            Status: http.StatusConflict,
            Data:   map[string][]string{"errors": {"Resource already exists"}},
        }
    }

    return shttp.Error(err)
}
```

### Route Registration (`services.go`)

```go
func Services(r *shttp.Router) *shttp.Service {
    s := r.NewService()

    s.NewEndpoint("/v1/app").
        Handler(shttp.MethodPost, "", WithAPIKey(handlerAppCreate, &Opts{MinimumScope: apikey.SCOPE_TEAM})).
        Handler(shttp.MethodGet,  "", WithAPIKey(handlerAppGet,    &Opts{MinimumScope: apikey.SCOPE_APP}))

    // Sub-path as the second argument to Handler
    s.NewEndpoint("/v1/app").
        Handler(shttp.MethodGet, "/config", WithAPIKey(handlerAppConf, &Opts{MinimumScope: apikey.SCOPE_APP}))

    // Path parameter (gorilla mux regex)
    s.NewEndpoint("/v1/deployments/{id:[0-9]+}").
        Handler(shttp.MethodGet, "", WithAPIKey(handlerDeploymentGet, &Opts{MinimumScope: apikey.SCOPE_ENV}))

    // Endpoint-level middleware (e.g. enterprise gate, upload limits)
    s.NewEndpoint("/v1/domains").
        Middleware(user.WithEE).
        Handler(shttp.MethodPut, "/cert", WithAPIKey(...))

    return s
}
```

- Prefer a single `NewEndpoint()` per base path; chain `.Handler()` for multiple HTTP methods.
- It is acceptable to register the same base path with multiple `NewEndpoint()` calls when endpoint-level middleware must differ (e.g. `/v1/domains` has a second registration with `Middleware(user.WithEE)` for the EE-only `/cert` sub-paths).
- Always wrap handlers with `WithAPIKey(handler, &Opts{MinimumScope: <scope>})`.

### Authentication & Scope

Scope constants (`apikey` package) represent the type of identifier a key carries. Each key has access only to the object it was created for:

| Constant            | Key must carry                                       |
| ------------------- | ---------------------------------------------------- |
| `apikey.SCOPE_USER` | UserID — access to everything the user has access to |
| `apikey.SCOPE_TEAM` | TeamID (or user who is a team member)                |
| `apikey.SCOPE_APP`  | AppID (or user/team with app access)                 |
| `apikey.SCOPE_ENV`  | EnvID — access to that environment only              |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stormkit-io/stormkit-io](https://github.com/stormkit-io/stormkit-io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
