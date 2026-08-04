---
trigger: always_on
description: This is the shared project guide for people and AI coding assistants working in this repository. Keep it safe to commit: add only project-level guidance, never personal preferences, local machine paths, private credentials, copied calendar data, or session artifacts.
---

# TimeTree MCP Project Guide

This is the shared project guide for people and AI coding assistants working in this repository. Keep it safe to commit: add only project-level guidance, never personal preferences, local machine paths, private credentials, copied calendar data, or session artifacts.

## Project Purpose

TimeTree MCP is an unofficial Model Context Protocol (MCP) server for TimeTree. It connects MCP-compatible clients to TimeTree calendar data through TimeTree web APIs.

Important context:

- This project is not affiliated with, endorsed by, or supported by TimeTree.
- The API behavior is based on observed TimeTree web app behavior and may change without notice.
- `package.json` must keep `"private": true` unless maintainers explicitly decide to publish.
- Do not publish this package to npm or an MCP marketplace without maintainer approval.
- Treat upstream endpoints, cookies, CSRF tokens, and session identifiers as implementation details.

## Current Capabilities

The server currently exposes tools for:

- Listing calendars.
- Reading calendar events and updated events.
- Creating, updating, and deleting events.
- Managing event comments.
- Listing and managing memos.
- Reading and updating calendar labels.
- Reading calendar members and virtual members.

Write operations depend on TimeTree web authentication and CSRF handling. Keep error messages clear, but do not expose internal auth material.

## Architecture Map

- `src/index.ts` - MCP server entry point and request routing.
- `src/config/config.ts` - TimeTree constants, API paths, and client metadata.
- `src/client/auth.ts` - Email/password authentication, CSRF extraction, and session state coordination.
- `src/client/api.ts` - TimeTree HTTP client operations, pagination, and domain-specific API methods.
- `src/tools/` - MCP tool definitions and input/output shaping.
- `src/types/` - Zod schemas and shared TimeTree types.
- `src/utils/http-client.ts` - HTTP wrapper, headers, cookies, and CSRF token handling.
- `src/utils/logger.ts` - Structured logging and sensitive-data masking.
- `src/utils/rate-limiter.ts` - Token-bucket request throttling.

Prefer small changes that follow these boundaries instead of adding new layers or dependencies.

## API Behavior Notes

- TimeTree event sync uses `since` as an incremental sync cursor, not as a date filter.
- Date filtering for general event reads should happen client-side with tool parameters such as `start_after`.
- Event timestamps are Unix timestamps in milliseconds; user-facing output should prefer ISO 8601 strings when helpful.
- The upstream API may return mixed types for the same field. Zod schemas should use flexible unions where observed and `.passthrough()` for unknown fields.
- Label IDs `1` through `10` map to the color metadata in `src/types/label-colors.ts`.

## Security and Privacy Rules

- Never commit real `TIMETREE_EMAIL`, `TIMETREE_PASSWORD`, cookies, CSRF tokens, session IDs, request captures, or personal calendar content.
- Do not create or commit `.env` files. Credentials must come from MCP client environment configuration.
- Session cookies and CSRF tokens should remain in memory only.
- All logs must go to stderr. MCP uses stdout for JSON-RPC, so `console.log()` can break protocol output.
- Use the shared logger so sensitive fields are masked consistently.
- Do not expose upstream implementation details in normal MCP tool responses unless they are necessary for debugging and safe to share.

## Development Workflow

```bash
npm install
npm run typecheck
npm test
npm run build
```

For manual MCP inspection:

```bash
npm run build
npx @modelcontextprotocol/inspector node dist/index.js
```

When testing manually with a real account, keep credentials only in the shell or MCP client environment. Do not paste real responses into committed fixtures or documentation.

## Documentation Standards

- Keep `README.md`, `README.ko.md`, and `README.ja.md` aligned for user-facing changes.
- Keep client setup details in `docs/MCP_CLIENTS.md` and update the installer output when setup instructions change.
- Keep shell script output in English unless maintainers decide otherwise.
- Use generic placeholders such as `your-email@example.com`, `your-password`, and `/absolute/path/to/...`.
- Avoid contributor-facing text that depends on a maintainer's local environment.

## Tool Design Guidelines

- Tool descriptions should be short and task-oriented.
- Validate inputs with Zod and return clear errors through MCP content responses.
- Keep output useful but bounded; support limits and filters for potentially large calendars.
- Avoid returning raw upstream payloads. Shape responses into stable, documented fields.
- For write tools, clearly describe side effects and required identifiers.

## Testing Expectations

- Prefer fixture or mock tests over live API tests.
- Add regression tests for parsing, logging, masking, tool output shape, and sync/pagination behavior.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ehs208/TimeTree-MCP](https://github.com/ehs208/TimeTree-MCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->
