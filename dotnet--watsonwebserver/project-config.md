---
trigger: always_on
description: This file provides repository-specific guidance for AI coding agents working in this codebase.
---

# CLAUDE.md

This file provides repository-specific guidance for AI coding agents working in this codebase.

## Project Overview

Watson 7 is a transport-owning HTTP server with support for HTTP/1.1, HTTP/2, and HTTP/3.

The repository ships a single `Watson` package. The `WatsonWebserver.Core` namespace within that package contains the shared abstractions, routing, OpenAPI, health checks, settings, serialization, and API-route support.

This repository no longer uses `http.sys` as its primary server model.

## Solution Layout

- `src/WatsonWebserver/` - server implementation (concrete types at root, shared core types under `Core/`)
- `src/Test.Automated/` - console-based automated integration suite
- `src/Test.XUnit/` - xUnit mirror over the shared automated coverage
- `src/Test.Benchmark/` - benchmark harness for Watson 6, WatsonLite 6, Watson 7, and Kestrel
- `src/Test.*` - sample and feature-specific projects

## Core Architecture

### Shared request pipeline

The `WatsonWebserver.Core` namespace (under `src/WatsonWebserver/Core/`) owns the common semantics:

- `WebserverBase`
- `HttpContextBase`
- `HttpRequestBase`
- `HttpResponseBase`
- route managers and routing groups
- middleware execution
- API-route helpers
- OpenAPI and health-check extensions

The concrete transport work lives under `src/WatsonWebserver/`:

- HTTP/1.1
- HTTP/2
- HTTP/3

### Routing order

Requests flow through this order:

1. `Routes.Preflight`
2. `Routes.PreRouting`
3. `Routes.PreAuthentication`
4. `Routes.AuthenticateRequest`
5. `Routes.AuthenticateApiRequest`
6. `Routes.PostAuthentication`
7. `Routes.Default`
8. `Routes.PostRouting`

Within a routing group, matching order is:

1. `Static`
2. `Content`
3. `Parameter`
4. `Dynamic`

### Middleware behavior

`WebserverBase.Middleware` wraps the matched route handler and executes in registration order.

- Middleware runs for all route types, not only API routes.
- Middleware can short-circuit by not calling `next()`.
- `PostRouting` still runs after the route pipeline completes.

## Current package and protocol model

When updating docs or examples, assume:

- `Watson` is the single package consumers install (there is no separate `Watson.Core` package as of v7.0.4)
- HTTP/1.1 is enabled by default
- HTTP/2 and HTTP/3 require explicit configuration
- Alt-Svc is explicit and off by default

Do not write new guidance that describes Watson 7 as `HttpListener` or `http.sys` based.

## API Routes

Watson 7 supports FastAPI-like API routes directly on `WebserverBase`.

### Basic usage

```csharp
WebserverSettings settings = new WebserverSettings("127.0.0.1", 8080, false);
Webserver server = new Webserver(settings, DefaultRoute);

server.Get("/users/{id}", async (req) =>
{
    Guid id = req.Parameters.GetGuid("id");
    return new UserResponse
    {
        Id = id,
        Name = "example"
    };
});

server.Post<CreateUserRequest>("/users", async (req) =>
{
    CreateUserRequest body = req.GetData<CreateUserRequest>();
    req.Http.Response.StatusCode = 201;

    return new UserResponse
    {
        Id = Guid.NewGuid(),
        Name = body.Name
    };
});
```

### Body access rules

For API routes:

- `Post<T>`, `Put<T>`, and `Patch<T>` deserialize into `ApiRequest.Data`
- `req.GetData<T>()` is the normal typed access path
- manual access is still available through `req.Http.Request`

For low-level routes:

- `ctx.Request.DataAsBytes` fully reads and caches the body on first access
- `ctx.Request.DataAsString` fully reads and caches the body on first access
- `await ctx.Request.ReadBodyAsync(ctx.Token)` is the explicit async read path when cancellation-aware body consumption is preferred

### Error handling

Use `WebserverException` from API routes for structured JSON errors.

```csharp
server.Get("/products/{id}", async (req) =>
{
    Guid id = req.Parameters.GetGuid("id");
    throw new WebserverException(ApiResultEnum.NotFound, "Product not found: " + id);
});
```

### Serializer configuration

`WebserverBase.Serializer` controls API-route request/response serialization.

- Default: `DefaultSerializationHelper`
- Replaceable by user code
- Used by `ApiRouteHandler` and `ApiResponseProcessor`

```csharp
server.Serializer = new DefaultSerializationHelper();
```

If you change API-route serialization behavior, check both:

- `src/WatsonWebserver/Core/Routing/ApiRouteHandler.cs`
- `src/WatsonWebserver/Core/Routing/ApiResponseProcessor.cs`

### Structured authentication

Use `Routes.AuthenticateApiRequest` for structured auth.

```csharp
server.Routes.AuthenticateApiRequest = async (ctx) =>
{
    string header = ctx.Request.RetrieveHeaderValue("Authorization");
    if (header == "Bearer test-token")
    {
        return new AuthResult
        {
            AuthenticationResult = AuthenticationResultEnum.Success,
            AuthorizationResult = AuthorizationResultEnum.Permitted,
            Metadata = new AuthMetadata
            {
                UserId = 1
            }
        };
    }

    return new AuthResult
    {
        AuthenticationResult = AuthenticationResultEnum.NotFound,
        AuthorizationResult = AuthorizationResultEnum.DeniedImplicit
    };
};
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dotnet/WatsonWebserver](https://github.com/dotnet/WatsonWebserver) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
