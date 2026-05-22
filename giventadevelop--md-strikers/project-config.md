---
trigger: always_on
description: Rules for Next.js API route structure and security
---


- **All API routes must...**
  - Use authentication middleware
  - Return JSON responses
  - etc.
- **No local DTO/interface declarations**
  - All DTOs should be imported from `@/types` if needed (none should be declared locally)
  - No DTO redeclaration: All DTOs should be imported from `@/types` (none needed in this handler)
  - Example:
    ```typescript
    // ✅ DO: Import DTOs
    import type { UserProfileDTO } from '@/types';
    // ❌ DON'T: Redeclare DTOs
    // interface UserProfileDTO { ... }
    ```

- **Consistent environment variable**
  - Use `process.env.NEXT_PUBLIC_API_BASE_URL` for the backend API base URL, matching the rest of your proxy routes
  - Do not use hardcoded URLs or other env vars for backend base URL
  - Example:
    ```typescript
    const API_BASE_URL = process.env.NEXT_PUBLIC_API_BASE_URL;
    ```

- **Query string handling**
  - Use a `buildQueryString` helper to forward all query params, just like in the user profile proxy
  - Ensures all filters, pagination, and sorts are preserved
  - Example:
    ```typescript
    function buildQueryString(query: Record<string, any>) {
      const params = new URLSearchParams();
      for (const key in query) {
        const value = query[key];
        if (Array.isArray(value)) {
          value.forEach(v => params.append(key, v));
        } else if (typeof value !== 'undefined') {
          params.append(key, value);
        }
      }
      return params.toString();
    }
    ```

- **Do NOT add tenantId.equals in your client/server code when calling the proxy**
  - The proxy handler will always inject tenantId.equals automatically.
  - Only add tenantId.equals if you are calling the backend API directly (not via /api/proxy/...).
  - This prevents duplicate tenantId.equals parameters and backend criteria errors.
  - Example:
    ```typescript
    // ✅ DO: Only add email.equals or userId.equals
    const params = new URLSearchParams({ 'email.equals': email });
    await fetch('/api/proxy/user-profiles?' + params.toString());
    // ❌ DON'T: Add tenantId.equals when calling the proxy
    const params = new URLSearchParams({ 'email.equals': email, 'tenantId.equals': tenantId });
    await fetch('/api/proxy/user-profiles?' + params.toString()); // Will result in duplicate tenantId
    ```

- **JWT handling**
  - Use `fetchWithJwtRetry` **from `@/lib/proxyHandler`** for all backend calls (it adds **`Authorization`** and **`X-Tenant-ID`**; see **X-Tenant-ID for tenant-scoped backend APIs** below). Older inline examples may omit `X-Tenant-ID` — do not copy them for direct backend URLs.
  - Do not call backend APIs directly with fetch; always use the helper
  - Example:
    ```typescript
    import { getCachedApiJwt, generateApiJwt } from '@/lib/api/jwt';
    async function fetchWithJwtRetry(apiUrl: string, options: any = {}, debugLabel = '') {
      let token = await getCachedApiJwt();
      let response = await fetch(apiUrl, {
        ...options,
        headers: {
          ...options.headers,
          Authorization: `Bearer ${token}`,
        },
      });
      if (response.status === 401) {
        token = await generateApiJwt();
        response = await fetch(apiUrl, {
          ...options,
          headers: {
            ...options.headers,
            Authorization: `Bearer ${token}`,
          },
        });
      }
      return response;
    }
    ```

- **Error handling**
  - Catch and log errors, returning a 500 with a clear message if something goes wrong
  - Example:
    ```typescript
    try {
      // ...
    } catch (err) {
      console.error('Proxy error:', err);
      res.status(500).json({ error: 'Internal server error', details: String(err) });
    }
    ```

- **Method handling**
  - Only allow GET and POST (or appropriate methods), with proper 405 responses for others
  - Example:
    ```typescript
    if (req.method === 'GET') { /* ... */ }
    else if (req.method === 'POST') { /* ... */ }
    else {
      res.setHeader('Allow', ['GET', 'POST']);
      res.status(405).end(`Method ${req.method} Not Allowed`);
    }
    ```

- **Required backend fields for create operations**
  - When creating resources via proxy API routes, all fields required by the backend (including timestamps like `createdAt` and `updatedAt`) must be included in the payload, even if not set by the client.
  - The proxy or client must ensure these fields are present to avoid backend validation errors (e.g., Spring Boot will reject null `createdAt`/`updatedAt`).
  - Example for ticket type creation:
    ```typescript
    // ✅ DO: Include all required fields
    const payload = {
      ...form,
      eventId,
      createdAt: new Date().toISOString(),
      updatedAt: new Date().toISOString(),
    };
    await fetch('/api/proxy/ticket-types', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(payload),
    });
    ```

- **ID field handling for create operations**
  - For POST (create) operations, do not include the 'id' field in the payload (or set it to null if required by the backend).
  - Only include 'id' for update (PUT/PATCH) operations.
  - This matches backend expectations and avoids sending unnecessary or misleading ids during creation.
  - Example for ticket type creation:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
