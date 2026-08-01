---
trigger: always_on
description: Instructions for AI coding agents working on this package.
---

# AGENTS.md — @slack/web-api

Instructions for AI coding agents working on this package.

## Package Overview

The core Slack Web API client. Wraps every Slack API method with typed request arguments and response types, automatic retries, rate limiting, cursor pagination, and streaming support.

## Architecture

`src/WebClient.ts` extends the `Methods` class (`src/methods.ts`) which defines all API method bindings. Key internals:

- **Axios** for HTTP requests
- **p-queue** for request concurrency control
- **p-retry** for automatic retries with backoff
- **ChatStreamer** (`src/ChatStreamer.ts`) for streaming support via `chatStream()`
- Built-in cursor pagination via `paginate()`
- Rate limiting with emitted events

## Critical Rules

1. **Never manually edit `src/types/response/*.ts`** — these are auto-generated. Look for the "DO NOT EDIT" banner.
2. **Request types ARE manually maintained** — `src/types/request/` is hand-written code.
3. **Use Biome**, not ESLint/Prettier — config is in `biome.json` at repo root.
4. **Response type imports use individual files**, not the barrel — e.g., `import type { ChatPostMessageResponse } from './types/response/ChatPostMessageResponse'`.
5. **Request type imports use the barrel** — `import type { ChatPostMessageArguments } from './types/request'`.

## Naming Conventions

- **Types**: PascalCase
  - Request: `{Namespace}{Action}Arguments` (e.g., `ChatPostMessageArguments`)
  - Response: `{Namespace}{Action}Response` (e.g., `ChatPostMessageResponse`)
- **Methods**: camelCase matching the API
  - e.g., `chat.postMessage` -> `postMessage`

## Development Philosophy

- **Follow existing patterns exactly** — when adding a new method, match the style of adjacent methods.
- **Reuse mixins** from `common.ts` and namespace-specific files rather than duplicating field definitions.
- **Every API method needs four things**: request type, response type, method binding, and type tests.
- **JSDoc on method bindings**: Always include `@description` and `@see` with a link to the API reference.

## Adding a New Slack API Method

This is the most common contribution. Follow these steps in order.

### Step 1: Look Up the API Method Documentation

Reference: `https://docs.slack.dev/reference/methods/{method.name}`

For example, for `chat.appendStream`: https://docs.slack.dev/reference/methods/chat.appendStream

### Step 2: Generate Response Types

```bash
bash scripts/generate-web-api-types.sh
```

This script:

1. Clones/updates `slackapi/java-slack-sdk` into `tmp/java-slack-sdk`
2. Reads JSON response samples from `tmp/java-slack-sdk/json-logs/samples/api/`
3. Runs `scripts/code_generator.rb` which uses quicktype to generate TypeScript types
4. Outputs `*Response.ts` files to `src/types/response/`
5. Regenerates `src/types/response/index.ts`
6. Runs `npm run lint:fix` on the generated files

**Prerequisites**: Ruby and npm must be installed.

Generated response types look like this (example: `ChatAppendStreamResponse.ts`):

```typescript
import type { WebAPICallResult } from '../../WebClient';
export type ChatAppendStreamResponse = WebAPICallResult & {
  channel?: string;
  error?: string;
  needed?: string;
  ok?: boolean;
  provided?: string;
  ts?: string;
};
```

All generated files extend `WebAPICallResult` and have all properties optional.

**Note**: If the JSON sample doesn't exist yet in `java-slack-sdk`, the API method has not been added to that project yet. The maintainers need to add it there first before it can be added here.

### Step 3: Add Request Argument Types

File: `src/types/request/<namespace>.ts`

Create an interface/type for the method's arguments. Reuse mixins from `src/types/request/common.ts`:

| Mixin | Purpose |
|-------|---------|
| `TokenOverridable` | Optional `token` override |
| `CursorPaginationEnabled` | `cursor` + `limit` pagination |
| `TimelinePaginationEnabled` | `oldest` + `latest` + `inclusive` pagination |
| `TraditionalPagingEnabled` | `count` + `page` pagination |
| `OptionalTeamAssignable` | Optional `team_id` |
| `LocaleAware` | Optional `include_locale` |

Also reuse namespace-specific mixins from the same file (e.g., `Channel`, `ChannelAndTS`, `AsUser` in `chat.ts`).

**Example** — `ChatAppendStreamArguments` from `src/types/request/chat.ts`:

```typescript
export interface ChatAppendStreamArguments extends TokenOverridable, ChannelAndTS, Partial<MarkdownText> {
  /**
   * @description An array of chunk objects to append to the stream.
   * Either `markdown_text` or `chunks` is required.
   */
  chunks?: AnyChunk[];
}
```

### Step 4: Export Request Types

File: `src/types/request/index.ts`

Add the new type to the appropriate export block:

```typescript
export type {
  ChatAppendStreamArguments,
  ChatStartStreamArguments,
  ChatStopStreamArguments,
  // ... existing exports
} from './chat';
```

### Step 5: Add Method Binding

File: `src/methods.ts`

1. Import the argument and response types at the top of the file.
2. Add a method binding in the appropriate namespace object of the `Methods` class.

**Import pattern**:

```typescript
// Request types are imported from the request index barrel
import type {
  ChatAppendStreamArguments,
  // ...
} from './types/request';

// Response types are imported from individual files

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [slackapi/node-slack-sdk](https://github.com/slackapi/node-slack-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
