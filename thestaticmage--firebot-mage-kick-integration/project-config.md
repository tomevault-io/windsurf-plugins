---
trigger: always_on
description: Integration for Firebot <https://github.com/crowbartools/firebot> to Kick live streaming.
---

# Firebot Kick Integration

Integration for Firebot <https://github.com/crowbartools/firebot> to Kick live streaming.

Instructions:

- When finished with a task, display a summary that is at most 3 sentences long.
- Do not display a detailed summary or create markdown files unless explicitly instructed to do so.
- Update this CLAUDE.md file when:
  - Implementing significant new features or architectural changes
  - Discovering new patterns, insights, or learnings about the codebase
  - Completing major phases or milestones
  - Establishing new testing patterns or coverage strategies
  - Finding optimizations or improvements to existing approaches
  - Clarifying ambiguities in existing conventions or practices

Key features:

- Authenticate to Kick via OAuth with automatic token refreshing
- Chat message integration using Pusher WebSocket service
- Chat messages from Kick show up in Firebot chat feed (dashboard)
- Chat effect that posts messages into Kick chat
- Stream events via webhook notifications (channel-based configuration)
- Effects for stream management (title/category changes, banning/timing out users)
- Comprehensive event support (follows, subscriptions, raids, hosts, kicks, etc.)

TODO:

- Extend event and effect support as the Kick public API expands

Tech: TypeScript, Jest

Learnings:

- Kick's public API is limited and incomplete; some events require undocumented Pusher WebSocket service
- OAuth tokens require automatic refresh to maintain authentication
- Kick has a "private API" which we do not use as it requires bypassing CloudFlare protections by pretending to be a browser
- Webhook delivery can be delayed by seconds or minutes on Kick's side
- **IPC Communication Patterns in Firebot UI Extensions:**
  - `frontendCommunicator` is used in backend (src/integration-singleton.ts) to communicate with frontend
  - `backendCommunicator` is injected into AngularJS controllers on the frontend
  - `fireEventSync()` calls must match synchronous handlers (`on()`) - returns values synchronously
  - `fireEvent()` calls must match asynchronous handlers (`onAsync()`) - handles return values via Promise
  - Never use `fireEventSync()` with `onAsync()` handlers - causes application to hang
  - Never use `fireEvent()` with `on()` handlers - ignores the return value
  - Electron IPC cannot serialize `undefined` values - always use empty strings ("") for tokens and 0 for numeric defaults
  - For handlers that need to send data immediately (like auth URLs), use synchronous `on()` with `fireEventSync()` from frontend
  - For handlers with async operations (deauth, API calls), use `onAsync()` with `fireEvent()` from frontend
  - Backend sends unsolicited updates via `frontendCommunicator.send()` which the frontend listens for with `backendCommunicator.on()`
- **Clipboard API in UI Extensions:**
  - `navigator.clipboard.writeText()` requires document focus and fails with `NotAllowedError` if document is not focused
  - Browser confirm dialogs can cause document to lose focus, breaking subsequent clipboard operations
  - Solution: call `window.focus()` before clipboard operations and provide fallback using `document.execCommand('copy')` with temporary textarea
  - Always use Angular's `$timeout` instead of native `setTimeout` for button state management in AngularJS controllers
- **Webhook Signature Verification:**
  - Production webhooks use RSA-SHA256 signatures (base64-encoded)
  - Test webhooks use Ed25519 signatures (hex-encoded)
  - Signature verification is in `src/internal/webhook-handler/webhook-signature-verifier.ts`
  - Test fixtures for keys are in `src/internal/webhook-handler/__fixtures__/` (checked into git)
  - Real production keys in `IntegrationConstants.WEBHOOK_PUBLIC_KEY`
  - Real test webhook keys: `private_key.pem` (git-ignored), `IntegrationConstants.TEST_WEBHOOK_PUBLIC_KEY`
  - Production webhook signature input: `${messageId}.${timestamp}.${JSON.stringify(payload)}`
  - Test webhook signature input: `JSON.stringify(payload)`

Conventions:

- TypeScript: camelCase, PascalCase classes, satisfies eslint rules defined in package
- "Kick": Capitalize as "Kick" (or "kick" in lowercase contexts where appropriate)
- Logging: Provide observability via logger.debug
- Documentation: In Markdown, placed in `doc` directory, referenced from `README.md`, satisfies markdownlint
- Build: Code consolidated to one file with webpack (webpack file loaded by Firebot as startup script)
- User ID: UserIDs from Kick are 'k' plus the given Kick user ID
- User name: Usernames from Kick are the given Kick username plus '@kick'
- No emojis in log messages or code comments
- Emojis are acceptable in documentation but must use GitHub markdown emojis (e.g. `:white_check_mark:`)
- No emdashes anywhere (code, comments, or documentation)
- Do not leave comments that only indicate something was removed
- Use comments to explain "why" or as headers before sections of code but do not leave obvious comments that describe short and straightforward implementation
- If something is being removed, remove it completely. Do not worry about backward compatibility or deprecation unless specifically instructed.

Tests:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TheStaticMage) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
