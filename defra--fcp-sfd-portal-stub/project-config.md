---
trigger: always_on
description: This is a **reference implementation** demonstrating how client portals integrate with the **SFD Document Upload Service**. It's a stub/example, not a production portal. The codebase showcases:
---

# Copilot Instructions for FCP SFD Portal Stub

## Project Overview

This is a **reference implementation** demonstrating how client portals integrate with the **SFD Document Upload Service**. It's a stub/example, not a production portal. The codebase showcases:

- OAuth2 authentication with AWS Cognito (via CDP API Gateway)
- Initiating upload sessions with business metadata via Object Processor API
- **Three upload patterns**: gateway-routing (recommended), frontend-redirect (multi-client), and direct (fallback)
- Browser-based file uploads (routing depends on mode - see Upload Modes section)
- Polling for upload status through Object Processor

**Critical Architecture**: This stub demonstrates **three different upload patterns**:
- **gateway-routing/frontend-redirect**: Files flow User Browser → Gateway → CDP Uploader → S3
- **direct mode**: Files flow User Browser → CDP Uploader → S3 (requires JavaScript)

The portal handles metadata and orchestration. In all modes, files **never pass through the portal backend** - they go directly from browser to uploader (via gateway or direct).

## Tech Stack Conventions

- **ESM modules only** - all `import`/`export`, no CommonJS (`require`/`module.exports`)
- **Hapi.js 21** - server framework with plugin architecture
- **Nunjucks** - templating engine with GOV.UK Frontend components
- **Webpack** - bundles client-side JS/SCSS from `src/client/`
- **Vitest** - testing framework with `vi` mocking utilities
- **Neostandard** - ESLint config (ECMAScript 2025)
- **Convict** - schema-based configuration management
- **Pino** - structured logging in ECS format (production) or pretty format (dev)

## Code Patterns

### Route Definitions

Routes export objects with `method`, `path`, and `handler` properties:

```javascript
export const metadataGet = {
  method: 'GET',
  path: '/document-upload/metadata',
  handler: (request, h) => {
    // Route logic
  }
}
```

All routes are auto-registered via [src/plugins/router.js](../src/plugins/router.js) which finds route files in `src/routes/`.

### Session State

Use `request.yar` (Hapi Yar plugin) for session management:

```javascript
request.yar.set('metadata', metadata)
const crn = request.yar.get('crn')
```

Session cookies configured in [src/plugins/session.js](../src/plugins/session.js).

### Configuration

Use [src/config/config.js](../src/config/config.js) convict schema. Access via:

```javascript
import { config } from './config/config.js'
const host = config.get('objectProcessor.host')
```

Override with environment variables (see `../.env.example`).

### Logging

Import `createLogger()` from [src/common/helpers/logging/logger.js](../src/common/helpers/logging/logger.js):

```javascript
const logger = createLogger()
logger.info({ submissionId }, 'Initiating upload')
logger.error({ error, statusUrl }, 'Status check failed')
```

Use structured logging with context objects.

### Object Processor Integration

Two main API calls in [src/common/helpers/object-processor.js](../src/common/helpers/object-processor.js):

1. **`initiateUpload(metadata)`** - POST `/api/v1/initiate` with business metadata, returns `{correlationId, uploadId, uploadUrl, statusUrl}`
2. **`getUploadStatus(statusUrl)`** - GET status URL to poll upload progress

Both automatically include OAuth2 Bearer token when Cognito enabled (`COGNITO_ENABLED=true`).

### Upload Modes

The portal supports **three upload patterns** controlled by `UPLOAD_MODE` environment variable:

#### 1. Gateway Routing Mode (Default, Recommended)
- **User access**: `http://localhost:3019` (nginx gateway)
- **Upload flow**: Browser → Gateway → CDP Uploader (standard HTML form, no JavaScript)
- **Redirect handling**: Relative redirects stay within gateway domain
- **CSP**: Simple - only `'self'` needed
- **Progressive enhancement**: ✅ Works without JavaScript
- **Best for**: CDP services, services with gateway infrastructure

#### 2. Frontend Redirect Mode (Multi-Client)
- **User access**: `http://localhost:3020` (portal directly)
- **Upload flow**: Browser → Gateway → CDP Uploader (standard HTML form, no JavaScript)
- **Redirect handling**: Gateway routes to document-upload-frontend-stub which maps client identifiers to absolute URLs
- **CSP**: Requires gateway domain in `form-action` directive
- **Progressive enhancement**: ✅ Works without JavaScript
- **Best for**: Multi-client scenarios, external services needing centralized redirect mapping
- **Key files**: `document-upload-frontend-stub/` service

#### 3. Direct Mode (Fallback Only)
- **User access**: `http://localhost:3020` (portal directly)
- **Upload flow**: Browser → CDP Uploader (JavaScript fetch with redirect interception)
- **Redirect handling**: JavaScript detects opaque redirect and manually navigates
- **CSP**: Requires uploader domain in `form-action` and `connect-src`
- **Progressive enhancement**: ❌ Requires JavaScript - fails without client-side code
- **Best for**: Situations where gateway infrastructure is not available (use as last resort)
- **Implementation**: [src/client/javascript/document-upload.js](../src/client/javascript/document-upload.js)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DEFRA/fcp-sfd-portal-stub](https://github.com/DEFRA/fcp-sfd-portal-stub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
