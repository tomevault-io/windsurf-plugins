---
trigger: always_on
description: Specification authoring — how to write, structure, and manage specs as the single source of truth
---


# Specification Authoring

## Principles

- Specs are **the single source of truth** for the system's behavior and contracts.
- Specs must be **machine-readable** whenever possible — they generate tests, docs, and stubs.
- Specs must be **human-readable** — any developer should understand the system from specs alone.
- Specs are **living documents** — they evolve with the project, never become stale.
- Specs are **versioned** — changes to specs are tracked, reviewed, and approved like code.

## Spec Directory Structure

```
specs/
  api/                    # API contracts
    openapi.yaml          # REST API specification (OpenAPI 3.x)
    schema.graphql        # GraphQL schema definition
    service.proto         # gRPC service definition
    asyncapi.yaml         # Event-driven API specification
  schemas/                # Data contracts
    user.schema.json      # JSON Schema for User entity
    order.schema.json     # JSON Schema for Order entity
    shared/               # Shared schema definitions ($ref targets)
      address.schema.json
      pagination.schema.json
      error.schema.json
  contracts/              # Integration contracts
    payment-api.pact.json # Consumer-driven contract (Pact)
    email-service.yaml    # External service contract
  features/               # Behavior specifications
    auth.feature          # Gherkin / Given-When-Then
    checkout.feature
  ui/                     # UI component contracts
    button.props.ts       # Component prop types and variants
    form-field.props.ts
  slos/                   # Performance/reliability specs
    api-performance.yaml  # SLOs, SLIs, error budgets
  decisions/              # Architecture Decision Records
    001-database-choice.md
    002-auth-strategy.md
```

## Data Contracts

Define every entity as a formal schema:

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "User",
  "type": "object",
  "required": ["id", "email", "name", "role", "createdAt"],
  "properties": {
    "id": { "type": "string", "format": "uuid" },
    "email": { "type": "string", "format": "email" },
    "name": { "type": "string", "minLength": 1, "maxLength": 100 },
    "role": { "enum": ["admin", "user", "viewer"] },
    "createdAt": { "type": "string", "format": "date-time" }
  },
  "additionalProperties": false
}
```

Rules for data contracts:
- Every entity has a schema. No untyped data flows through the system.
- Use `$ref` for shared definitions (address, pagination, error envelope).
- Define `required` fields explicitly. Default to required, opt-in to optional.
- Use `additionalProperties: false` to catch unexpected fields.
- Include format constraints (`email`, `uuid`, `date-time`, `uri`).
- Define enums for all finite value sets.

## API Contracts

### REST API (OpenAPI)

Define every endpoint in OpenAPI format:

```yaml
paths:
  /api/v1/users:
    get:
      summary: List users
      operationId: listUsers
      parameters:
        - $ref: '#/components/parameters/PageParam'
        - $ref: '#/components/parameters/LimitParam'
      responses:
        '200':
          description: Paginated list of users
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/UserListResponse'
        '401':
          $ref: '#/components/responses/Unauthorized'
    post:
      summary: Create a user
      operationId: createUser
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateUserRequest'
      responses:
        '201':
          description: User created
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
        '400':
          $ref: '#/components/responses/ValidationError'
        '409':
          $ref: '#/components/responses/Conflict'
```

Rules for REST API contracts:
- Every endpoint has a full OpenAPI definition BEFORE implementation.
- Define all response codes (success + every error case).
- Use `$ref` to reference data schemas — don't duplicate.
- Include request/response examples for each endpoint.
- Define shared components: error envelope, pagination, auth headers.
- Version the API in the spec (`/api/v1/`).

### GraphQL Schema

Define the schema using SDL:

```graphql
type Query {
  """List users with pagination"""
  users(page: Int = 1, limit: Int = 20): UserConnection!

  """Get a single user by ID"""
  user(id: ID!): User
}

type Mutation {
  """Create a new user"""
  createUser(input: CreateUserInput!): CreateUserPayload!

  """Update an existing user"""
  updateUser(id: ID!, input: UpdateUserInput!): UpdateUserPayload!

  """Delete a user"""
  deleteUser(id: ID!): DeleteUserPayload!
}

type User {
  id: ID!
  email: String!
  name: String!
  role: UserRole!
  createdAt: DateTime!
  updatedAt: DateTime!
}

enum UserRole {
  ADMIN
  USER
  VIEWER
}

input CreateUserInput {
  email: String!
  name: String!
  password: String!
  role: UserRole = USER
}

type CreateUserPayload {
  user: User
  errors: [UserError!]
}

type UserError {
  field: String!
  message: String!
  code: String!
}

type UserConnection {
  nodes: [User!]!

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GaetanOff/WAF-GaetanDev](https://github.com/GaetanOff/WAF-GaetanDev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
