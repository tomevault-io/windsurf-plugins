---
trigger: always_on
description: MFE boundary review: Remediation patterns. Load when this topic is in scope; part of mfe-skills.
---


# Remediation patterns

**Version**: 1.2 | **Skill**: reviewing-mfe-boundaries | **Source**: *Building Micro-Frontends* (O'Reilly)

Each section maps to one of the eight boundary rules.

---

## Rule 1 — Reframing a component as a micro-frontend (or vice versa)

If a unit named or behaving as a UI component is being integrated as a micro-frontend, the fix is to decide which it actually is:

**If it is truly a shared UI primitive** (button, form field, layout wrapper): move it to an npm design system package. Remove the runtime integration overhead. It should be a versioned package dependency, not a deployed micro-frontend.

**If it is truly a business capability** masquerading as a component: rename it to reflect the domain, assign a single team to own it, and ensure it fetches its own data rather than receiving domain objects as props.

**If the boundary is ambiguous**: run the boundary health check from `mfe-core-concepts/references/rules.md`. If it fails two or more tests, it is in the wrong category.

---

## Rule 2 — Reducing an oversized API surface

**Before** — container owns context:
```jsx
<CheckoutMicrofrontend
  user={user}
  cart={cart}
  shippingOptions={shippingOptions}
  paymentMethods={paymentMethods}
  discountCodes={discountCodes}
/>
```

**After — step 1**: identify which props are domain data the micro-frontend should fetch itself.

In this example: `shippingOptions`, `paymentMethods`, and `discountCodes` are checkout domain data. The checkout micro-frontend should retrieve them from its own backend (BFF or API layer), not receive them from the container.

**After — step 2**: reduce to the minimum context identifiers:
```jsx
// The checkout MFE fetches its own shipping options, payment methods, discounts
<CheckoutMicrofrontend userId={userId} cartId={cartId} />
```

**After — step 3**: inside the checkout micro-frontend, fetch domain data directly:
```javascript
// Inside checkout-mfe — fetches its own data
function CheckoutApp({ userId, cartId }) {
  const cart = useCartData(cartId)           // own API call
  const shippingOptions = useShipping()      // own API call
  const paymentMethods = usePaymentMethods() // own API call
}
```

**When the container genuinely needs to pass more context**: if the use case truly requires more than 5 props, consider whether the boundary is in the right place. Two micro-frontends that share substantial context may belong to the same domain and should be one micro-frontend owned by one team.

---

## Rule 3 — Removing a cross-boundary import

**Before** — direct import from another MFE's internals:
```javascript
// In checkout-mfe — VIOLATION
import { UserStore } from '@org/auth-mfe/store'
import { getAuthToken } from '@org/auth-mfe/auth'
```

**Fix option A — auth transparency patterns** (choose the pattern that matches the shell architecture):

```javascript
// Pattern 1: MFE reads token directly from storage — fully independent
const authToken = sessionStorage.getItem('auth_token')
// or from a cookie set by the auth MFE
const authToken = document.cookie.match(/auth_token=([^;]+)/)?.[1]
// MFE uses this token to call its own BFF directly
```

```javascript
// Pattern 2: Shell provides a fetch wrapper that injects auth headers automatically
// The MFE makes plain fetch() calls with no knowledge of auth at all.
// The shell installs the wrapper at startup — token refresh is handled transparently.

// shell/src/fetchWrapper.js
const originalFetch = window.fetch
window.fetch = async (url, options = {}) => {
  const token = await getValidToken()  // shell handles refresh logic
  return originalFetch(url, {
    ...options,
    headers: { ...options.headers, Authorization: `Bearer ${token}` }
  })
}

// checkout-mfe — calls its BFF with no auth concern
const cart = await fetch('/api/cart').then(r => r.json())
```

Pattern 2 handles credential plumbing transparently. However, in long-running applications MFEs may hold reactive state that derives from auth — a logged-in flag, a user role, a display name — and this state must update when the shell refreshes or invalidates the token. The fetch wrapper alone does not propagate these changes.

Combine Pattern 2 with auth state events on the shared event bus. The shell emits whenever auth state changes; each MFE that holds derived auth state subscribes and reacts:

```javascript
// shell — emits on every token refresh and on logout
eventBus.emit('auth:tokenRefreshed', { userId, roles, expiresAt })
eventBus.emit('auth:sessionEnded', {})

// checkout-mfe — subscribes to auth state changes
eventBus.on('auth:tokenRefreshed', ({ userId, roles }) => {
  setCurrentUser({ userId, roles })   // update local reactive state
  // fetch wrapper already updated the credential — just sync identity state
})
eventBus.on('auth:sessionEnded', () => {
  clearLocalState()
  showSessionExpiredMessage()
})
```

```typescript
// Pattern 3: Shell passes a single auth token via InjectionToken (Angular) or prop (React)
// The MFE uses it to bootstrap one BFF call; the BFF owns session from there.

// Angular — shell provides token once at startup
providers: [{ provide: AUTH_TOKEN, useValue: currentToken }]

// checkout.component.ts (remote)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lucamezzalira/mfe-skills](https://github.com/lucamezzalira/mfe-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
