---
trigger: always_on
description: Use these rules as defaults, not as a reason to add ceremonial folders or wrapper layers.
---

Use these rules as defaults, not as a reason to add ceremonial folders or wrapper layers.

## Core Principles

- Organize code around product capabilities, not framework vocabulary.
- Keep related UI, state, rules, and data access close until a real boundary justifies moving
  them apart.
- Dependencies point from composition and UI toward stable rules and narrow capabilities.
- Protect rendering code from business, state-management, and infrastructure complexity.
- Keep one source of truth and derive everything else.
- Apply KISS, YAGNI, and DRY together. Remove duplicated knowledge, not merely similar syntax.
- Prefer explicit, readable flow over clever abstractions and hidden behavior.

## Screaming Architecture

The repository structure and public APIs should reveal what the product does.

Prefer:

```text
features/
  checkout/
  search/
  account-security/
```

Avoid making the application read primarily as:

```text
components/
hooks/
services/
stores/
utils/
```

Technical folders are useful inside a capability, where their owner is clear. Generic top-level
folders easily become dependency magnets with unclear ownership.

Names should use product language. Prefer `useCheckoutSummary`, `reserveStock`, and
`AccountSecurityPanel` over `useData`, `processItems`, and `GenericPanel`.

## Suggested Structure

Start with the smallest structure that makes ownership obvious:

```text
src/
  app/                 startup, providers, router, global composition
  pages/               route-level composition
  features/
    <capability>/
      index.ts         optional public API
      ui/              optional rendering components
      model/           optional state, view models, decisions
      api/             optional external data access
      lib/             optional feature-local pure helpers
  domains/             optional shared product rules and types
  shared/
    ui/                domain-free visual primitives
    api/               generic transport/query infrastructure
    lib/               genuinely generic pure helpers
```

Folders are created when they contain a real responsibility. A small feature may be one cohesive
file. Do not create empty layers in anticipation of future complexity.

## Dependency Direction

- `app` installs providers, constructs dependencies, and composes the application.
- `pages` compose capabilities for a route. They do not own business rules or data protocols.
- A feature owns one user-recognizable capability end to end.
- Feature UI consumes its own model/view-model API, not raw infrastructure.
- Shared domain code contains reusable product rules and stays independent of React and I/O.
- `shared` contains only domain-free code. Product-specific code is not shared merely because
  two files use it.
- Avoid feature-to-feature imports. Compose features in a page, promote truly shared rules to a
  domain module, or introduce a named workflow when coordination is the actual responsibility.
- Cyclic imports are an architecture problem, not something to solve with a tooling workaround.

For a simple feature, direct `ui -> model -> api` dependencies are sufficient. Introduce ports,
facades, dependency injection, or workflows only when they hide real complexity, enable
important tests, or separate unstable infrastructure.

## Make Composition Read Like The Product

Pages and other composition boundaries should use capability-level APIs.

Prefer:

```tsx
<CheckoutSummary />
<PlaceOrderButton />
```

Over:

```tsx
<Card>
  <Select options={paymentOptions} onChange={handlePaymentChange} />
  <Button onClick={handleSubmit}>Submit</Button>
</Card>
```

The second version makes the page understand checkout behavior and low-level UI configuration.
That knowledge belongs to the checkout capability.

This does not mean wrapping every native element or design-system primitive. Semantic HTML and
visual primitives are correct inside feature UI. Create a capability component when it hides
product behavior or gives composition code a clearer product-level API.

Avoid "raw components" whose consumers must know internal options, state transitions, query
shapes, or protocol details. Avoid generic configuration-driven components that combine
unrelated product modes behind dozens of props.

## UI Boundary

- Components render data and translate DOM events into named user intents.
- Keep business decisions, data mapping, persistence, protocol handling, and multi-step async
  flows outside rendering components.
- UI receives render-ready values. It should not reconstruct domain meaning from raw DTOs.
- Prefer intent props and commands such as `onApprove`, `renameProject`, or `submitOrder` over
  generic `onChange`, `setState`, or `patch` APIs at capability boundaries.
- Keep ephemeral visual state local: focus, hover, open/closed, and uncommitted input usually
  belong in the component.
- Split components by responsibility and API clarity, not by arbitrary line limits.
- Prefer slots and composition over components with many layout modes and boolean props.
- Use semantic HTML and preserve accessibility behavior.

A view-model hook is useful when it protects UI from state shape, async coordination, or business

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [k1tbyte/Wand-Enhancer](https://github.com/k1tbyte/Wand-Enhancer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
