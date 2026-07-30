---
trigger: always_on
description: MFE boundary review: MFE boundary rules — toolchain patterns. Load when this topic is in scope; part of mfe-skills.
---


# MFE boundary rules — toolchain patterns

**Version**: 1.3 | **Skill**: reviewing-mfe-boundaries | **Source**: *Building Micro-Frontends* (O'Reilly)

Toolchain-specific code patterns for each boundary rule. Load this file when the user is working with a specific framework or when a rule violation needs framework-specific guidance. For principle definitions and violation signals, see `rules-core.md`.

## URL routing — shell first segment, MFE below (Rule 7)

Principle: shell loads first segment from a dynamic manifest; MFE hardcodes deeper paths. Navigation API is flexible. Shell handles `shell:*` platform events only. See `routing-ownership.md`.

```tsx
// React Router — shell
<Route path="/catalog/*" element={<RemoteMount scope="catalog_mfe" />} />

// React Router — catalog remote (basename matches shell prefix)
<BrowserRouter basename="/catalog">
  <Routes>
    <Route path="/" element={<ProductList />} />
    <Route path="/product/:productId" element={<ProductDetail />} />
  </Routes>
</BrowserRouter>
```

```typescript
// Angular — shell first segment
{ path: 'catalog', loadChildren: () => loadRemoteModule('catalog', './routes') }
// catalog/routes.ts (catalog team): { path: 'product/:id', component: ProductDetailComponent }
```

```javascript
// Single SPA — shell first segment via activeWhen prefix
registerApplication({
  name: 'catalog',
  app: () => import('catalog/main'),
  activeWhen: (location) => location.pathname.startsWith('/catalog'),
})
// catalog/main mounts its own router for /catalog/product/:id
```

---

## Rule 2 — Exposes a minimal API surface to its container

**Toolchain patterns — same rule, different surfaces:**
```typescript
// Native Federation (Angular) — shell passes props via loadRemoteModule input binding
// ✓ Minimal context — route param only; remote fetches its own data
// app.routes.ts (shell)
{
  path: 'checkout/:cartId',
  loadComponent: () =>
    loadRemoteModule('checkout', './CheckoutComponent')
      .then(m => m.CheckoutComponent)
  // cartId passed via route param — remote reads ActivatedRoute itself
}

// ✗ Shell resolving full domain objects before loading the remote
{
  path: 'checkout',
  resolve: { cart: CartResolver, user: UserResolver, shipping: ShippingResolver },
  loadComponent: () =>
    loadRemoteModule('checkout', './CheckoutComponent')
      .then(m => m.CheckoutComponent)
  // resolved data injected via route — container now owns context
}
```

```javascript
// Module Federation v2 — props passed via runtime bridge or input binding
// ✓ Minimal: remote receives only IDs via shared routing context
import { init, loadRemote } from '@module-federation/runtime'

await init({
  name: 'shell',
  remotes: [{ name: 'checkout', entry: window.__remotes__.checkout }],
})
const { CheckoutApp } = await loadRemote('checkout/CheckoutApp')
// cartId and userId passed via URL params — remote reads them independently

// ✗ Shell passing full resolved objects into the remote via bridge
bridge.provide('checkoutContext', { user, cart, shippingOptions, paymentMethods })
// remote now depends on the shell's data shape
```

```javascript
// Single SPA — props passed via the application's customProps
// ✓ Minimal context
registerApplication({
  name: 'checkout',
  app: () => import('checkout/main'),
  activeWhen: '/checkout',
  customProps: { userId, cartId }  // identifiers only
})

// ✗ Full domain objects in customProps
registerApplication({
  name: 'checkout',
  app: () => import('checkout/main'),
  activeWhen: '/checkout',
  customProps: { user, cart, shippingOptions, paymentMethods, theme }
  // 5+ props including domain objects — violation
})
```

---


## Rule 3 — Hides implementation details behind an API contract

**Toolchain patterns — same rule, different surfaces:**
```typescript
// Native Federation (Angular) — cross-boundary import via exposed Angular service
// ✗ CRITICAL — importing another MFE's exposed service directly
import { AuthService } from 'auth/AuthService'  // resolves to auth remote at runtime

@Component({ ... })
export class CheckoutComponent {
  constructor(private auth: AuthService) {}  // runtime coupling to auth MFE internals
}

// ✓ Auth token read from shared storage — no import of another MFE's service
@Component({ ... })
export class CheckoutComponent implements OnInit {
  private authToken = inject(AUTH_TOKEN)  // token provided by shell via InjectionToken
  // or: read directly from sessionStorage / cookie
}
```

```javascript
// Module Federation v2 — cross-boundary import via @module-federation/runtime
// ✗ Eagerly loading another MFE's internal module
import { userStore } from '@org/auth-mfe/store'
// Even in MF v2 this creates a runtime dependency on auth-mfe's deployment

// ✗ Using loadRemote to import another MFE's internals
import { loadRemote } from '@module-federation/runtime'
const { UserStore } = await loadRemote('auth/store')  // still a boundary violation
// loadRemote is for your own remotes or a shell loading its children — not peer-to-peer

// ✓ Checkout MFE exposes only what it owns; reads auth via contract (cookie/storage)
const token = document.cookie.match(/auth_token=([^;]+)/)?.[1]
```

```javascript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lucamezzalira/mfe-skills](https://github.com/lucamezzalira/mfe-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
