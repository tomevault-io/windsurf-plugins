---
trigger: always_on
description: This document outlines the API design patterns and standards for the OpenFrame project.
---

# API Design

This document outlines the API design patterns and standards for the OpenFrame project.

## RESTful API Design

### URL Structure

- Use resource-based URLs
- Use plural nouns for resource collections
- Use hierarchical structure for nested resources
- Use kebab-case for multi-word resource names
- Include API version in the URL path

Examples:
```
GET /api/v1/devices                  # Get all devices
GET /api/v1/devices/{id}             # Get a specific device
GET /api/v1/devices/{id}/scripts     # Get scripts for a device
POST /api/v1/devices                 # Create a new device
PUT /api/v1/devices/{id}             # Update a device
DELETE /api/v1/devices/{id}          # Delete a device
```

### HTTP Methods

- Use appropriate HTTP methods for operations:
  - GET: Retrieve resources
  - POST: Create resources
  - PUT: Update resources (full update)
  - PATCH: Partial update of resources
  - DELETE: Remove resources

### Status Codes

- Use appropriate HTTP status codes:
  - 200 OK: Successful request
  - 201 Created: Resource created successfully
  - 204 No Content: Successful request with no response body
  - 400 Bad Request: Invalid request parameters
  - 401 Unauthorized: Authentication required
  - 403 Forbidden: Authenticated but not authorized
  - 404 Not Found: Resource not found
  - 409 Conflict: Request conflicts with current state
  - 422 Unprocessable Entity: Validation errors
  - 500 Internal Server Error: Server-side error

### Request/Response Format

- Use JSON for request and response bodies
- Use consistent property naming (camelCase)
- Include appropriate content-type headers
- Provide meaningful error messages
- Use pagination for large collections
- Support filtering, sorting, and field selection

Example response:
```json
{
  "data": [
    {
      "id": "123",
      "hostname": "device-1",
      "operatingSystem": "Windows",
      "status": "online",
      "lastSeen": "2023-04-01T12:00:00Z"
    }
  ],
  "pagination": {
    "total": 100,
    "page": 1,
    "pageSize": 10,
    "totalPages": 10
  }
}
```

Example error response:
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid request parameters",
    "details": [
      {
        "field": "hostname",
        "message": "Hostname is required"
      }
    ]
  }
}
```

## GraphQL API Design

### Schema Design

- Use descriptive type names
- Follow naming conventions (PascalCase for types, camelCase for fields)
- Define clear relationships between types
- Use input types for mutations
- Include appropriate descriptions for types and fields
- Use enums for fixed sets of values

Example schema:
```graphql
"""
Represents a device in the system
"""
type Device {
  "Unique identifier for the device"
  id: ID!
  "Hostname of the device"
  hostname: String!
  "Operating system of the device"
  operatingSystem: String!
  "Current status of the device"
  status: DeviceStatus!
  "When the device was last seen"
  lastSeen: DateTime
  "Scripts associated with this device"
  scripts: [Script!]!
}

"""
Status of a device
"""
enum DeviceStatus {
  ONLINE
  OFFLINE
  MAINTENANCE
}

"""
Input for creating a new device
"""
input CreateDeviceInput {
  hostname: String!
  operatingSystem: String!
}
```

### Query Design

- Design queries around specific use cases
- Support pagination for collections
- Allow filtering and sorting
- Enable field selection
- Use arguments for customization
- Consider query complexity and depth

Example query:
```graphql
query GetDevices($status: DeviceStatus, $page: Int, $pageSize: Int) {
  devices(status: $status, page: $page, pageSize: $pageSize) {
    nodes {
      id
      hostname
      operatingSystem
      status
      lastSeen
    }
    pageInfo {
      totalCount
      hasNextPage
      hasPreviousPage
    }
  }
}
```

### Mutation Design

- Use descriptive names (createDevice, updateDevice, etc.)
- Accept input types for arguments
- Return the modified resource
- Include error information in the response
- Use consistent naming patterns

Example mutation:
```graphql
mutation CreateDevice($input: CreateDeviceInput!) {
  createDevice(input: $input) {
    device {
      id
      hostname
      operatingSystem
      status
    }
    errors {
      field
      message
    }
  }
}
```

## API Gateway Patterns

### Routing

- Route requests based on path prefixes
- Use consistent routing patterns
- Support path rewriting for backend services
- Handle versioning at the gateway level

Example gateway configuration:
```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: device-service
          uri: lb://device-service
          predicates:
            - Path=/api/v1/devices/**
          filters:
            - RewritePath=/api/v1/devices/(?<path>.*), /devices/$\{path}
```

### Authentication and Authorization

- Implement JWT authentication at the gateway
- Validate tokens for each request
- Include user information in request headers
- Support role-based access control
- Use consistent authorization patterns

Example authentication filter:
```java
@Component
public class JwtAuthenticationFilter implements WebFilter {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, WebFilterChain chain) {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [flamingo-stack/openframe-oss-tenant](https://github.com/flamingo-stack/openframe-oss-tenant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
