---
trigger: always_on
description: MFE boundary review: MFE boundary rules. Load when this topic is in scope; part of mfe-skills.
---


# MFE boundary rules

**Version**: 1.3 | **Skill**: reviewing-mfe-boundaries | **Source**: *Building Micro-Frontends* (O'Reilly). Canvas facilitation: separate micro-frontend-canvas skill.

Principle-level definitions, violation signals, and canonical code patterns for all eight boundary rules. Tool-agnostic — applies regardless of composition mechanism, framework, or toolchain.

For framework-specific code patterns (Module Federation v2, Angular/Native Federation, Single SPA): load `references/rules-toolchain.md`.

Each rule entry contains: the canonical definition, violation signals, and code-checkable patterns.

---

## Rule 1 — Represents a business subdomain, not a component

A micro-frontend is the technical representation of a business subdomain. It is not a reusable UI component.

**The distinction**: a component addresses a technical challenge through abstraction and reusability, and its API is frequently coupled with its container. A micro-frontend completely owns a business domain; it is context-aware, not designed for reuse across domains.

**How to identify a domain**: base boundaries on user journey steps (checkout, search, profile, authentication) or clear business capabilities — not on technical frameworks, component types, or UI layout regions.

**The rule of thumb**: if you can reuse it everywhere, it is probably a component. If it represents a domain and can live on its own, it is a micro-frontend.

**Violation signals**:
- A micro-frontend named after a UI element (header, sidebar, card) rather than a business capability
- A micro-frontend that is reused across multiple unrelated domains
- Boundaries drawn along technical layers (data layer, presentation layer) rather than domain lines

**Code-checkable patterns**:
```jsx
// ✗ Named after a UI element — not a domain
<HeaderMicrofrontend />
<SidebarMicrofrontend />
<CardMicrofrontend />

// ✓ Named after a business capability
<CheckoutMicrofrontend />
<CatalogMicrofrontend />
<AuthenticationMicrofrontend />
```

If a micro-frontend is imported and used in many unrelated domains, it is likely a component that should move to a shared design system package — not a runtime-integrated micro-frontend.

---


---

## Rule 2 — Exposes a minimal API surface to its container

Streamline the API surface to the essential minimum required for the micro-frontend to understand the user's context. Typically this means little more than a session token and a context identifier such as a product ID.

**The Canvas threshold**: fewer than 5 props exposed to the container.

**Why this matters**: when you expose too many properties, the container starts owning the context instead of the micro-frontend. The micro-frontend becomes a dumb rendering layer, and the container accumulates domain knowledge it should not have. This produces accidental complexity and forces constant coordination across teams.

**Violation signals**:
- More than 5 props passed from container to micro-frontend
- Props that represent entire domain objects (a full User, a full Order, a full Cart)
- The container fetching data on behalf of the micro-frontend and passing it in
- Teams coordinating constantly because a prop change in one triggers changes in the other

**Code-checkable patterns**:
```jsx
// ✓ Minimal context — identifier + session access
<CheckoutMicrofrontend userId={userId} cartId={cartId} />

// ✗ Container owns context — too many props, domain objects passed
<CheckoutMicrofrontend
  user={user}                    // full domain object
  cart={cart}                    // full domain object
  shippingOptions={shippingOptions}
  paymentMethods={paymentMethods}
  discountCodes={discountCodes}
  onComplete={handleComplete}
  onError={handleError}
  theme={theme}
/>
```

**Secondary signal**: if the container is making API calls to fetch data before passing it to the micro-frontend, the micro-frontend is not fetching its own data — the container owns context it should not.


---

## Rule 3 — Hides implementation details behind an API contract

Define the API contract upfront between producer and consumer teams. Internal implementation details — frameworks, data fetching strategies, database schemas, code structure — stay hidden behind it.

**The API-first principle**: the contract is the binding agreement. Both teams can work in parallel, focused on their side of the contract. Either team can change their internals freely without affecting the other, as long as the contract is respected.

**Strong encapsulation is required** to avoid domain leaks into other parts of the application. When a micro-frontend exposes its internals — through direct imports, shared modules, or coupled data structures — changes inside cascade outside and independent deployment breaks down.

**Violation signals**:
- Direct imports from one micro-frontend's source into another
- Shared internal modules or utilities across micro-frontend boundaries
- Contract changes that require simultaneous updates across multiple teams
- A micro-frontend exposing more than what consumers strictly need

**Code-checkable patterns**:
```javascript
// ✗ CRITICAL — cross-boundary import, bypasses the API contract
import { UserStore } from '@org/auth-mfe/store'

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lucamezzalira/mfe-skills](https://github.com/lucamezzalira/mfe-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
