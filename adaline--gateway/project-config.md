---
trigger: always_on
description: The types package contains shared type definitions, Zod schemas, and interfaces used across the entire Adaline Gateway ecosystem. This package serves as the single source of truth for all data structures.
---


# Types Package Rules & Instructions

## Package Overview

The types package contains shared type definitions, Zod schemas, and interfaces used across the entire Adaline Gateway ecosystem. This package serves as the single source of truth for all data structures.

## Type Definition Rules

### 1. Schema-First Approach

- **ALWAYS** define Zod schemas first, then derive TypeScript types
- **ALWAYS** export both the schema and the inferred type
- **ALWAYS** use descriptive schema names ending with descriptive suffixes
- **NEVER** define types without corresponding schemas

### 2. Naming Conventions

- **ALWAYS** use PascalCase for schema names: `ChatResponse`, `MessageContent`
- **ALWAYS** use PascalCase + `Type` suffix for type names: `ChatResponseType`, `MessageContentType`
- **ALWAYS** use descriptive names that clearly indicate purpose
- **ALWAYS** follow the pattern: `{SchemaName}` and `{SchemaName}Type`

## Schema Definition Instructions

### 1. Basic Schema Structure

```typescript
// ✅ CORRECT: Define schema first, then type
const ChatResponse = z.object({
  messages: z.array(Message()),
  usage: ChatUsage.optional(),
  logProbs: ChatLogProbs.optional(),
});
type ChatResponseType = z.infer<typeof ChatResponse>;

// ❌ INCORRECT: Define type without schema
type ChatResponseType = {
  messages: MessageType[];
  usage?: ChatUsageType;
  logProbs?: ChatLogProbsType;
};
```

### 2. Schema Composition

```typescript
// ✅ CORRECT: Compose schemas from smaller parts
const ChatUsage = z.object({
  promptTokens: z.number().nonnegative(),
  completionTokens: z.number().nonnegative(),
  totalTokens: z.number().nonnegative(),
});

const ChatResponse = z.object({
  messages: z.array(Message()),
  usage: ChatUsage.optional(),
});

// ❌ INCORRECT: Duplicate schema definitions
const ChatResponse = z.object({
  messages: z.array(Message()),
  usage: z.object({
    promptTokens: z.number().nonnegative(),
    completionTokens: z.number().nonnegative(),
    totalTokens: z.number().nonnegative(),
  }).optional(),
});
```

### 3. Optional vs Required Fields

```typescript
// ✅ CORRECT: Use .optional() for truly optional fields
const Message = z.object({
  role: z.enum(['user', 'assistant', 'system']), // Required
  content: z.array(MessageContent()), // Required
  name: z.string().optional(), // Optional
  toolCalls: z.array(ToolCall()).optional(), // Optional
});

// ❌ INCORRECT: Making required fields optional
const Message = z.object({
  role: z.enum(['user', 'assistant', 'system']).optional(), // Should be required
  content: z.array(MessageContent()).optional(), // Should be required
});
```

## Type Export Instructions

### 1. Export Pattern

```typescript
// ✅ CORRECT: Export both schema and type
export {
  ChatResponse,
  ChatUsage,
  type ChatResponseType,
  type ChatUsageType,
};

// ❌ INCORRECT: Export only types
export type {
  ChatResponseType,
  ChatUsageType,
};
```

### 2. Index File Organization

```typescript
// ✅ CORRECT: Re-export from subdirectories
export * from "./chat";
export * from "./config";
export * from "./embedding";
export * from "./errors";
export * from "./message";
export * from "./pricing";
export * from "./tool";
export * from "./utils";

// ❌ INCORRECT: Export individual items
export { ChatResponse, ChatUsage } from "./chat";
export { ConfigType } from "./config";
```

## Validation Instructions

### 1. Input Validation

```typescript
// ✅ CORRECT: Validate input using schemas
function processChatResponse(data: unknown): ChatResponseType {
  return ChatResponse.parse(data);
}

// ❌ INCORRECT: Type assertion without validation
function processChatResponse(data: unknown): ChatResponseType {
  return data as ChatResponseType; // Unsafe!
}
```

### 2. Safe Parsing

```typescript
// ✅ CORRECT: Use safeParse for error handling
const result = ChatResponse.safeParse(data);
if (result.success) {
  // result.data is ChatResponseType
  return result.data;
} else {
  // result.error contains validation errors
  throw new ValidationError(result.error.errors);
}

// ❌ INCORRECT: Ignoring validation errors
try {
  return ChatResponse.parse(data);
} catch (error) {
  // Error handling is missing
  throw error;
}
```

## Schema Design Instructions

### 1. Union Types

```typescript
// ✅ CORRECT: Use discriminated unions for different content types
const MessageContent = z.discriminatedUnion('type', [
  z.object({
    type: z.literal('text'),
    text: z.string(),
  }),
  z.object({
    type: z.literal('image'),
    imageUrl: z.string().url(),
    altText: z.string().optional(),
  }),
]);

// ❌ INCORRECT: Simple union without discrimination
const MessageContent = z.union([
  z.string(),
  z.object({ imageUrl: z.string() }),
]);
```

### 2. Constrained Types

```typescript
// ✅ CORRECT: Use appropriate constraints
const TokenCount = z.number().int().nonnegative();
const ModelName = z.string().min(1).max(100);
const ApiKey = z.string().regex(/^sk-[a-zA-Z0-9]{32,}$/);

// ❌ INCORRECT: Overly permissive types
const TokenCount = z.number(); // Allows negative numbers
const ModelName = z.string(); // Allows empty strings

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [adaline/gateway](https://github.com/adaline/gateway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
