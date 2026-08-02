---
trigger: always_on
description: <!-- Purpose: Immutable bridge-specific engineering rules for discord-slack-bridge. -->
---

<!-- Purpose: Immutable bridge-specific engineering rules for discord-slack-bridge. -->

# discord-slack-bridge

## Package purpose

This package exists to let Kimaki (from the `cli` package) run on Slack in
the future with minimal behavior differences. The adapter translates Discord
Gateway and REST semantics to Slack APIs so Kimaki can keep the same runtime
model:

- Discord `guild` maps to Slack `team` (workspace).
- Discord channels map to Slack channels.
- Discord threads map to Slack threads (similar reply-thread model).

The goal is feature parity where Kimaki behaves in Slack as close as possible
to how it behaves in Discord, with this bridge handling protocol translation.

## Canonical references

- Bridge behavior spec: `slop/discord-slack-bridge-spec.md`
- Bridge implementation:
  - `discord-slack-bridge/src/server.ts`
  - `discord-slack-bridge/src/event-translator.ts`
  - `discord-slack-bridge/src/rest-translator.ts`
  - `discord-slack-bridge/src/file-upload.ts`
  - `discord-slack-bridge/src/component-converter.ts`
  - `discord-slack-bridge/src/gateway.ts`
  - `discord-slack-bridge/src/types.ts`
- Slack SDK request type references:
  - `opensrc/repos/github.com/slackapi/node-slack-sdk/packages/web-api/src/types/request/chat.ts`
  - `opensrc/repos/github.com/slackapi/node-slack-sdk/packages/web-api/src/types/request/conversations.ts`
  - `opensrc/repos/github.com/slackapi/node-slack-sdk/packages/web-api/src/types/request/reactions.ts`
  - `opensrc/repos/github.com/slackapi/node-slack-sdk/packages/web-api/src/types/request/files.ts`
  - `opensrc/repos/github.com/slackapi/node-slack-sdk/packages/web-api/src/types/request/views.ts`

## Echo bot integration smoke checks

- Use `discord-slack-bridge/scripts/echo-bot.ts` to verify end-to-end Slack + gateway behavior.
- For deployed gateway testing, run `pnpm echo-bot --gateway` from `discord-slack-bridge/`.
- This validates Discord REST + Gateway routing through `slack-gateway.kimaki.dev` and Slack webhook/interactivity handling at `/slack/events`.
- Important: this requires real user interaction in Slack. The script only starts the bridge client and registers commands; someone must send messages, run slash commands, and click interactive components in Slack to exercise Events + Interactivity webhooks end-to-end.

## Non-negotiable typing rules

- Do not use `as` assertions/casts in bridge source code.
- Do not duplicate Slack payload types when official SDK/types are available.
- Prefer `@slack/web-api` concrete request argument types for API calls
  (e.g. `satisfies ChatPostMessageArguments`).
- **Slack API response types**: use the SDK response types for all Slack API
  call results. The WebClient methods return typed responses
  (`ChatPostMessageResponse`, `ConversationsInfoResponse`, etc.) — access
  fields directly on the result (e.g. `result.ts`, `result.channel?.name`)
  instead of passing them through `Record<string, unknown>` + `readString`
  helpers. This ensures misspelled field names are caught at compile time.
- **Extracting nested Slack types**: the SDK does not re-export nested types
  like `Channel`, `User`, `MessageElement` from the main entry because they
  collide across response modules. Use indexed access on the response type:
  ```ts
  import type { ConversationsInfoResponse } from '@slack/web-api'
  type SlackChannel = NonNullable<ConversationsInfoResponse['channel']>
  ```
  See `rest-translator.ts` imports for the full set of extracted types.
- Prefer importing Slack types from the official Slack SDK instead of defining
  bridge-local copies. This keeps bridge code aligned with Slack's source of
  truth and automatically in sync when Slack updates type definitions.
- Keep inbound payload boundary normalization in `server.ts`:
  - parse as `unknown`
  - validate/narrow at runtime
  - pass normalized typed objects downstream
- The `Record<string, unknown>` + `readString`/`readRecord` pattern is ONLY
  acceptable for inbound webhook payloads from Slack Events API (raw JSON that
  needs runtime validation). Never use it for Slack SDK WebClient responses.

## Protocol/constants rules

- Avoid magic numbers and string literals for Discord protocol values.
- Prefer enums and protocol types from `discord-api-types/v10`.
- Follow payload-shaping patterns used by `discord-digital-twin`.

## ID mapping between Discord and Slack

discord.js parses certain IDs as BigInt snowflakes internally (for
`createdTimestamp`, sorting, caching). Any ID that discord.js treats as a
snowflake **must** be a valid BigInt string — non-numeric IDs like
`MSG_C04_17000...` cause `Cannot convert to BigInt` crashes at runtime.

### Which IDs must be snowflake-compatible

**Message IDs** — always parsed as BigInt by discord.js (`Snowflake.timestampFrom`
in `Message._patch`). Must be pure numeric.

**Thread channel IDs** — also parsed as snowflakes because discord.js treats
threads as channels and accesses `createdTimestamp` on them. Must be pure
numeric.

**Guild/channel/user IDs** — discord.js does NOT parse these as snowflakes in
tested code paths (only `createdTimestamp` getter would break, which typical
bot code doesn't call). These keep their Slack format as-is (`T04ABC123`,
`C04ABC123`, `U04ABC123`).

### Encoding scheme

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [remorses/kimaki](https://github.com/remorses/kimaki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
