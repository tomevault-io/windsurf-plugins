---
trigger: always_on
description: User wants to add support for a completely new Nylas API resource (e.g., /v3/widgets).
---

# Nylas Node.js SDK Development Rules

## Rule 1: Adding a New API Resource

### Description
User wants to add support for a completely new Nylas API resource (e.g., /v3/widgets).

### Conditions
- Matches files: Any file
- Code selection matches: Case-insensitive phrases like "add new resource", "new endpoint group", etc.

### Instructions

To add support for a new Nylas API resource (let's call it `{{RESOURCE_NAME}}`), follow these steps:

#### 1. Define Models
- Create a new file `src/models/{{RESOURCE_NAME_LC}}.ts`
- Define TypeScript interfaces/types for:
  - The main resource object (e.g., `{{RESOURCE_NAME}}`) with all properties documented
  - Request bodies (e.g., `Create{{RESOURCE_NAME}}Request`, `Update{{RESOURCE_NAME}}Request`)
  - Query parameters (e.g., `List{{RESOURCE_NAME}}QueryParams`, `Find{{RESOURCE_NAME}}QueryParams`)
  - Use standard response types from `src/models/response.ts` (`NylasResponse<T>`, `NylasListResponse<T>`)
  - Use consistent property naming (camelCase) and proper TSDoc annotations
- Export all types from the new model file
- Add the export statement to `src/models/index.ts`

Example model structure:
```typescript
import { ListQueryParams } from './listQueryParams.js';

/**
 * Interface representing a Nylas {{RESOURCE_NAME}} object.
 */
export interface {{RESOURCE_NAME}} {
  id: string;
  grantId: string;
  object: '{{RESOURCE_NAME_LC}}';
  // Add all resource properties with appropriate types and optionality
}

/**
 * Interface representing the query parameters for listing {{RESOURCE_NAME_LC}}s.
 */
export interface List{{RESOURCE_NAME}}QueryParams extends ListQueryParams {
  // Add specific query parameters
}

/**
 * Interface representing the query parameters for retrieving a single {{RESOURCE_NAME_LC}}.
 */
export interface Find{{RESOURCE_NAME}}QueryParams {
  // Add specific query parameters
}

/**
 * Interface for creating a {{RESOURCE_NAME_LC}}.
 */
export type Create{{RESOURCE_NAME}}Request = {
  // Add all properties needed for creation
};

/**
 * Interface for updating a {{RESOURCE_NAME_LC}}.
 */
export type Update{{RESOURCE_NAME}}Request = {
  // Add all properties needed for updating
};
```

#### 2. Define Resource Class
- Create a new file `src/resources/{{RESOURCE_NAME_LC}}.ts`
- Create a class `{{RESOURCE_NAME}}` that extends `Resource` from `src/resources/resource.ts`
- Import all models defined in the previous step
- Define interface parameters for each endpoint method
- Implement public methods for each endpoint (e.g., `list`, `find`, `create`, `update`, `destroy`)
- Use typed parameters and return values for all methods
- Call appropriate protected methods from the base `Resource` class

Example resource implementation:
```typescript
import { Overrides } from '../config.js';
import {
  Create{{RESOURCE_NAME}}Request,
  {{RESOURCE_NAME}},
  List{{RESOURCE_NAME}}QueryParams,
  Find{{RESOURCE_NAME}}QueryParams,
  Update{{RESOURCE_NAME}}Request,
} from '../models/{{RESOURCE_NAME_LC}}.js';
import {
  NylasResponse,
  NylasListResponse,
  NylasBaseResponse,
} from '../models/response.js';
import { AsyncListResponse, Resource } from './resource.js';

/**
 * @property {{RESOURCE_NAME_LC}}Id The id of the {{RESOURCE_NAME}} to retrieve.
 * @property identifier The identifier of the grant to act upon
 * @property queryParams The query parameters to include in the request
 */
interface Find{{RESOURCE_NAME}}Params {
  identifier: string;
  {{RESOURCE_NAME_LC}}Id: string;
  queryParams: Find{{RESOURCE_NAME}}QueryParams;
}

/**
 * @property identifier The identifier of the grant to act upon
 * @property queryParams The query parameters to include in the request
 */
interface List{{RESOURCE_NAME}}Params {
  identifier: string;
  queryParams: List{{RESOURCE_NAME}}QueryParams;
}

/**
 * @property identifier The identifier of the grant to act upon
 * @property requestBody The values to create the {{RESOURCE_NAME}} with
 */
interface Create{{RESOURCE_NAME}}Params {
  identifier: string;
  requestBody: Create{{RESOURCE_NAME}}Request;
}

/**
 * @property identifier The identifier of the grant to act upon
 * @property {{RESOURCE_NAME_LC}}Id The id of the {{RESOURCE_NAME}} to retrieve.
 * @property requestBody The values to update the {{RESOURCE_NAME}} with
 */
interface Update{{RESOURCE_NAME}}Params {
  identifier: string;
  {{RESOURCE_NAME_LC}}Id: string;
  requestBody: Update{{RESOURCE_NAME}}Request;
}

/**
 * @property identifier The identifier of the grant to act upon
 * @property {{RESOURCE_NAME_LC}}Id The id of the {{RESOURCE_NAME}} to retrieve.
 */
interface Destroy{{RESOURCE_NAME}}Params {
  identifier: string;
  {{RESOURCE_NAME_LC}}Id: string;
}

/**
 * Nylas {{RESOURCE_NAME}}s API
 *
 * [Add a brief description of the resource]
 */
export class {{RESOURCE_NAME}}s extends Resource {
  /**
   * Return all {{RESOURCE_NAME}}s
   * @return The list of {{RESOURCE_NAME}}s
   */
  public list({
    identifier,
    queryParams,
    overrides,
  }: List{{RESOURCE_NAME}}Params & Overrides): AsyncListResponse<
    NylasListResponse<{{RESOURCE_NAME}}>
  > {
    return super._list({
      queryParams,
      path: `/v3/grants/${identifier}/{{RESOURCE_NAME_LC}}s`,
      overrides,
    });
  }

  /**
   * Return a {{RESOURCE_NAME}}

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nylas/nylas-nodejs](https://github.com/nylas/nylas-nodejs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
