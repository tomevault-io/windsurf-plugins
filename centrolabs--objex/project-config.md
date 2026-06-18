---
trigger: always_on
description: Self-hosted blob storage built with Clean Architecture in .NET 10.
---

# ObjeX — AI Context

Self-hosted blob storage built with Clean Architecture in .NET 10.

---

## Project Layout

```
src/
├── ObjeX.Api/           # ASP.NET Core host — Program.cs, Endpoints/, Middleware/, Auth/
│   ├── Endpoints/       # AccountEndpoints, DownloadEndpoints, PresignEndpoints
│   │   └── S3Endpoints/ # S3BucketEndpoint, S3ObjectEndpoint, S3MultipartEndpoint, S3PostObjectEndpoint
│   ├── Middleware/      # SigV4AuthMiddleware
│   ├── Auth/            # HangfireAuthorizationFilter
│   ├── S3/              # SigV4Parser, SigV4Signer, S3Xml, S3Errors, StorageQuota
│   └── Metrics/         # ObjeXMetrics, BucketMetricsSyncJob
├── ObjeX.Core/          # Domain — zero framework dependencies
│   ├── Interfaces/      # IMetadataService, IObjectStorageService, IHashService, IHasTimestamps
│   ├── Models/          # Bucket, BlobObject, S3Credential, User, AuditEntry, ListObjectsResult, MultipartUpload, MultipartUploadPart, SystemSettings
│   ├── Utilities/       # HashingStream (MD5 passthrough for ETag computation during upload), PresignedUrlGenerator
│   └── Validation/      # BucketNameValidator (GetValidationError)
├── ObjeX.Infrastructure/
│   ├── Data/            # ObjeXDbContext (EF Core + SQLite, extends IdentityDbContext<User>)
│   ├── Hashing/         # Sha256HashService
│   ├── Health/          # BlobStorageHealthCheck
│   ├── Jobs/            # CleanupOrphanedBlobsJob, VerifyBlobIntegrityJob, CleanupAbandonedMultipartJob (Hangfire job classes)
│   ├── Metadata/        # SqliteMetadataService
│   ├── Migrations/      # EF Core migrations
│   └── Storage/         # FileSystemStorageService
├── ObjeX.Migrations.PostgreSql/  # PostgreSQL-specific EF Core migrations
├── ObjeX.Tests/         # xUnit — unit (Core validators, hashing) + integration (WebApplicationFactory, real SQLite)
│   ├── Unit/            # BucketNameValidator, ObjectKeyValidator, HashingStream, Sha256HashService
│   └── Integration/     # S3 API round-trips, auth, multipart, quotas, resilience, cookie auth, health
└── ObjeX.Web/           # Blazor Server UI — components, pages, dialogs
    ├── Helpers/         # FileHelper
    └── Components/
        ├── Pages/       # Dashboard, Buckets, Objects, Settings, Login, NotFound, Users, ChangePassword, AuditLog, Error, Profile
        ├── Dialogs/     # CreateBucketDialog, UploadObjectDialog, CreateS3CredentialDialog, ShowS3CredentialDialog, CreateFolderDialog, CreateUserDialog, ShowUserPasswordDialog, ChangeOwnerDialog, FilePreviewDialog, FileMetadataDialog, PresignedUrlDialog
        └── Layout/      # MainLayout, NavMenu, EmptyLayout
```

---

## Architecture Rules

- **ObjeX.Core** has zero framework/NuGet dependencies — only BCL. Keep it that way.
- **ObjeX.Infrastructure** implements Core interfaces. Never reference Api or Web.
- **ObjeX.Api** wires everything together via DI in `Program.cs`. No business logic here.
- **ObjeX.Web** references both `ObjeX.Core` and `ObjeX.Infrastructure` (for `ObjeXDbContext` injection in Blazor components).
- New storage backends → implement `IObjectStorageService`. New metadata stores → implement `IMetadataService`. No other changes needed.

---

## Authentication & Authorization

### Overview — Dual Auth

ObjeX uses **two authentication mechanisms** operating independently:

| Mechanism | Scheme name | Used by |
|---|---|---|
| Cookie (ASP.NET Core Identity) | `Identity.Application` | Browser / Blazor UI |
| AWS Signature V4 | `"SigV4"` (custom middleware) | S3 clients on port 9000 |

The cookie is the default for the browser. S3 clients on port 9000 authenticate via AWS Sig V4 — no cookie, no `X-API-Key`.

### Middleware Pipeline Order

```
UseWhen(!api)              ← UseStatusCodePagesWithRedirects("/not-found") — only non-API paths
UseStaticFiles
UseWhen(port == 9000)      ← UseCors("S3") — permissive CORS for S3 clients only; port 9001 has no CORS (same-origin)
UseRateLimiter
app.Use(...)               ← security headers (X-Content-Type-Options, X-Frame-Options, etc.)
UseAuthentication          ← runs Identity cookie handler, sets context.User for cookie sessions
UseWhen(port == 9000)      ← SigV4AuthMiddleware: validates Sig V4, sets context.User for S3 clients
UseAuthorization           ← enforces policies on the already-resolved context.User
UseAntiforgery
```

### HTTP Security Headers

Set in a raw `app.Use` middleware in `Program.cs` (after `UseCors`, before auth):

| Header | Value | Condition |
|--------|-------|-----------|
| `Server` | removed | `AddServerHeader = false` on Kestrel |
| `X-Powered-By` | removed | `ctx.Response.Headers.Remove(...)` |
| `X-Content-Type-Options` | `nosniff` | always |
| `X-Frame-Options` | `SAMEORIGIN` | always |
| `X-Permitted-Cross-Domain-Policies` | `none` | always |
| `Referrer-Policy` | `strict-origin-when-cross-origin` | always |
| `Strict-Transport-Security` | `max-age=63072000; includeSubDomains` | non-dev only |

CSP is intentionally omitted — Blazor Server requires inline scripts and a SignalR WebSocket (`ws://`/`wss://`), making a safe policy non-trivial. Deferred.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [centrolabs/ObjeX](https://github.com/centrolabs/ObjeX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
