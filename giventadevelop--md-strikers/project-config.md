---
trigger: always_on
description: This document outlines the correct patterns for implementing email functionality in the Next.js application, based on the working QR code endpoint pattern and successful send-ticket-email implementation.
---

# Email Endpoint Implementation Rules

## Overview
This document outlines the correct patterns for implementing email functionality in the Next.js application, based on the working QR code endpoint pattern and successful send-ticket-email implementation.

## Key Principles

### 1. Base64 Encoding for emailHostUrlPrefix
**CRITICAL:** Always use Base64 encoding for `emailHostUrlPrefix` in URL paths, NOT URL encoding.

```typescript
// ✅ CORRECT - Base64 encoding like working QR code endpoint
const encodedEmailHostUrlPrefix = Buffer.from(emailHostUrlPrefix).toString('base64');

// ❌ WRONG - URL encoding (causes backend issues)
const encodedEmailHostUrlPrefix = encodeURIComponent(emailHostUrlPrefix);
```

### 2. URL Pattern Structure
Follow the exact same URL structure as the working QR code endpoint:

```
/api/events/{eventId}/transactions/{transactionId}/emailHostUrlPrefix/{base64EncodedUrlPrefix}/{action}
```

**Working Examples:**
- QR Code: `/api/events/3/transactions/5652/emailHostUrlPrefix/aHR0cDovL2xvY2FsaG9zdDozMDAw/qrcode`
- Email: `/api/events/1/transactions/4956/emailHostUrlPrefix/aHR0cDovL2xvY2FsaG9zdDozMDAw/send-ticket-email`

Where `aHR0cDovL2xvY2FsaG9zdDozMDAw` is Base64 for `http://localhost:3000`

## Implementation Patterns

### 1. Custom Proxy Handler Pattern
Use the shared `createProxyHandler` with custom backend path construction:

```typescript
import { createProxyHandler } from '@/lib/proxyHandler';
import { getEmailHostUrlPrefix } from '@/lib/env';

export default async function handler(req: any, res: any) {
  const { id, transactionId } = req.query;

  // Get emailHostUrlPrefix from request headers or use default
  const emailHostUrlPrefix = req.headers['x-email-host-url-prefix'] as string ||
                           getEmailHostUrlPrefix();

  // CRITICAL: Use Base64 encoding like the working QR code endpoint
  const encodedEmailHostUrlPrefix = Buffer.from(emailHostUrlPrefix).toString('base64');

  // Create custom backend path with Base64-encoded emailHostUrlPrefix
  const customBackendPath = `/api/events/${id}/transactions/${transactionId}/emailHostUrlPrefix/${encodedEmailHostUrlPrefix}/send-ticket-email`;

  // Use shared proxy handler
  const proxyHandler = createProxyHandler({
    backendPath: customBackendPath,
    allowedMethods: ['POST', 'GET'],
    injectTenantId: false
  });

  return proxyHandler(req, res);
}
```

### 2. Server Action Pattern
For server-side API calls (like in ApiServerActions.ts):

```typescript
export async function sendTicketEmail(eventId: number, transactionId: number): Promise<boolean> {
  const baseUrl = process.env.NEXT_PUBLIC_APP_URL || 'http://localhost:3000';
  const emailHostUrlPrefix = getEmailHostUrlPrefix();
  
  // CRITICAL: Use Base64 encoding
  const encodedEmailHostUrlPrefix = Buffer.from(emailHostUrlPrefix).toString('base64');

  const response = await fetchWithJwtRetry(
    `${baseUrl}/api/proxy/events/${eventId}/transactions/${transactionId}/emailHostUrlPrefix/${encodedEmailHostUrlPrefix}/send-ticket-email`,
    {
      method: 'GET', // or 'POST' depending on backend
      headers: {
        'Content-Type': 'application/json',
      }
    }
  );

  return response.ok;
}
```

### 3. Client-Side Pattern
For frontend API calls:

```typescript
// In client components, pass emailHostUrlPrefix via headers
const handleResendEmail = async (email: string) => {
  const response = await fetch(`/api/proxy/events/${eventId}/transactions/${transactionId}/send-ticket-email?to=${encodeURIComponent(email)}`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'x-email-host-url-prefix': window.location.origin, // Will be Base64 encoded by proxy
    },
  });
  
  return response.ok;
};
```

## Email Host URL Prefix Handling

### 1. Header-Based Approach (Recommended)
Pass `emailHostUrlPrefix` via request headers and let the proxy handler encode it:

```typescript
// Client side - pass in header
headers: {
  'x-email-host-url-prefix': 'http://localhost:3000'
}

// Proxy handler - get from header and encode
const emailHostUrlPrefix = req.headers['x-email-host-url-prefix'] as string ||
                         getEmailHostUrlPrefix();
const encoded = Buffer.from(emailHostUrlPrefix).toString('base64');
```

### 2. Direct Encoding (For Server Actions)
When calling from server actions, encode directly:

```typescript
const emailHostUrlPrefix = getEmailHostUrlPrefix();
const encoded = Buffer.from(emailHostUrlPrefix).toString('base64');
```

## Backend API Endpoint Patterns

Based on working examples, the backend likely expects these patterns:

### Working Endpoints
- **QR Code Generation:** `GET /api/events/{eventId}/transactions/{transactionId}/emailHostUrlPrefix/{base64}/qrcode`
- **Email Sending:** `GET|POST /api/events/{eventId}/transactions/{transactionId}/emailHostUrlPrefix/{base64}/send-ticket-email`

### Query Parameters
Additional parameters can be passed as query strings:
```
/send-ticket-email?to=user@example.com&subject=Custom Subject
```

## Common Mistakes to Avoid

### ❌ Wrong Encoding
```typescript
// NEVER use URL encoding for emailHostUrlPrefix in paths

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
