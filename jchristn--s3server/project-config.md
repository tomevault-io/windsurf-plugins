---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

S3Server is a .NET library that provides a server-side interface for emulating AWS S3 operations. It's a NuGet package that allows developers to build S3-compatible storage servers (see [Less3](https://github.com/jchristn/less3) for a full implementation).

The library parses incoming S3 HTTP requests and routes them to callback methods that developers must implement. It does NOT provide the storage logic itself - that's the responsibility of the consuming application.

## Build and Test Commands

### Build the solution
```bash
dotnet build src/S3Server.sln
```

### Build specific configuration
```bash
dotnet build src/S3Server.sln -c Release
dotnet build src/S3Server.sln -c Debug
```

### Build the main library project
```bash
dotnet build src/S3Server/S3Server.csproj
```

### Run test server (must run as administrator on Windows for wildcard listeners)
```bash
dotnet run --project src/Test.Server/Test.Server.csproj
```

### Run test client
```bash
dotnet run --project src/Test.Client/Test.Client.csproj
```

### Run automated tests (runs against both HTTP and TCP server modes)
```bash
dotnet run --project src/Test.Automated/Test.Automated.csproj
```

### Pack NuGet package
```bash
dotnet pack src/S3Server/S3Server.csproj -c Release
```

## Architecture

### Core Request Flow

1. **S3Server** (S3Server.cs:19) - Main entry point that wraps WatsonWebserver
   - Listens for HTTP requests via Watson webserver
   - Routes all requests to `RequestHandler` method (line 181)
   - Manages lifecycle (Start/Stop/Dispose)

2. **S3Context** (S3Context.cs:11) - Request context passed to all callbacks
   - Contains `S3Request`: Parsed S3 request details
   - Contains `S3Response`: Response object for sending data back
   - Contains `Http`: Underlying HTTP context from WatsonWebserver
   - Contains `Metadata`: User-defined metadata (useful for PreRequestHandler auth)

3. **S3Request** (S3Request.cs:16) - Parsed S3 request information
   - Determines `RequestType` (bucket vs object operation)
   - Determines `RequestStyle` (path-style vs virtual-hosted)
   - Extracts bucket name, object key, query parameters
   - Parses AWS signature information

4. **Callback System** - Three categories of callbacks:
   - `ServiceCallbacks` (ServiceCallbacks.cs:10): Service-level operations (list buckets, service exists)
   - `BucketCallbacks`: Bucket operations (create, delete, read, ACLs, tags, etc.)
   - `ObjectCallbacks`: Object operations (read, write, delete, multipart, ACLs, etc.)

### Request Handling Flow

```
HTTP Request → S3Server.RequestHandler()
  → S3Context created (wraps HTTP context)
  → S3Request parses request (determines type, style, bucket, key)
  → PreRequestHandler (optional - for auth/logging)
  → Signature validation (if enabled)
  → Switch on RequestType → Invoke appropriate callback
  → Callback returns data or throws S3Exception
  → S3Response sends HTTP response
  → PostRequestHandler (optional - for logging/metrics)
```

### Request Types and URL Patterns

**Path-style URLs** (default): `http://host:port/bucket/key`
**Virtual-hosted URLs**: `http://bucket.basedomain/key` (requires `FindMatchingBaseDomain` callback)

The library automatically detects which style is being used based on the hostname and configured base domains.

## Key Implementation Details

### S3ServerSettings Configuration

Settings must be configured BEFORE starting the server:

- `Webserver`: WebserverSettings from Watson (hostname, port, SSL)
- `Logger`: Action<string> for logging
- `Logging`: Toggle logging for HTTP requests, S3 requests, signatures
- `EnableSignatures`: Enable AWS signature V4 validation
- `UseTcpServer`: Deprecated in v7.0. Watson now uses TCP natively. Retained for backward compatibility but has no effect.
- `OperationLimits`: Size limits (e.g., MaxPutObjectSize)
- `PreRequestHandler`: Called before routing (return true to terminate)
- `DefaultRequestHandler`: Called when no callback matches
- `PostRequestHandler`: Called after response sent

### Callback Patterns

Callbacks follow three patterns:

1. **Return typed result**: `Task<ListAllMyBucketsResult> ListBuckets(S3Context ctx)`
2. **Return void**: `Task BucketDelete(S3Context ctx)` - just return after completion
3. **Throw S3Exception**: `throw new S3Exception(new Error(ErrorCode.NoSuchBucket))`

The library automatically serializes return values to XML and sets appropriate status codes.

### Error Handling

Use `S3Exception` with `Error` objects for S3-compliant errors:
- `throw new S3Exception(new Error(ErrorCode.NoSuchBucket))`
- `throw new S3Exception(new Error(ErrorCode.AccessDenied))`
- See ErrorCode enum for all available error codes

### Multipart Upload Support

The library supports multipart upload operations through callbacks:
- `CreateMultipartUpload`: Initiate upload
- `UploadPart`: Upload individual parts
- `CompleteMultipartUpload`: Finalize upload
- `AbortMultipartUpload`: Cancel upload
- `ReadParts`: List parts of an upload


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jchristn/S3Server](https://github.com/jchristn/S3Server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
