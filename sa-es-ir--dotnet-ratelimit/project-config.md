---
trigger: always_on
description: DotNet.RateLimit is a .NET library that provides rate limiting functionality for ASP.NET Core applications using ActionFilters and EndPointFilters. It supports both in-memory caching and Redis for distributed scenarios, and works with both traditional controllers and minimal APIs.
---

# DotNet.RateLimit
DotNet.RateLimit is a .NET library that provides rate limiting functionality for ASP.NET Core applications using ActionFilters and EndPointFilters. It supports both in-memory caching and Redis for distributed scenarios, and works with both traditional controllers and minimal APIs.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

### Install Required .NET Versions
Always install both .NET 9.0 SDK and .NET 8.0 runtime components first:
- `curl -sSL https://dot.net/v1/dotnet-install.sh | bash /dev/stdin --channel 9.0`
- `curl -sSL https://dot.net/v1/dotnet-install.sh | bash /dev/stdin --channel 8.0 --runtime dotnet`
- `curl -sSL https://dot.net/v1/dotnet-install.sh | bash /dev/stdin --channel 8.0 --runtime aspnetcore`
- `export PATH="$HOME/.dotnet:$PATH"` (required for each terminal session)

### Bootstrap, Build and Test the Repository
- `dotnet restore` -- takes 20-25 seconds. NEVER CANCEL. Set timeout to 3+ minutes.
- `dotnet build` -- takes 10 seconds with 2 warnings (expected). NEVER CANCEL. Set timeout to 3+ minutes.
- `dotnet test` -- takes 10-15 seconds, runs 66 tests. NEVER CANCEL. Set timeout to 3+ minutes.
  - All tests pass successfully, validating rate limiting behavior works correctly
  - Tests verify rate limiting properly returns 429 status codes when limits are exceeded
  - Tests use Docker containers (via Testcontainers) for Redis integration tests
- `dotnet pack src/DotNet.RateLimiter/DotNet.RateLimiter.csproj -c Release` -- takes 4 seconds for packaging

### Run the Demo Application
- ALWAYS run the bootstrapping steps first (install .NET, restore packages)
- `cd demo/DotNet.RateLimiter.Demo && dotnet run`
- Application runs on:
  - HTTP: http://localhost:5196
  - HTTPS: https://localhost:7196 (with untrusted dev certificate)
- Swagger UI available at: https://localhost:7196/swagger/index.html

## Validation
- ALWAYS manually validate rate limiting functionality after making changes
- Test rate limiting endpoints:
  - `curl -k https://localhost:7196/rate-limit-on-actions` (first 3 requests should return 200)
  - `curl -k https://localhost:7196/rate-limit-on-actions` (4th+ requests should return 429)
- ALWAYS test with the demo application to ensure end-to-end functionality works
- ALWAYS run all tests with `dotnet test` and verify all tests pass successfully

## Common Tasks

### Project Structure
The repository contains:
- `src/DotNet.RateLimiter/` - Main library project (multi-targets: netstandard2.0, netstandard2.1, net8.0, net9.0)
- `test/DotNet.RateLimiter.Test/` - Unit tests (targets net8.0)
- `demo/DotNet.RateLimiter.Demo/` - Demo web application (targets net9.0)
- `DotNet.RateLimit.sln` - Solution file with all projects

### Key Configuration Files
- `src/DotNet.RateLimiter/DotNet.RateLimiter.csproj` - Main library project file with multi-targeting
- `demo/DotNet.RateLimiter.Demo/appsettings.json` - Demo app configuration with RateLimitOption settings
- `.github/workflows/package.yml` - CI/CD pipeline for packaging and releases
- `azure-pipelines.yml` - Azure DevOps CI/CD pipeline configuration

### Configuration Options (appsettings.json)
```json
"RateLimitOption": {
    "EnableRateLimit": true,           // Optional: default is true
    "HttpStatusCode": 429,              // Optional: default is 429
    "ErrorMessage": "Rate limit Exceeded", // Optional
    "IpHeaderName": "X-Forwarded-For",  // Optional: default is X-Forwarded-For
    "RedisConnection": "127.0.0.1:6379", // Optional: for distributed scenarios
    "IpWhiteList": ["::1"],             // Optional: IPs to exclude from rate limiting
    "ClientIdentifier": "X-Client-Id",  // Optional: use Client ID instead of IP
    "ClientIdentifierWhiteList": ["test-client"], // Optional
    "ResponseStructure": "{\"error\": {\"message\": \"$(ErrorMessage)\", \"code\": $(HttpStatusCode)}}" // Optional: custom response format
}
```

### Build Requirements
- .NET 9.0 SDK required for building all targets
- .NET 8.0 runtime required for running tests (ASP.NET Core 8.0)
- Multi-target build supports: netstandard2.0, netstandard2.1, net8.0, net9.0

### Rate Limiting Features Tested
- IP-based rate limiting (default)
- Route parameter-based rate limiting (`RouteParams`)
- Query parameter-based rate limiting (`QueryParams`)  
- Body parameter-based rate limiting (`BodyParams`)
- Controller-level rate limiting (`RateLimitScope.Controller`)
- White-listing (IP and Client ID)
- Redis distributed caching support
- In-memory caching (default)
- Custom response structures with placeholders (`$(ErrorMessage)`, `$(HttpStatusCode)`)

### Key Features
- **ActionFilters and EndpointFilters**: Works with both traditional controllers and minimal APIs (.NET 7+)
- **Redis Support**: Use existing Redis connections or create new ones via configuration
- **Flexible Rate Limiting**: Based on IP, Client ID, route params, query params, or body params

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sa-es-ir/DotNet.RateLimit](https://github.com/sa-es-ir/DotNet.RateLimit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
