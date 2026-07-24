---
trigger: always_on
description: Azure Storage Explorer is a web-based application for managing Azure Storage resources including blobs, tables, queues, and file shares. The application provides an intuitive interface for developers to interact with Azure Storage without installing local clients.
---

# Azure Storage Explorer - Agent Guide

## Project Overview

Azure Storage Explorer is a web-based application for managing Azure Storage resources including blobs, tables, queues, and file shares. The application provides an intuitive interface for developers to interact with Azure Storage without installing local clients.

**Live Demo:** https://azurestorage.azurewebsites.net  
**Docker Hub:** https://hub.docker.com/r/sebagomez/azurestorageexplorer/

### Technology Stack

- **Framework:** .NET 10.0 with Blazor Server
- **Previous Versions:** Originally built with ASP.NET WebForms, migrated through .NET Core 2.1, 2.2, 3.1, 5.0, 6, 7, 8, and Angular (later moved to Blazor to avoid npm dependency issues)
- **Build Tool:** [just](https://github.com/casey/just) with justfile for build automation
- **Container:** Docker images available and automatically built via GitHub Actions
- **Deployment:** Supports Docker, Docker Compose, Kubernetes, and Helm

### Key Features

1. **Blob Storage Management**
   - Create public or private containers
   - Upload BlockBlobs (other blob types not yet supported)
   - Download and delete blobs

2. **Queue Management**
   - Create queues
   - Create and manage messages

3. **File Share Support**
   - Navigate file shares
   - Browse directory structures

4. **Table Storage**
   - Create tables and entities
   - Query entities with OData-style operators
   - Support for typed properties with EDM data types

5. **Multi-Cloud Support (Beta)**
   - Azure Storage (primary)
   - AWS S3 buckets
   - Google Cloud Platform (GCP) buckets
   - Local Azurite emulator support

## Project Structure

```
azurestorageexplorer/
├── justfile                 # Build automation commands
├── docker-compose.yml       # Docker Compose manifest for Azurite + Explorer
├── k8s/                     # Kubernetes manifests
│   ├── deployment.yaml
│   └── service.yaml
└── src/                     # .NET 10.0 Blazor Server application
```

## Authentication & Configuration

### Connection Methods

The application supports multiple authentication methods:

1. **Account Name + Key**
2. **Shared Access Signature (SAS)**
3. **Connection String** (recommended)

### Environment Variables

**Azure Storage:**
- `AZURE_STORAGE_CONNECTIONSTRING` - Full connection string (takes precedence)
- `AZURE_STORAGE_ACCOUNT` - Account name
- `AZURE_STORAGE_KEY` - Access key
- `AZURE_STORAGE_ENDPOINT` - Custom endpoint
- `AZURITE` - Set to `true` for Azurite emulator

**AWS S3:**
- `CLOUD_PROVIDER=AWS`
- `AWS_ACCESS_KEY`
- `AWS_SECRET_KEY`
- `AWS_REGION`

**Google Cloud Platform:**
- `CLOUD_PROVIDER=GCP`
- `GCP_CREDENTIALS_FILE` - Full path to service account credentials file

**Note:** If `AZURE_STORAGE_CONNECTIONSTRING` is set, other Azure variables are ignored. Otherwise, all three (account, key, endpoint) must be present.

## Building and Running

### Prerequisites
- .NET 10.0 SDK: https://dotnet.microsoft.com/en-us/download
- just (optional): https://github.com/casey/just

### Local Development

```bash
# Build the project
just build

# Publish to bin folder
just publish

# Application will start with Kestrel, typically on http://localhost:5000
```

### Docker

```bash
# Run latest version
docker run --rm -it -p 8000:8080 sebagomez/azurestorageexplorer

# Access at http://localhost:8000
```

### Docker Compose (with Azurite)

```bash
# Start Azurite + Storage Explorer
just compose

# Access at http://localhost:8080 (auto-logged into Azurite)
```

### Kubernetes

```bash
# Apply manifests
kubectl apply -f ./k8s

# Port forward
kubectl port-forward svc/azurestorageexplorer 8080:8080

# Access at http://localhost:8080
```

### Helm Chart (v2.7.1+)

```bash
# Add repository
helm repo add sebagomez https://sebagomez.github.io/azurestorageexplorer

# Install chart
helm install azurestorageexplorer sebagomez/azurestorageexplorer

# Port forward
kubectl port-forward service/azurestorageexplorer 8080:8080
```

## Working with Table Storage

### Creating Entities

Entities are created using property-value pairs, one per line in the format:
```
<PropertyName>='<PropertyValue>'
```

**Example: Creating a movie entity**
```
PartitionKey=Action
RowKey=1
Title=Die Hard
```

**Default Values:**
- `PartitionKey`: "1" (if not specified)
- `RowKey`: Current timestamp (if not specified)

### Typed Properties

Set data types using EDM notation:
```
Year=1978
[Year@odata.type]=Edm.Int32
```

**Supported EDM Types:**
- `Edm.Int64`
- `Edm.Int32`
- `Edm.Boolean`
- `Edm.DateTime`
- `Edm.Double`
- `Edm.Guid`
- Default: String (for any other type)

### Querying Entities

Query syntax: `<PropertyName> [operator] <PropertyValue>`

**Supported Operators:**
- `eq` - equals
- `gt` - greater than
- `ge` - greater or equal
- `lt` - less than
- `le` - less or equal
- `ne` - not equal

**Important:** Operators must have spaces before and after them.

**Example:**
```
PartitionKey eq 'Action'
```

**Note:** Empty query retrieves all entities from the table.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sebagomez/azurestorageexplorer](https://github.com/sebagomez/azurestorageexplorer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
