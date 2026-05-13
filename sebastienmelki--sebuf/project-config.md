---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is `sebuf`, a specialized Go protobuf toolkit for building HTTP APIs. It consists of five complementary protoc plugins that together enable modern, type-safe API development:

- **`protoc-gen-go-http`**: Generates HTTP handlers, routing, request/response binding, and automatic validation
- **`protoc-gen-go-client`**: Generates type-safe Go HTTP clients with functional options pattern
- **`protoc-gen-ts-client`**: Generates TypeScript HTTP clients with full type safety, header helpers, and error handling
- **`protoc-gen-ts-server`**: Generates TypeScript HTTP server handlers using the Web Fetch API (Request/Response), framework-agnostic
- **`protoc-gen-openapiv3`**: Creates comprehensive OpenAPI v3.1 specifications

The toolkit enables developers to build HTTP APIs directly from protobuf definitions without gRPC dependencies, targeting web and mobile API development with built-in request validation.

## Architecture

The project follows a clean Go protoc plugin architecture with separated concerns across two main components:

### Plugin Structure
- **cmd/protoc-gen-go-http/**: HTTP handler generator entry point
- **cmd/protoc-gen-go-client/**: Go HTTP client generator entry point
- **cmd/protoc-gen-ts-client/**: TypeScript HTTP client generator entry point
- **cmd/protoc-gen-ts-server/**: TypeScript HTTP server generator entry point
- **cmd/protoc-gen-openapiv3/**: OpenAPI specification generator entry point
- **internal/httpgen/**: HTTP handler generation logic, annotations, and header validation middleware
- **internal/clientgen/**: Go HTTP client generation logic and annotations
- **internal/tscommon/**: Shared TypeScript type mapping and generation (used by ts-client and ts-server)
- **internal/tsclientgen/**: TypeScript HTTP client generation logic
- **internal/tsservergen/**: TypeScript HTTP server generation logic, header validation, route creation
- **internal/openapiv3/**: OpenAPI generation logic, type mapping, and header parameter generation
- **proto/sebuf/http/**: HTTP annotation definitions including headers.proto for header validation
- **scripts/**: Test automation and build scripts

### Core Components

1. **HTTP Handler Generator** (`internal/httpgen/generator.go:22`): Generates HTTP handlers, request binding, routing configuration, automatic body validation, and header validation middleware
2. **Go HTTP Client Generator** (`internal/clientgen/generator.go:13`): Generates type-safe Go HTTP clients with functional options pattern, automatic request/response marshaling, and error handling
3. **TypeScript HTTP Client Generator** (`internal/tsclientgen/generator.go`): Generates TypeScript HTTP clients with typed interfaces, service/method header helpers, query parameter encoding, path parameter substitution, and structured error handling (ValidationError/ApiError)
4. **TypeScript HTTP Server Generator** (`internal/tsservergen/generator.go`): Generates framework-agnostic TypeScript HTTP server handlers using the Web Fetch API (`Request` → `Promise<Response>`), with route descriptors, header validation, query/body parsing, and error handling
5. **OpenAPI Generator** (`internal/openapiv3/generator.go:53`): Creates comprehensive OpenAPI v3.1 specifications from protobuf definitions with full header parameter support, generating one file per service for better organization
6. **Shared TypeScript Types** (`internal/tscommon/`): Shared TypeScript type mapping, interface generation, error types, and proto-defined error message collection (messages ending with "Error") used by both ts-client and ts-server generators
7. **HTTP Annotations** (`proto/sebuf/http/annotations.proto`): Custom protobuf extensions for HTTP configuration
5. **Header Validation** (`proto/sebuf/http/headers.proto`): Protobuf definitions for service and method-level header validation
6. **Validation System**: Automatic request body validation via buf.validate/protovalidate and header validation middleware

### Generated Output Examples

**HTTP Handlers** - Complete HTTP server infrastructure:
```go
// UserServiceServer is the server API for UserService
type UserServiceServer interface {
    CreateUser(context.Context, *CreateUserRequest) (*User, error)
}

// RegisterUserServiceServer registers HTTP handlers for UserService
func RegisterUserServiceServer(server UserServiceServer, opts ...ServerOption) error
```

**HTTP Clients** - Type-safe HTTP client with functional options:
```go
// UserServiceClient is the client API for UserService
type UserServiceClient interface {
    CreateUser(ctx context.Context, req *CreateUserRequest, opts ...UserServiceCallOption) (*User, error)
}

// Create a client with options
client := NewUserServiceClient(
    "http://localhost:8080",
    WithUserServiceHTTPClient(&http.Client{Timeout: 30 * time.Second}),
    WithUserServiceAPIKey("your-api-key"),  // From service_headers annotation
)

// Make requests with per-call options
user, err := client.CreateUser(ctx, req,
    WithUserServiceHeader("X-Request-ID", "req-123"),
    WithUserServiceCallContentType(ContentTypeProto),
)
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SebastienMelki/sebuf](https://github.com/SebastienMelki/sebuf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
