---
trigger: always_on
description: ElizaOS API / Client Architecture
---

> You are an expert in creating robust API client integrations for ElizaOS plugins. You focus on writing clear, maintainable, and resilient code that follows best practices for security, error handling, and performance.

## API Client Architecture

When a plugin needs to communicate with an external API, it should encapsulate all API logic within a dedicated client module. This promotes reusability and separates API concerns from the core plugin logic (actions, providers, etc.).

```mermaid
graph TD
    A[Plugin Component] -->|Calls Action/Provider| B(Action/Provider);
    B -->|Calls API Client Method| C(API Client Module);
    C -->|Validates Config| D(Environment Util);
    D -->|API Key, Base URL| E[process.env / runtime.getSetting()];
    C -->|Builds Request| F(Request Builder);
    C -->|Makes HTTP Call| G(HTTP Library e.g., Axios/Fetch);
    G --> H[External API];
    H --> G;
    G -->|Handles Response/Error| C;
    C --> B;
    B --> A;
```

## Project Structure

Organize your API client logic into a dedicated directory within your plugin's `src`.

```
plugin-my-api/
├── src/
│   ├── index.ts               # Main plugin definition
│   ├── actions/               # Actions that use the client
│   │   └── someAction.ts
│   └── client/                # All API client logic
│       ├── index.ts           # Exports client functions
│       ├── request.ts         # Core request/sender function
│       ├── builder.ts         # Request payload builder
│       ├── validator.ts       # API key and param validation
│       └── types.ts           # TypeScript types for API req/res
├── ...
└── package.json
```

## Core Implementation Patterns

This pattern is based on the implementation in `@elizaos/plugin-openai`.

### 1. Validator (`validator.ts`)

Create functions to validate configuration and parameters before making an API call. This fails fast and provides clear errors.

```typescript
// src/client/validator.ts

// ✅ DO: Validate the presence and format of the API key
export function validateApiKey(runtime: IAgentRuntime): string {
  const apiKey = runtime.getSetting("MY_API_KEY");
  if (!apiKey) {
    throw new Error("MY_API_KEY is not set in the .env file or agent settings.");
  }
  return apiKey;
}

// ✅ DO: Validate the input parameters for an API call
export function validatePrompt(prompt: string): void {
  if (!prompt || !prompt.trim()) {
    throw new Error("Prompt cannot be empty.");
  }
  if (prompt.length > 8000) { // Example limit
    throw new Error("Prompt exceeds maximum length.");
  }
}
```

### 2. Request Builder (`builder.ts`)

A builder function separates the logic of creating the request payload from the act of sending it. It should handle defaults and parameter mapping.

```typescript
// src/client/builder.ts
import { DEFAULT_MODEL, DEFAULT_MAX_TOKENS } from "./constants";
import { type MyApiRequestData } from "./types";

// ✅ DO: Centralize request payload creation
export function buildRequestData(
  prompt: string,
  model: string = DEFAULT_MODEL,
  maxTokens: number = DEFAULT_MAX_TOKENS
): MyApiRequestData {
  return {
    model,
    prompt,
    max_tokens: maxTokens,
  };
}
```

### 3. Core Request Function (`request.ts`)

This is the heart of the client. It uses an HTTP library like `axios` or `fetch` to make the actual API call, handling headers, authentication, and errors.

```typescript
// src/client/request.ts
import axios, { type AxiosRequestConfig } from "axios";
import { type MyApiRequestData, type MyApiResponse } from "./types";
import { API_BASE_URL, DEFAULT_TIMEOUT } from "./constants";

// ✅ DO: Create a single, reusable function to call the API
export async function callMyApi<T extends MyApiResponse>(
  endpoint: string,
  data: MyApiRequestData,
  apiKey: string,
): Promise<T> {
  try {
    const config: AxiosRequestConfig = {
      headers: {
        'Authorization': `Bearer ${apiKey}`,
        'Content-Type': 'application/json',
      },
      timeout: DEFAULT_TIMEOUT,
    };

    const response = await axios.post<T>(`${API_BASE_URL}/${endpoint}`, data, config);
    return response.data;

  } catch (error) {
    console.error("Error communicating with My API:", error.message);
    
    // ✅ DO: Handle specific API errors like rate limiting
    if (axios.isAxiosError(error) && error.response?.status === 429) {
      throw new Error("My API rate limit exceeded. Please try again later.");
    }

    // Throw a generic error for other issues
    throw new Error("Failed to communicate with My API.");
  }
}
```

### 4. Putting It Together in an Action

The plugin's actions (or providers) use the exported client functions to perform their logic.

```typescript
// src/actions/someAction.ts
import { type Action, type IAgentRuntime } from "@elizaos/core";
import { 
  validateApiKey, 
  validatePrompt, 
  buildRequestData, 
  callMyApi 
} from '../client';

export const someAction: Action = {
  name: "do-something-with-api",
  description: "An action that calls My API.",
  async handler(runtime: IAgentRuntime, message) {
    // 1. Validate config and input
    const apiKey = validateApiKey(runtime);
    const prompt = message.content.text || "";
    validatePrompt(prompt);

    // 2. Build the request data
    const requestData = buildRequestData(prompt);

    // 3. Call the API

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bio-xyz) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
