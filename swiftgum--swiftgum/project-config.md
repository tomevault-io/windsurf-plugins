---
trigger: always_on
description: Implementing new providers
---

 # Provider Implementation Guide

This guide outlines the step-by-step process for implementing a new provider in the Swiftgum system. Follow these instructions to ensure all necessary components are properly implemented across the codebase.

## 1. Shared Repository Components

### 1.1. Define Provider Schema

Create a schema definition for your provider:

```typescript
// File: apps/shared/src/interfaces/providers/[provider-name]/index.ts

import { z } from "zod";
import { providerSchema } from "../provider";
import { storedToken } from "../shared";

// Define provider-specific task schemas
const yourProviderPendingTask = z.object({
  step: z.literal("pending"),
  // Add provider-specific fields (e.g., contentId, pageId, etc.)
  tokenId: z.string(),
  title: z.string(),
  remoteUrl: z.string(),
  // Other required fields
});

// Define the internal task schema (add more task types if needed)
const yourProviderInternalTask = z.discriminatedUnion("step", [yourProviderPendingTask]);

// Export the provider schema
export const yourProviderSchema = providerSchema({
  identifier: "your-provider" as const,
  indexingTask: storedToken,
  internalTask: yourProviderInternalTask,
});
```

### 1.2. Update Provider Interfaces

Add your provider schema to the shared interfaces:

```typescript
// File: apps/shared/src/interfaces/providers/index.ts

import { yourProviderSchema } from "./your-provider";
// ... existing imports

// Add your schema to exports
export { googleDriveSchema, notionSchema, yourProviderSchema };
export const providerSchemas = { 
  googleDriveSchema, 
  notionSchema, 
  yourProviderSchema 
};
```

### 1.3. Implement Authentication

Create an authentication implementation for your provider:

```typescript
// File: apps/shared/src/providers/[provider-name]/auth.ts

import { oauth2ProviderAuth } from "../generic/auth/oauth2";

export const yourProviderAuth = oauth2ProviderAuth<"your-provider">({
  providerId: "your-provider",
  method: "direct", // or "issuer" depending on the OAuth implementation
  authorizationUrl: "https://api.yourprovider.com/oauth/authorize",
  tokenUrl: "https://api.yourprovider.com/oauth/token",
  tokenEndpointAuthMethod: "client_secret_basic", // or other methods as needed
  issuer: "https://api.yourprovider.com",
  scope: "required-scopes", // optional, provider-specific scopes
});
```

> **Note**: If your provider doesn't use OAuth2, you may need to implement a custom authentication provider.

### 1.4. Update Provider Registry

Register your provider in the authentication registry:

```typescript
// File: apps/shared/src/providers/registry.ts

import { yourProviderAuth } from "./your-provider/auth";
// ... existing imports

// Add to authProviders array
export const authProviders = [
  googleDriveAuth,
  notionAuth,
  yourProviderAuth,
] as const satisfies ProviderAuthProvider[];

// Add to authIntegrationCredential
export const authIntegrationCredential = z.discriminatedUnion("providerId", [
  // ... existing providers
  yourProviderAuth.configurationSchema,
]);

// Add to authIntegrationAuthSession
export const authIntegrationAuthSession = z.discriminatedUnion("providerId", [
  // ... existing providers
  yourProviderAuth.authSessionSchema.merge(authIntegrationAuthSessionContext),
]);

// Add to authIntegrationCredentials
export const authIntegrationCredentials = z.discriminatedUnion("providerId", [
  // ... existing providers
  yourProviderAuth.credentialsSchema,
]);
```

## 2. Engine Repository Components

### 2.1. Implement Provider Logic

Create the main provider implementation:

```typescript
// File: apps/engine/src/providers/[provider-name]/index.ts

import { providerSchemas } from "@knowledgex/shared/interfaces";
import { provider } from "../abstract";
import { getToken } from "../../utils/token";
import { makeSgid } from "../../export";
// Import any provider-specific client libraries
import { YourProviderClient } from "@yourprovider/client";

// Initialize API client
const getYourProviderClient = ({ accessToken }) => {
  return new YourProviderClient({ auth: accessToken });
};

export const yourProviderProvider = provider({
  schema: providerSchemas.yourProviderSchema,
  
  // Process individual content items
  internal: async ({ task, exportFile }) => {
    const token = await getToken(task);
    const client = getYourProviderClient({
      accessToken: token.decrypted_tokenset.data.accessToken,
    });
    
    // Process content based on task type
    if (task.step === "pending") {
      try {
        // 1. Fetch content from the provider
        const content = await client.getContent(task.contentId);
        
        // 2. Convert to appropriate format (e.g., Markdown)
        const processedContent = convertToMarkdown(content);
        
        // 3. Export the file
        await exportFile({
          workspaceId: token.workspace_id,
          task: {
            content: processedContent,
            metadata: {
              fileId: task.contentId,
              fileName: task.title,
              remoteUrl: task.remoteUrl,
              provider: providerSchemas.yourProviderSchema.identifier,
              tokenId: task.tokenId,
              mimeType: "text/markdown", // or appropriate type
              sgid: makeSgid({
                contentSignature: processedContent,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Swiftgum/swiftgum](https://github.com/Swiftgum/swiftgum) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
