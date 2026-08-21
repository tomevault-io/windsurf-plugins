---
trigger: always_on
description: When working with this repository:
---

# GitHub Copilot Instructions for 2BA OpenAPI Specification

## Copilot Behavior Instructions

When working with this repository:

1. **Always follow the established naming conventions**:
   
   **File Naming (for organization)**:
   - **PascalCase** for schema files: `ErrorResponse.yaml`, `Price.yaml`, `NetPriceResponse.yaml`
   - **kebab-case** for parameter/response files: `page-number.yaml`, `400-bad-request.yaml`
   
   **Component Names (for $ref references)**:
   - **PascalCase** for ALL component references: `Price`, `PageNumber`, `BadRequest`, `NotFound`
   - Component keys must match regex: `^[a-zA-Z0-9\.\-_]+$`
   - References always use component names, not filenames
   
   **Schema Properties & Parameters**:
   - **camelCase** for properties: `netPrice`, `errorCode`, `pageNumber`
   - **camelCase** for parameter names: `supplierGln`, `tradeItemId`, `quantity`
   - **camelCase** for operationIds: `getTradeItemNetPrice`, `getProducts`
   
   **API Paths**:
   - **kebab-case** for path segments: `/netprices`, `/trade-items`
   
  **Response Envelope Pattern (Single-Item and Bulk)**:
  - All responses wrap content in a `data` property — never return domain objects directly
  - The `data` property **must always use a named `$ref`** — NEVER define `data` as an inline anonymous `type: object`
  - Inline anonymous objects cause code generators (NSwag) to produce ambiguous types (`Data`, `Data2`, `Data3`)
  - For single-item responses: `data: { $ref: ./XxxResponseData.yaml }` where the `*ResponseData` schema contains composite key + domain data
  - For bulk responses: `data: { type: array, items: { $ref: ... } }` with `meta: { $ref: CursorPaginationMetadata }`
  - Naming: `{Entity}{Aspect}ResponseData.yaml` (e.g., `ProductDetailsResponseData`, `TradeItemPricingsResponseData`)
  - `*ResponseData` files go in `schemas/responses/` and must be registered in `openapi.yaml` under `components/schemas`
  - Use `meta` for cursor pagination information (reuse `CursorPaginationMetadata` with `cursor`, `prevCursor`, `hasNext`, `hasPrev`, `limit`, `estimatedTotal`)

2. **OpenAPI 3.1 / JSON Schema 2020-12 Requirements**:
   - Optional ETIM scalars use a non-null type and are omitted from `required`; absent means no value
   - Optional ETIM objects use a direct `$ref` and are omitted from `required`
   - Use JSON Schema null types only where the API intentionally gives `null` distinct semantics (NOT deprecated `nullable: true`)
   - Use `type: array` (required, never nullable) for collection properties in sub-resource response schemas — empty collections use `[]`, never `null`
   - **Exception**: Aggregate root response schemas (`ProductResponseData`, `TradeItemResponseData`) use `type: ["array", "null"]` for collection properties to support partial inclusion (`null` = not included in this response, `[]` = included but empty)
   - **Exception**: Aggregate root response schemas also use `anyOf: [$ref, type: "null"]` + optional (not in `required`) for singular object properties (`details`, `lcaEnvironmental`, `ordering`) to support three-state field selection (absent = not requested, `null` = requested but no data, object = has data)
   - Use `anyOf`, `oneOf`, `allOf` for composition (avoid deprecated patterns)
  - Allow `additionalProperties` throughout the object model, including nested models, to preserve backward compatibility when optional fields are added
   - Use `examples` array (plural) in schemas, not `example` (singular, deprecated)
   - Include `format` for type hints: `uri`, `email`, `date-time`, `uuid`, etc.
   - Use `format: decimal` on all ETIM-converted number fields (code-gen hint for NSwag/.NET `decimal`; see [design decisions](../docs/product-data-openapi-design-decisions.md#format-decimal--code-generator-hint))
   - Use `minLength`, `maxLength`, `minimum`, `maximum` for validation
   - Prefer `const` over single-value `enum` for literal values

3. **Maintain the directory structure** - Place schemas in appropriate domain folders
4. **Use $ref extensively** - Don't repeat schema definitions; create reusable components
5. **Create separate files** for reusable enumerations and shared types
6. **Follow the DTO pattern** - Keep response schemas separate from domain schemas
7. **Include comprehensive examples** - Every schema should have realistic examples
8. **Document business context** - Add descriptions explaining purpose and constraints
9. **Support flexible identifiers** - Use `anyOf`/`oneOf` patterns for GLN/DUNS/alternative IDs
10. **Maintain consistent error responses** - Use RFC 9457 Problem Details format (`application/problem+json`). Clients branch on `type` and `status`, never on `detail` or `title` strings. No exception-message matching.

11. **Security scheme convention**:
   - All APIs use **OAuth 2.0 Client Credentials** (`clientCredentials` flow)
   - Token URL: `https://identity.2ba.nl/connect/token` (production)
   - Scope naming: `read:{resource}` (e.g., `read:products`, `read:tradeitems`, `read:netprices`, `read:stock`)
   - Support **RFC 7523 client assertion** (`client_assertion` + `client_assertion_type`) as an alternative to `client_secret`
   - Do NOT use `bearerAuth` (HTTP Bearer) or `apiKeyAuth` — all auth goes through OAuth2

12. **Tag and grouping conventions**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [2BABV/product-data-openapi](https://github.com/2BABV/product-data-openapi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
