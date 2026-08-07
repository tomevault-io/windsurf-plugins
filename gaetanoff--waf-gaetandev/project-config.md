---
trigger: always_on
description: Ready-to-use specification templates — OpenAPI, JSON Schema, Gherkin, ADR, Pact, AsyncAPI
---


# Spec Templates

## Overview

These templates ensure every specification follows a consistent, complete format. Use them as starting points — adapt to the project's needs but preserve the required sections.

## OpenAPI 3.1 Skeleton

Start every API spec from this skeleton:

```yaml
openapi: 3.1.0
info:
  title: <Project Name> API
  description: <One-line purpose of the API>
  version: 1.0.0
  contact:
    name: <Team Name>
    email: <team@example.com>

servers:
  - url: http://localhost:3000/api/v1
    description: Local development
  - url: https://staging.example.com/api/v1
    description: Staging
  - url: https://api.example.com/api/v1
    description: Production

security:
  - bearerAuth: []

paths:
  /health:
    get:
      summary: Health check
      operationId: healthCheck
      tags: [System]
      security: []
      responses:
        '200':
          description: Service is healthy
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/HealthResponse'

  # Add resource endpoints here following the pattern:
  # /<resource>:
  #   get:   listResources   → 200 (paginated list)
  #   post:  createResource  → 201 (created entity)
  # /<resource>/{id}:
  #   get:    getResource    → 200 (single entity)
  #   put:    updateResource → 200 (updated entity)
  #   delete: deleteResource → 204 (no content)

components:
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT

  parameters:
    PageParam:
      name: page
      in: query
      required: false
      schema:
        type: integer
        minimum: 1
        default: 1
    LimitParam:
      name: limit
      in: query
      required: false
      schema:
        type: integer
        minimum: 1
        maximum: 100
        default: 20
    IdParam:
      name: id
      in: path
      required: true
      schema:
        type: string
        format: uuid

  schemas:
    HealthResponse:
      type: object
      required: [status, timestamp]
      properties:
        status:
          type: string
          enum: [healthy, degraded, unhealthy]
        timestamp:
          type: string
          format: date-time
        version:
          type: string
        services:
          type: object
          additionalProperties:
            type: string
            enum: [up, down]

    PaginatedResponse:
      type: object
      required: [data, meta]
      properties:
        data:
          type: array
          items: {}
        meta:
          $ref: '#/components/schemas/PaginationMeta'

    PaginationMeta:
      type: object
      required: [page, limit, total, totalPages]
      properties:
        page:
          type: integer
        limit:
          type: integer
        total:
          type: integer
        totalPages:
          type: integer
        hasNextPage:
          type: boolean
        hasPreviousPage:
          type: boolean

    ErrorResponse:
      type: object
      required: [error]
      properties:
        error:
          type: object
          required: [code, message, timestamp]
          properties:
            code:
              type: string
              description: Machine-readable error code (UPPER_SNAKE_CASE)
            message:
              type: string
              description: Human-readable error message
            timestamp:
              type: string
              format: date-time
            requestId:
              type: string
              format: uuid
              description: Correlation ID for tracing
            details:
              type: array
              items:
                type: object
                required: [field, message]
                properties:
                  field:
                    type: string
                  message:
                    type: string
                  code:
                    type: string

  responses:
    BadRequest:
      description: Invalid request payload
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ErrorResponse'
          example:
            error:
              code: VALIDATION_ERROR
              message: Request validation failed
              timestamp: '2024-01-01T00:00:00Z'
              details:
                - field: email
                  message: Must be a valid email address
                  code: INVALID_FORMAT
    Unauthorized:
      description: Missing or invalid authentication
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ErrorResponse'
          example:
            error:
              code: UNAUTHORIZED
              message: Authentication required
              timestamp: '2024-01-01T00:00:00Z'
    Forbidden:
      description: Insufficient permissions
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ErrorResponse'
    NotFound:
      description: Resource not found
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ErrorResponse'
    Conflict:
      description: Resource conflict (duplicate)
      content:
        application/json:
          schema:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GaetanOff/WAF-GaetanDev](https://github.com/GaetanOff/WAF-GaetanDev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
