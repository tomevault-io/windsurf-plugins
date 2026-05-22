---
trigger: always_on
description: Clerk authentication middleware configuration, admin role lookup, and satellite/primary sign-in and sign-out patterns for Next.js App Router
---


# Clerk Authentication and Admin Role Lookup Pattern

## **Overview**
This rule defines the correct pattern for configuring Clerk middleware to enable `auth()` calls in server components, particularly for admin role lookup in the root layout. It explains when routes should be in `ignoredRoutes` vs `publicRoutes`, how to properly check admin status after user login, and **why the admin check must run on all routes (including public pages)** so the Admin menu appears in the Header on every page—not only on `/admin` and sub-pages.

## **Problem Solved**
- **Admin Menu Not Appearing**: Ensures admin role lookup works correctly after user login
- **Admin Menu on Public Pages**: Ensures the Admin menu appears when a logged-in admin visits **any** page (homepage, events, gallery, pricing, etc.), not only `/admin` and sub-pages
- **Clerk auth() Errors**: Prevents "Clerk can't detect usage of authMiddleware()" errors
- **Next.js 15+ Compatibility**: Properly handles async `headers()` calls
- **Route Configuration**: Clarifies when routes need Clerk middleware vs when they can be ignored

---

## **Core Pattern: Middleware Configuration**

### **Routes That Call `auth()` MUST NOT Be in `ignoredRoutes`**

**CRITICAL**: Any route that calls `auth()` or `currentUser()` in server components **MUST** have Clerk middleware running. This means:

- ✅ **DO**: Keep route in `publicRoutes` (if it should be accessible without auth)
- ❌ **DON'T**: Put route in `ignoredRoutes` (this bypasses Clerk middleware completely)

**Example - Homepage Admin Lookup**:
```typescript
// ✅ DO: Homepage in publicRoutes but NOT in ignoredRoutes
export default authMiddleware({
  publicRoutes: [
    '/',  // Homepage - accessible without auth, but middleware runs for auth() calls
    // ... other public routes
  ],

  ignoredRoutes: [
    // ❌ DON'T: '/',  // This would break auth() calls in layout.tsx
    '/events(.*)',  // OK - these pages don't call auth()
    '/gallery(.*)',  // OK - these pages don't call auth()
    // ... other routes that don't need auth()
  ],
});
```

---

## **When to Use `ignoredRoutes` vs `publicRoutes`**

### **Use `ignoredRoutes` When:**
- ✅ Route **does NOT** call `auth()` or `currentUser()` in server components
- ✅ Route is a static page with no authentication needs
- ✅ Route is an API endpoint that handles its own authentication
- ✅ Route is for automated testing (Playwright, etc.) that needs no Clerk middleware

**Examples**:
```typescript
ignoredRoutes: [
  '/events(.*)',      // Static public pages, no auth() calls
  '/gallery(.*)',     // Static public pages, no auth() calls
  '/api/proxy/(.*)',  // API routes handle their own auth
  '/api/webhooks/(.*)', // Webhook routes use service JWT, not Clerk
],
```

### **Use `publicRoutes` (NOT `ignoredRoutes`) When:**
- ✅ Route **DOES** call `auth()` or `currentUser()` in server components
- ✅ Route needs to check user authentication status (even if login not required)
- ✅ Route needs to look up user profile or admin status
- ✅ Route should be accessible without login, but needs Clerk middleware for `auth()` calls

**Examples**:
```typescript
publicRoutes: [
  '/',              // Homepage - layout.tsx calls auth() for admin lookup
  '/polls(.*)',     // Poll pages call auth() to check user participation
  '/pricing(.*)',   // Pricing page calls auth() to check subscription status
  '/profile(.*)',   // Profile pages need auth() to get user data
],
```

---

## **Admin Role Lookup Pattern in Root Layout**

### **Server-Side Admin Check in `src/app/layout.tsx`**

**Pattern**:
```typescript
export default async function RootLayout({ children }: { children: React.ReactNode }) {
  // CRITICAL: Next.js 15+ requires headers() to be awaited first
  const headersList = await headers();
  const hostname = headersList.get('host') || '';

  // ... other setup code ...

  // Determine tenant-scoped admin flag on the server
  let isTenantAdmin = false;
  try {
    // CRITICAL: Call auth() AFTER headers() is awaited
    const authResult = await auth();
    const userId = authResult?.userId || null;

    if (userId) {
      const tenantId = getTenantId();

      // Fetch user profile to check admin role
      const url = `${baseUrl}/api/proxy/user-profiles?userId.equals=${userId}&tenantId.equals=${tenantId}&size=1`;
      const resp = await fetch(url, { cache: 'no-store' });

      if (resp.ok) {
        const data = await resp.json();
        const profile = Array.isArray(data) ? data[0] : data;
        isTenantAdmin = profile?.userRole === 'ADMIN';
      }
    }
  } catch (error) {
    // Fail closed (no admin) on error
    console.error('[Layout] Error determining admin status:', error);
    isTenantAdmin = false;
  }

  return (
    <ClerkProvider {...clerkProps}>
      {/* ... */}
      <Header isTenantAdmin={isTenantAdmin} />
      {/* ... */}
    </ClerkProvider>
  );
}
```

### **Admin Check on ALL Routes (Including Public Pages)**

**CRITICAL**: Run the admin check on **every route** where the root layout renders (i.e., whenever `pathname` is present). Do **not** limit the admin lookup to `/admin` or protected routes only.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
