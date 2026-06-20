---
trigger: always_on
description: Shopify Admin & Storefront GraphQL APIs via curl, with Bankr bridges for identity, x402 checkout, and webhook-driven onchain settlement.
---


# Shopify — Admin & Storefront GraphQL APIs (Bankr Edition)

> The core Shopify sections of this skill are adapted from the upstream [`NousResearch/hermes-agent`](https://github.com/NousResearch/hermes-agent/blob/main/optional-skills/productivity/shopify/SKILL.md) skill (MIT, `author: community`). The four "Bankr bridge" sections at the end are new and specific to the Bankr ecosystem.

Work with Shopify stores directly through `curl`: list products, manage inventory, pull orders, update customers, read metafields. No SDK, no app framework — just the GraphQL endpoint and a custom-app access token. Then bridge the merchant data to onchain primitives via Bankr.

The REST Admin API is legacy since 2024-04 and only receives security fixes. **Use GraphQL Admin** for all admin work. Use **Storefront GraphQL** for read-only customer-facing queries (products, collections, cart).

## Prerequisites

1. In Shopify admin: **Settings → Apps and sales channels → Develop apps → Create an app**.
2. Click **Configure Admin API scopes**, select what you need (examples below), save.
3. **Install app** → the Admin API access token appears ONCE. Copy it immediately — Shopify will never show it again. Tokens start with `shpat_`.
4. Save to your env file:
   ```
   SHOPIFY_ACCESS_TOKEN=shpat_xxxxxxxxxxxxxxxxxxxx
   SHOPIFY_STORE_DOMAIN=my-store.myshopify.com
   SHOPIFY_API_VERSION=2026-01
   BANKR_API_KEY=...   # only for the Bankr bridge sections
   ```

> **Heads up:** As of January 1, 2026, new "legacy custom apps" created in the Shopify admin are gone. New setups should use the **Dev Dashboard** (`shopify.dev/docs/apps/build/dev-dashboard`). Existing admin-created apps keep working. If the user's shop has no existing custom app and it's after 2026-01-01, direct them to Dev Dashboard instead of the admin flow.

Common scopes by task:
- Products / collections: `read_products`, `write_products`
- Inventory: `read_inventory`, `write_inventory`, `read_locations`
- Orders: `read_orders`, `write_orders` (30 most recent without `read_all_orders`)
- Customers: `read_customers`, `write_customers`
- Draft orders: `read_draft_orders`, `write_draft_orders`
- Fulfillments: `read_fulfillments`, `write_fulfillments`
- Metafields / metaobjects: covered by the matching resource scopes

## API Basics

- **Endpoint:** `https://$SHOPIFY_STORE_DOMAIN/admin/api/$SHOPIFY_API_VERSION/graphql.json`
- **Auth header:** `X-Shopify-Access-Token: $SHOPIFY_ACCESS_TOKEN` (NOT `Authorization: Bearer`)
- **Method:** always `POST`, always `Content-Type: application/json`, body is `{"query": "...", "variables": {...}}`
- **HTTP 200 does not mean success.** GraphQL returns errors in a top-level `errors` array and per-field `userErrors`. Always check both.
- **IDs are GID strings:** `gid://shopify/Product/10079467700516`, `gid://shopify/Variant/...`, `gid://shopify/Order/...`. Pass these verbatim — don't strip the prefix.
- **Rate limit:** calculated via query cost (leaky bucket). Each response has `extensions.cost` with `requestedQueryCost`, `actualQueryCost`, `throttleStatus.{currentlyAvailable, maximumAvailable, restoreRate}`. Back off when `currentlyAvailable` drops below your next query's cost. Standard shops = 100 points bucket, 50/s restore; Plus = 1000/100.

Base curl pattern (reusable):

```bash
shop_gql() {
  local query="$1"
  local variables="${2:-{}}"
  curl -sS -X POST \
    "https://${SHOPIFY_STORE_DOMAIN}/admin/api/${SHOPIFY_API_VERSION:-2026-01}/graphql.json" \
    -H "Content-Type: application/json" \
    -H "X-Shopify-Access-Token: ${SHOPIFY_ACCESS_TOKEN}" \
    --data "$(jq -nc --arg q "$query" --argjson v "$variables" '{query: $q, variables: $v}')"
}
```

Pipe through `jq` for readable output. `-sS` keeps errors visible but hides the progress bar.

## Discovery

### Shop info + current API version
```bash
shop_gql '{ shop { name myshopifyDomain primaryDomain { url } currencyCode plan { displayName } } }' | jq
```

### List all supported API versions
```bash
shop_gql '{ publicApiVersions { handle supported } }' | jq '.data.publicApiVersions[] | select(.supported)'
```

## Products

### Search products (first 20 matching query)
```bash
shop_gql '
query($q: String!) {
  products(first: 20, query: $q) {
    edges { node { id title handle status totalInventory variants(first: 5) { edges { node { id sku price inventoryQuantity } } } } }
    pageInfo { hasNextPage endCursor }
  }
}' '{"q":"hoodie status:active"}' | jq
```

Query syntax supports `title:`, `sku:`, `vendor:`, `product_type:`, `status:active`, `tag:`, `created_at:>2025-01-01`. Full grammar: https://shopify.dev/docs/api/usage/search-syntax

### Paginate products (cursor)
```bash
shop_gql '
query($cursor: String) {
  products(first: 100, after: $cursor) {
    edges { cursor node { id handle } }
    pageInfo { hasNextPage endCursor }
  }
}' '{"cursor":null}'
# subsequent calls: pass the previous endCursor
```

### Get a product with variants + metafields
```bash
shop_gql '
query($id: ID!) {
  product(id: $id) {
    id title handle descriptionHtml tags status
    variants(first: 20) { edges { node { id sku price compareAtPrice inventoryQuantity selectedOptions { name value } } } }

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BankrBot/bankr-skill-shopify](https://github.com/BankrBot/bankr-skill-shopify) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
