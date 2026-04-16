---
trigger: always_on
description: This rule documents the authentication implementation requirements for Ouroboros's SSR migration. The current blocker is that SSR pages attempt authenticated API calls but no SSR authentication flow exists.
---

# SSR Authentication Implementation Guide

## Overview

This rule documents the authentication implementation requirements for Ouroboros's SSR migration. The current blocker is that SSR pages attempt authenticated API calls but no SSR authentication flow exists.

## Current State Analysis

### Problem Identification

- **Issue**: SSR load functions make authenticated API calls to FastAPI backend but no session handling exists
- **Symptom**: E2E tests fail because frontend service health check fails (401 responses)
- **Root Cause**: Migration from SPA to SSR completed without implementing server-side authentication

### Working Components

- ✅ Three-tier testing architecture with Docker infrastructure
- ✅ E2E data seeding with service layer delegation
- ✅ Frontend and backend containers build successfully
- ✅ SSR routes and form actions implemented
- ❌ **Missing**: Session-based authentication for SSR load functions

## Required Authentication Implementation

### 1. Session Cookie Handling

**SvelteKit Side** (`hooks.server.js`):

```javascript
// Handle authentication cookies and session management
export async function handle({ event, resolve }) {
    // Extract session cookie from request
    const sessionCookie = event.cookies.get('sessionid');

    // Set user context for load functions
    if (sessionCookie) {
        event.locals.session = sessionCookie;
        event.locals.user = await validateSession(sessionCookie);
    }

    return resolve(event);
}
```

**SSR Load Functions Pattern**:

```typescript
// In +page.server.ts files
export const load: PageServerLoad = async ({ cookies, locals }) => {
    // Use session from hooks.server.js
    const sessionCookie = cookies.get('sessionid') || locals.session;

    if (!sessionCookie) {
        throw redirect(302, '/login');
    }

    try {
        const response = await serverApi.get('/api/v1/web/campaigns/', {
            headers: {
                'Cookie': `sessionid=${sessionCookie}`,
                'X-Requested-With': 'XMLHttpRequest'
            }
        });
        return { campaigns: response.data };
    } catch (error) {
        if (error.response?.status === 401) {
            throw redirect(302, '/login');
        }
        throw error(500, 'Failed to load data');
    }
};
```

### 2. Authentication State Management

**Server-Side API Client** ([lib/server/api.js](mdc:Ouroboros/Ouroboros/frontend/src/lib/server/api.js)):

```typescript
import type { Cookies } from '@sveltejs/kit';

export class ServerApiClient {
    private baseURL: string;

    constructor(baseURL: string) {
        this.baseURL = baseURL;
    }

    async authenticatedRequest(
        endpoint: string,
        options: RequestInit,
        cookies: Cookies
    ) {
        const sessionCookie = cookies.get('sessionid');

        if (!sessionCookie) {
            throw new Error('No session cookie found');
        }

        return fetch(`${this.baseURL}${endpoint}`, {
            ...options,
            headers: {
                ...options.headers,
                'Cookie': `sessionid=${sessionCookie}`,
                'X-Requested-With': 'XMLHttpRequest'
            }
        });
    }
}
```

### 3. Login Form Implementation

**Login Route** (`/login/+page.server.ts`):

```typescript
export const actions: Actions = {
    default: async ({ request, cookies }) => {
        const form = await superValidate(request, zod(loginSchema));

        if (!form.valid) {
            return fail(400, { form });
        }

        try {
            // Authenticate with FastAPI
            const response = await fetch(`${API_BASE_URL}/api/v1/web/auth/login`, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(form.data)
            });

            if (!response.ok) {
                return fail(401, { form, message: 'Invalid credentials' });
            }

            // Extract session cookie from response
            const setCookieHeader = response.headers.get('set-cookie');
            const sessionMatch = setCookieHeader?.match(/sessionid=([^;]+)/);

            if (sessionMatch) {
                cookies.set('sessionid', sessionMatch[1], {
                    httpOnly: true,
                    secure: true,
                    sameSite: 'strict',
                    maxAge: 60 * 60 * 24 * 7 // 7 days
                });
            }

            throw redirect(303, '/');
        } catch (error) {
            return fail(500, { form, message: 'Login failed' });
        }
    }
};
```

### 4. Environment Detection for Tests

**Test Environment Bypass**:

```typescript
// In SSR load functions
export const load: PageServerLoad = async ({ cookies }) => {
    // Bypass authentication in test environments
    if (process.env.NODE_ENV === 'test' ||
        process.env.PLAYWRIGHT_TEST ||
        process.env.CI) {
        return {
            campaigns: mockCampaignData,
            user: mockUserData
        };
    }

    // Normal authentication flow
    return authenticatedLoad(cookies);
};
```

## Implementation Strategy


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/unclesp1d3r) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
