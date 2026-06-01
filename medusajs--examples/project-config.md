---
trigger: always_on
description: You are an expert in generating OpenAPI 3.0 specifications for Medusa.js projects. When asked to create an OpenAPI spec for a Medusa project, follow these comprehensive rules and patterns.
---

# OpenAPI Generation Rules for Medusa Projects

You are an expert in generating OpenAPI 3.0 specifications for Medusa.js projects. When asked to create an OpenAPI spec for a Medusa project, follow these comprehensive rules and patterns.

## Core OpenAPI Structure

### Basic Information

- Always use OpenAPI 3.0.3
- Set title to a descriptive name based on the project's main functionality
- Set version to "1.0.0" unless specified otherwise
- Include a clear description of the API's purpose

### Server Configuration

- Use `{BACKEND_URL}` as the server URL variable
- Set default to `http://localhost:9000` (Medusa's default)
- Include description: "Base URL of the backend server"

### Security Schemes

- Always include `bearerAuth` security scheme:

  ```yaml
  bearerAuth:
    type: http
    scheme: bearer
    bearerFormat: JWT
  ```

- Apply `bearerAuth: []` globally unless specific endpoints don't require auth

## Tag Organization

### Standard Tags

- **Admin**: For admin-only endpoints (`/admin/*`)
- **Store**: For store/public endpoints (`/store/*`)
- **Admin > Auth**: For admin authentication endpoints (`/auth/user/emailpass`) - **REQUIRED when `/admin/*` routes exist**
- **Store > Auth**: For store authentication endpoints (`/auth/customer/emailpass`, `/store/customers`) - **REQUIRED when `/store/customers/me/*` routes exist OR when customer-specific store functionality requires authentication**
- **Admin > [Feature]**: For specific admin feature groups
- **Store > [Feature]**: For specific store feature groups

### Tag Naming Convention

- Use title case for main tags
- Use "Admin > Feature" format for nested admin features
- Use "Store > Feature" format for nested store features
- Group related endpoints under the same tag

## Schema Patterns

### Authentication Schemas

```yaml
LoginRequest:
  type: object
  required: [email, password]
  properties:
    email:
      type: string
      format: email
    password:
      type: string
      format: password
  example:
    email: admin@medusajs.com
    password: supersecret

LoginResponse:
  type: object
  properties:
    token:
      type: string
      description: JWT access token
  example:
    token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

### Request/Response Schemas

- Use descriptive names ending with "Request" or "Response"
- Include required fields explicitly
- Add examples for all schemas
- Use proper data types and formats
- Include descriptions for complex fields

### Common Response Patterns

```yaml
GenericSuccessResponse:
  type: object
  properties:
    status:
      type: string
      example: success
    message:
      type: string

GenericErrorResponse:
  type: object
  properties:
    message:
      type: string
    code:
      type: string
    type:
      type: string
      enum: [error, warning]
```

## Path Patterns

### Medusa Route Structure

- **Admin routes**: `/admin/{resource}` or `/admin/{resource}/{id}`
- **Store routes**: `/store/{resource}` or `/store/{resource}/{id}`
- **Auth routes**: `/auth/{method}` (e.g., `/auth/user/emailpass`)
- **Nested resources**: `/admin/{parent}/{id}/{child}`

### Path Parameter Conventions

- Use `{id}` for single resource identifiers
- Use `{category_id}`, `{product_id}`, `{order_id}` for specific resource types
- Use `{id}` for generic identifiers when context is clear

### HTTP Method Mapping

- **GET**: Retrieve resources (list or single)
- **POST**: Create new resources or trigger actions
- **PUT**: Update entire resources
- **PATCH**: Partial updates
- **DELETE**: Remove resources

## Request/Response Patterns

### Request Body Patterns

- Always include `required: true` for POST/PUT/PATCH
- Use `application/json` content type
- Reference schema definitions
- Include examples

### Response Patterns

- **200**: Successful GET requests
- **201**: Successful POST requests (creation)
- **400**: Bad request (validation errors)
- **401**: Unauthorized (missing/invalid auth)
- **403**: Forbidden (insufficient permissions)
- **404**: Not found
- **500**: Internal server error

### Content Types

- Use `application/json` for most responses
- Use `application/pdf` for file downloads
- Use `text/plain` for simple text responses

## Medusa-Specific Patterns

### Authentication Endpoints

#### Admin Authentication (Required when `/admin/*` routes exist)

```yaml
/auth/user/emailpass:
  post:
    tags: [Admin > Auth]
    summary: Login with email and password
    security: []  # No auth required for login
    requestBody:
      required: true
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/LoginRequest'
    responses:
      '200':
        description: Successful authentication
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/LoginResponse'
      '401':
        description: Invalid credentials
```

#### Store Authentication (Required when `/store/customers/me/*` routes exist)

```yaml
/auth/customer/emailpass:
  post:
    tags: [Store > Auth]
    summary: Get JWT / Login
    security: []  # No auth required for login
    requestBody:
      required: true

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [medusajs/examples](https://github.com/medusajs/examples) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
