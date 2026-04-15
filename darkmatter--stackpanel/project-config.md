---
trigger: always_on
description: Always use bun to install dependencies.
---

Always use bun to install dependencies.

All dependencies must be peer dependencies.

Always use alchemy.secret() instead of new Secret() to create secrets.

# Running Tests with Vitest

We use Vitest for testing. Here's how to run tests:

```bash
# Run all tests
bunx vitest

# Run tests in a specific file
bunx vitest alchemy/test/stripe/price.test.ts

# Run a specific test in a specific file
bunx vitest --test-name-pattern="create and update price" alchemy/test/stripe/price.test.ts
```

For resource tests, create a dedicated test file for each resource type following the pattern `alchemy/test/service-name/resource-name.test.ts`.

# Creating a New Service Resource

This guide provides step-by-step instructions for creating a new resource for a service (like Stripe's Price, Product, or Webhook resources).

## Step 1: Create the Resource File

Create a new file in the service directory with kebab-case naming:

```
alchemy/src/{{service-name}}/{{resource-name}}.ts
```

Example: `alchemy/src/stripe/price.ts`

## Step 2: Define Resource Interfaces

Start by importing dependencies and defining the resource interfaces:

```typescript
import type { Context } from "../context";
import { Resource } from "../resource";

/**
 * Properties for creating or updating a {{ResourceName}}
 */
export interface {{ResourceName}}Props {
  /**
   * {{Property description}}
   */
  propertyName: string;

  /**
   * {{Property description}}
   */
  anotherProperty?: number;

  // Add all required and optional properties
  // Include JSDoc comments for each property
}

/**
 * Output returned after {{ResourceName}} creation/update
 * IMPORTANT: The interface name MUST match the exported resource name
 * For example, if your resource is exported as "Product", this interface
 * should be named "Product" (not "ProductOutput")
 *
 */
export interface {{ResourceName}} extends Resource<"{{service-name}}::{{ResourceName}}"> {{ResourceName}}Props {
  /**
   * The ID of the resource
   */
  id: string;

  /**
   * Time at which the object was created
   */
  createdAt: number;

  // Add all additional properties returned by the service
  // Include JSDoc comments for each property
}
```

## Step 3: API Client Implementation

Create a minimal API client that wraps fetch calls without excessive abstraction:

```typescript
/**
 * Options for {{ServiceName}} API requests
 */
export interface {{ServiceName}}ApiOptions {
  /**
   * API key or token to use (overrides environment variable)
   */
  apiKey?: string;

  /**
   * Account or project ID (overrides environment variable)
   */
  accountId?: string;
}

/**
 * Minimal API client using raw fetch
 */
export class {{ServiceName}}Api {
  /** Base URL for API */
  readonly baseUrl: string;

  /** API key or token */
  readonly apiKey: string;

  /** Account ID */
  readonly accountId: string;

  /**
   * Create a new API client
   *
   * @param options API options
   */
  constructor(options: {{ServiceName}}ApiOptions = {}) {
    // Initialize with environment variables or provided values
    this.baseUrl = "https://api.{{service-name}}.com/v1";
    this.apiKey = options.apiKey || process.env.{{SERVICE_API_KEY}} || '';
    this.accountId = options.accountId || process.env.{{SERVICE_ACCOUNT_ID}} || '';

    // Validate required configuration
    if (!this.apiKey) {
      throw new Error("{{SERVICE_API_KEY}} environment variable is required");
    }
  }

  /**
   * Make a request to the API
   *
   * @param path API path (without base URL)
   * @param init Fetch init options
   * @returns Raw Response object from fetch
   */
  async fetch(path: string, init: RequestInit = {}): Promise<Response> {
    // Set up authentication headers
    const headers: Record<string, string> = {
      "Content-Type": "application/json",
      "Authorization": `Bearer ${this.apiKey}`
    };

    // Add headers from init if provided
    if (init.headers) {
      const initHeaders = init.headers as Record<string, string>;
      Object.keys(initHeaders).forEach(key => {
        headers[key] = initHeaders[key];
      });
    }

    // For FormData, remove Content-Type
    if (init.body instanceof FormData) {
      delete headers["Content-Type"];
    }

    // Make the request
    return fetch(`${this.baseUrl}${path}`, {
      ...init,
      headers
    });
  }

  /**
   * Helper for GET requests
   */
  async get(path: string, init: RequestInit = {}): Promise<Response> {
    return this.fetch(path, { ...init, method: "GET" });
  }

  /**
   * Helper for POST requests
   */
  async post(path: string, body: any, init: RequestInit = {}): Promise<Response> {
    const requestBody = body instanceof FormData ? body : JSON.stringify(body);
    return this.fetch(path, { ...init, method: "POST", body: requestBody });
  }

  /**
   * Helper for PUT requests
   */
  async put(path: string, body: any, init: RequestInit = {}): Promise<Response> {
    const requestBody = body instanceof FormData ? body : JSON.stringify(body);
    return this.fetch(path, { ...init, method: "PUT", body: requestBody });
  }

  /**
   * Helper for DELETE requests
   */
  async delete(path: string, init: RequestInit = {}): Promise<Response> {
    return this.fetch(path, { ...init, method: "DELETE" });
  }
}
```

## Step 4: Implement the Resource


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/darkmatter) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
