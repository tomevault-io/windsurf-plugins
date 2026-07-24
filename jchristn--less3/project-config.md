---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Less3 is an S3-compatible object storage platform written in C# (.NET 8.0) that can be deployed anywhere. It implements AWS S3 APIs using the S3Server library and provides both path-style and virtual-hosted URL support for bucket access.

## Build and Run Commands

### Building the Project
```bash
# Build the solution
dotnet build src/Less3.sln

# Build specific configuration
dotnet build src/Less3.sln -c Release
dotnet build src/Less3.sln -c Debug

# Publish for deployment
dotnet publish src/Less3/Less3.csproj -c Release -o ./publish
```

### Running the Application
```bash
# Run from the project directory
cd src/Less3
dotnet run

# Run with setup wizard (creates system.json and less3.db)
dotnet run setup

# Run from published output
cd publish
dotnet Less3.dll
```

### Docker Deployment
```bash
# Using Docker Compose
cd Docker
docker compose up -d
docker compose down

# Or use the provided scripts
./Docker/compose-up.sh    # Linux/Mac
./Docker/compose-up.bat   # Windows
```

### Testing with AWS CLI
See `AWSCLI.md` for comprehensive AWS CLI testing commands. Key endpoints:
- Default access key: `default`
- Default secret key: `default`
- Default endpoint: `http://localhost:8000`

## Architecture Overview

### Core Architecture Layers

**Program.cs** (src/Less3/Program.cs)
- Entry point and initialization
- Creates all managers in specific order: Logging → Database → Config → Bucket → Auth → API Handler → Admin API Handler → Console → S3Server
- Hosts the S3Server and routes requests through PreRequestHandler → S3Server APIs → PostRequestHandler
- Handles authentication/authorization in PreRequestHandler before delegating to S3Server

**Manager Layer**
- `ConfigManager`: Manages users, credentials, buckets via WatsonORM
- `BucketManager`: Manages bucket lifecycle and maintains BucketClient instances for each bucket
- `AuthManager`: Handles authentication and authorization, produces RequestMetadata with auth results
- `ConsoleManager`: Interactive console for administration when enabled

**API Layer** (src/Less3/Api/)
- `ApiHandler`: Primary S3 API facade that delegates to specialized handlers
- `ServiceHandler`: Service-level APIs (ListBuckets)
- `BucketHandler`: Bucket operations (Create, Delete, Read, Write, ACLs, Tags, Versioning)
- `ObjectHandler`: Object operations (Read, Write, Delete, Range reads, ACLs, Tags)
- `AdminApiHandler`: Administrative APIs accessed via x-api-key header

**Storage Layer** (src/Less3/Storage/)
- `StorageDriverBase`: Abstract base class for storage backends
- `DiskStorageDriver`: File system-based storage implementation
- Objects stored in `{bucket.DiskDirectory}/{obj.BlobFilename}`

**Data Models** (src/Less3/Classes/)
- `Bucket`, `Obj`, `User`, `Credential`: Core entities stored in WatsonORM
- `BucketAcl`, `ObjectAcl`: Access control lists
- `BucketTag`, `ObjectTag`: Tagging support
- `RequestMetadata`: Contains authentication/authorization results, bucket/object references for each request

### Request Flow

1. HTTP request arrives → `PreRequestHandler` in Program.cs
2. Authentication: Extract access key from request, look up User and Credential
3. Authorization: Check bucket/object ownership, ACLs (AllUsers, AuthenticatedUsers, per-user), or bucket global config (EnablePublicRead/Write)
4. Store `RequestMetadata` in `ctx.Metadata`
5. Delegate to appropriate handler (Service/Bucket/Object)
6. Handler retrieves metadata via `ApiHelper.GetRequestMetadata(ctx)`
7. Handler performs operation, interacts with BucketClient
8. Response sent, `PostRequestHandler` logs metrics

### Authentication & Authorization Architecture

**Authentication Flow** (AuthManager.AuthenticateAndBuildMetadata)
- Extracts access key from Authorization header
- Looks up Credential and User
- Populates RequestMetadata with authentication result (Authenticated, NotAuthenticated, NoMaterialSupplied, AccessKeyNotFound, UserNotFound)
- Loads Bucket, Object, ACLs into RequestMetadata

**Authorization Flow** (AuthManager.Authorize* methods)
Authorization is checked in this order:
1. Admin API key (full access)
2. Bucket/Object global config (EnablePublicRead/Write)
3. AllUsers ACLs (anonymous access)
4. Authenticated user checks:
   - Bucket/Object ownership
   - AuthenticatedUsers ACLs
   - Per-user ACLs

**Authorization Results**
- `AdminAuthorized`: Admin API key used
- `PermitBucketOwnership` / `PermitObjectOwnership`: User owns the resource
- `PermitBucketGlobalConfig`: Bucket allows public access
- `PermitBucketAllUsersAcl` / `PermitObjectAllUsersAcl`: AllUsers ACL grants access
- `PermitBucketAuthUserAcl` / `PermitObjectAuthUserAcl`: AuthenticatedUsers ACL grants access
- `PermitBucketUserAcl` / `PermitObjectUserAcl`: Per-user ACL grants access
- `NotAuthorized`: Access denied

### Database Schema (WatsonORM)

Tables initialized in Program.cs InitializeGlobals:
- `User`: Users in the system (GUID, Name, Email, etc.)
- `Credential`: Access keys and secret keys linked to users
- `Bucket`: Bucket metadata (Name, OwnerGUID, DiskDirectory, EnablePublicRead/Write, EnableVersioning)
- `BucketAcl`: Bucket-level access control

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jchristn/Less3](https://github.com/jchristn/Less3) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
