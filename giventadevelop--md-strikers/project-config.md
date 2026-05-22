---
trigger: always_on
description: Playwright testing setup, middleware fixes, and error handling patterns for public and admin tests
---


# Playwright Testing Setup and Middleware Fixes

## **Overview**
This rule documents the fixes applied to enable Playwright automated testing for both public pages and admin pages, including middleware configuration changes, error handling patterns, and authentication workarounds.

## **Problem Solved**
- **Public Page 401 Errors**: Playwright tests failing with 401 Unauthorized for public pages
- **Admin Test Authentication**: Admin tests failing due to Clerk authentication and admin role checks
- **Middleware Interference**: Clerk middleware blocking Playwright requests without session cookies
- **Strict Error Detection**: Tests failing on false positives (401/403 text in HTML/JS, not actual errors)

---

## **Core Pattern: Middleware Wrapper for Playwright Compatibility**

### **Custom Middleware Wrapper in `src/middleware.ts`**

**CRITICAL**: We wrap Clerk's `authMiddleware` with a custom middleware function that intercepts 401/redirect responses for public routes. This allows Playwright tests to work while maintaining `auth()` functionality.

**Pattern**:
```typescript
// Create Clerk middleware (still called for all routes)
const clerkMiddleware = authMiddleware({
  publicRoutes: [
    '/',              // Homepage - needs auth() for admin lookup
    '/events(.*)',    // Public pages
    '/api/proxy(.*)', // API proxy routes
    // ... other public routes
  ],
  ignoredRoutes: [
    '/api/proxy/(.*)',  // Completely bypass Clerk for API proxy (mobile browser compatibility)
    '/api/webhooks/(.*)',
    // ... other ignored routes
  ],
});

// Custom wrapper that intercepts 401/redirects for public routes
export default async function middleware(req: NextRequest) {
  const pathname = req.nextUrl.pathname;
  const isPublic = isPublicRoute(pathname);

  // Always call Clerk middleware (even for public routes) so auth() works in layout.tsx
  let response = clerkMiddleware(req);
  if (response instanceof Promise) {
    response = await response;
  }

  // CRITICAL: If Clerk returned 401 or redirected to sign-in for a public route, override it
  if (isPublic && response instanceof NextResponse) {
    const location = response.headers.get('location');
    const isRedirectToSignIn = location && (location.includes('/sign-in') || location.includes('sign-in'));
    const isUnauthorized = response.status === 401 || response.status === 307 || response.status === 308;

    if (isUnauthorized || isRedirectToSignIn) {
      // Override to 200 - allow access for Playwright tests
      const publicResponse = NextResponse.next();
      publicResponse.headers.set('x-pathname', pathname);
      // Copy headers from Clerk's response (except location)
      response.headers.forEach((value, key) => {
        if ((key.startsWith('x-') || key === 'set-cookie') && key !== 'location') {
          publicResponse.headers.set(key, value);
        }
      });
      return publicResponse;
    }
  }

  return response;
}
```

### **Key Requirements**:
1. ✅ **Call `clerkMiddleware` for all routes** - Ensures `auth()` works in `layout.tsx`
2. ✅ **Intercept 401/redirects for public routes** - Allows Playwright tests without session cookies
3. ✅ **Preserve Clerk headers** - Copy `x-*` and `set-cookie` headers from Clerk's response
4. ✅ **Don't break Clerk detection** - Clerk detects `authMiddleware()` by checking file contents, not export

---

## **Public Routes Configuration**

### **Routes That Call `auth()` MUST Be in `publicRoutes`**

**CRITICAL**: Routes that call `auth()` or `currentUser()` in server components **MUST** be in `publicRoutes` (NOT `ignoredRoutes`) so Clerk middleware runs.

**Example**:
```typescript
publicRoutes: [
  '/',              // ✅ Homepage - layout.tsx calls auth() for admin lookup
  '/polls(.*)',     // ✅ Poll pages call auth() to check user participation
  '/pricing(.*)',   // ✅ Pricing page calls auth() to check subscription status
  '/events(.*)',    // ✅ Public pages (may call auth() for user-specific content)
  '/api/proxy(.*)', // ✅ API proxy routes (public access, backend handles auth)
],
```

### **Routes That DON'T Call `auth()` Can Be in `ignoredRoutes`**

**Example**:
```typescript
ignoredRoutes: [
  '/api/proxy/(.*)',  // ✅ API routes handle their own auth (JWT)
  '/api/webhooks/(.*)', // ✅ Webhook routes use service JWT, not Clerk
  // NOTE: Public page routes like /events, /gallery are NOT in ignoredRoutes
  // because layout.tsx calls auth() to check admin status for header menu visibility
],
```

---

## **Relaxed Error Detection Pattern**

### **Problem: False Positives from 401/403 Text in HTML/JS**

Playwright tests were failing when they found "401" or "403" text in HTML comments, JavaScript code, or hidden elements, even though the page loaded successfully.

### **Solution: Only Check Visible Error Elements**

**Pattern**:
```javascript
// ❌ DON'T: Check for any "401" or "403" text in page content
const pageContent = await page.content();
if (pageContent.includes('401') || pageContent.includes('403')) {
  throw new Error('401/403 error detected');
}

// ✅ DO: Only check for visible error elements with specific selectors
const errorSelectors = [
  '[role="alert"]',
  '[class*="error"][class*="message"]',

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
