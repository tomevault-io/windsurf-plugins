---
trigger: always_on
description: >
---


# Kanvas Core JS SDK

Reference skill for building applications with `@kanvas/core`. The SDK is **fully modular** —
each capability is a separate subpath import. Always import only what's needed.

## Package install

```bash
npm install @kanvas/core
```

---

## Two client types — pick the right one

This is the most common source of bugs. The choice depends on **where** the code runs.

| Situation | Use |
|---|---|
| Browser / React client component | `createClient` |
| Next.js Server Component, API Route, middleware, Deno server | `createAdminClient` |

**Never put `adminKey` in browser-side code.** It will leak.

### `createClient` (browser-safe)

```typescript
import { createClient } from "@kanvas/core/app";

const client = createClient({
  appKey: process.env.NEXT_PUBLIC_KANVAS_API_KEY!,
  baseUrl: process.env.NEXT_PUBLIC_KANVAS_URL!,
  headers: {
    // Dynamic — re-evaluated on every request
    Authorization: () => localStorage.getItem("token") ?? "",
  },
});
```

### `createAdminClient` (server-only)

```typescript
import { createAdminClient } from "@kanvas/core/app";

const adminClient = createAdminClient({
  appKey: process.env.KANVAS_API_KEY!,
  adminKey: process.env.KANVAS_ADMIN_KEY!,
  baseUrl: process.env.KANVAS_URL!,
  ssrMode: true, // set true in Next.js App Router / SSR contexts
});
```

---

## Module map — imports at a glance

```
@kanvas/core/app          → createClient, createAdminClient
@kanvas/core/auth         → createAuth
@kanvas/core/settings     → createSettings
@kanvas/core/file-system  → createFileSystem
@kanvas/core/receiver     → createReceiver
@kanvas/core/commerce     → createCart, createOrder
@kanvas/core/locations    → createLocations
@kanvas/core/types        → Type-only imports (Client, AuthLoginResponse, CartItem, Order…)
```

All modules follow the same pattern: `createXxx(client)` returns an instance with methods.

---

## Common usage patterns

### Auth

```typescript
import { createAuth } from "@kanvas/core/auth";

const auth = createAuth(client);

// Email/password
const session = await auth.login({ email, password });

// Token refresh (call before expiry)
const renewed = await auth.refreshToken(session.refresh_token);

// OAuth / social
const socialSession = await auth.socialLogin({ token: oauthToken, provider: "google" });
```

### Settings

```typescript
import { createSettings } from "@kanvas/core/settings";

const settings = createSettings(adminClient); // usually admin
const appSettings = await settings.appSettings();
```

### File system

```typescript
import { createFileSystem } from "@kanvas/core/file-system";

const fs = createFileSystem(client);

const file = new File([blob], "photo.jpg", { type: "image/jpeg" });
const uploaded = await fs.uploadFile(file);

// Profile photo shortcut
await fs.updateUserPhotoProfile(file, userId);
```

### Commerce — cart & orders

```typescript
import { createCart, createOrder } from "@kanvas/core/commerce";

const cart   = createCart(client);
const orders = createOrder(client);

await cart.addToCart([{ variant_id: "sku-123", quantity: 2 }]);

const result = await orders.createOrderFromCart({
  cartId: "default",
  customer: { email: "buyer@example.com", phone: "+18095551234" },
});
```

### Locations

```typescript
import { createLocations } from "@kanvas/core/locations";

const locations = createLocations(client);

const countries = await locations.getAllCountries({ first: 10 });
const states    = await locations.getStatesByCountry(countryId);
```

---

## Next.js integration

### App Router — Server Components

Create a shared factory so the client isn't re-instantiated per request:

```typescript
// lib/kanvas.ts
import { createAdminClient } from "@kanvas/core/app";
import { createSettings }    from "@kanvas/core/settings";

export function getKanvasClient() {
  const client   = createAdminClient({ /* env vars */ ssrMode: true });
  const settings = createSettings(client);
  return { client, settings };
}

// app/page.tsx
import { getKanvasClient } from "@/lib/kanvas";

export default async function Page() {
  const { settings } = getKanvasClient();
  const data = await settings.appSettings();
  return <main>{/* ... */}</main>;
}
```

### Pages Router — API Routes

```typescript
// pages/api/settings.ts
import type { NextApiRequest, NextApiResponse } from "next";
import { createAdminClient } from "@kanvas/core/app";
import { createSettings }    from "@kanvas/core/settings";

export default async function handler(req: NextApiRequest, res: NextApiResponse) {
  const client   = createAdminClient({ /* env vars */ });
  const settings = createSettings(client);
  res.json(await settings.appSettings());
}
```

---

## Building custom modules

When the built-in modules don't cover a use case, extend the client directly with GraphQL:

```typescript
import { Client } from "@kanvas/core/app";
import { gql }    from "@apollo/client";

class Leads {
  #client: Client;
  constructor(client: Client) { this.#client = client; }

  async create(input: LeadInput) {
    const { data } = await this.#client.mutate({
      mutation: gql`mutation CreateLead($input: LeadInput!) {
        createLead(input: $input) { id uuid title }
      }`,
      variables: { input },
    });
    return data.createLead;
  }

  async list(opts: { first?: number; page?: number } = {}) {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bakaphp/kanvas-core-js](https://github.com/bakaphp/kanvas-core-js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
