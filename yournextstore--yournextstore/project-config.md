---
trigger: always_on
description: Your Next Store — e-commerce app built with Next.js App Router + Commerce Kit SDK.
---

# AGENTS.md

Your Next Store — e-commerce app built with Next.js App Router + Commerce Kit SDK.

## Commands

```bash
bun dev           # Dev server (port 3000)
bun run build     # Production build
bun start         # Production server
bun run lint      # Biome lint (--write to auto-fix)
bun run format    # Biome format
bun test          # Run tests (bun:test)
tsgo --noEmit     # Type check
```

## Key Files & Directories

```
app/                  # Pages, layouts, actions (App Router)
components/ui/        # Shadcn UI components (50+)
lib/commerce.ts       # Commerce API client
lib/money.ts          # Currency formatting (formatMoney)
lib/utils.ts          # Utilities
biome.json            # Lint/format config
next.config.mjs       # Next.js config
hooks/                # Custom React hooks
```

## Project Patterns

- Use `safe-try` for error handling: `const [error, result] = await safe(...)`
- Format prices with `formatMoney` from `lib/money.ts`
- Use functional array methods (`map`, `filter`, `reduce`), not loops
- No `any` types; rely on type inference; minimal return type annotations
- **Always quote paths** with special characters in shell commands: `rg "term" "app/(auth)/login"`
- **ALL `/checkout` links MUST be plain `<a>` tags.** Never use `<Link>` (or any link wrapper) for `/checkout` links — they 500.

## Biome Rules

Avoid: default exports, `any`, `for...of`, `forEach` for mutations, missing hook deps, unnecessary type annotations, function names ending with "Action" (unless server action).

Default export exceptions (Biome-allowed): `page.tsx`, `layout.tsx`, `loading.tsx`, `error.tsx`, `not-found.tsx`.

Prefer: named exports, `map`/`filter`/`reduce`, type inference, `as const`, template literals.

## Commerce Kit SDK

```tsx
// Product browsing
const products = await commerce.productBrowse({
  active: true, limit: 12, offset: 0,
  // search: "query", category: "id", tags: ["tag"]
});

// Product details (accepts ID or slug)
const product = await commerce.productGet({ idOrSlug: productId });
// product.variants[].{id, price (minor units string), stock, images, attributes}

// Cart
const cart = await commerce.cartUpsert({ cartId, variantId: "v-123", quantity: 1 });
const cart = await commerce.cartGet({ cartId });
```

## Code Examples

### Page with caching
```tsx
// app/search/page.tsx
import { commerce } from "@/lib/commerce";
import { SearchResults } from "./search-results";

export default async function SearchPage({
  searchParams,
}: {
  searchParams: Promise<{ q?: string }>;
}) {
  "use cache";
  const { q } = await searchParams;
  const products = q
    ? await commerce.productBrowse({ search: q, active: true })
    : { data: [] };
  return <SearchResults products={products.data} query={q} />;
}
```

### Error handling
```tsx
import { commerce } from "@/lib/commerce";
import { formatMoney } from "@/lib/money";
import { safe } from "safe-try";

const [error, result] = await safe(
  commerce.productGet({ idOrSlug: productId })
);
if (error || !result) {
  return <div>Product not found</div>;
}
const price = formatMoney({
  amount: result.variants[0].price,
  currency: "USD",
  locale: "en-US",
});
```

### Unit test (bun:test)
```typescript
import { test, expect } from "bun:test";
import { formatMoney } from "@/lib/money";

test("formatMoney handles USD correctly", () => {
  const result = formatMoney({ amount: "1999", currency: "USD", locale: "en-US" });
  expect(result).toBe("$19.99");
});
```

## Validation Checklist

- [ ] `tsgo --noEmit` — no type errors
- [ ] `bun run lint` — no lint errors
- [ ] `bun run format` — code formatted
- [ ] `bun test` — tests pass
- [ ] `bun run build` — build succeeds
- [ ] `bun dev` — runs without errors, feature works in browser
- [ ] No console errors, images load, responsive layout
- [ ] No hardcoded secrets; env vars set (`.env.local` / Vercel dashboard)

Required env: `YNS_API_KEY`

## Troubleshooting

| Error | Cause | Fix |
|-------|-------|-----|
| `Cannot read property 'variants' of undefined` | Product data missing | Use optional chaining (`product?.variants`) |
| `Missing env.YNS_API_KEY` | Env not loaded | Create `.env.local`, restart dev server |
| `noDefaultExport` | Default export in non-special file | Use named export |
| `BigInt literal syntax` | Using `0n` with ES2020 | Use `BigInt(0)` |

## Agent Workflow Notes

- **Explore agent**: Start with `lib/commerce.ts`, `app/layout.tsx`, `app/page.tsx`. Search `"use server"`/`"use cache"` for patterns.
- **Plan agent**: Check existing code first. Map to: routes (`app/`), API (`lib/commerce.ts`), UI (`components/ui/`), actions (`actions.ts`). Consider caching and server vs client components.
- **Implementation agent**: Validate with commands above before and after changes. Follow Biome rules, reuse existing UI components.
- **Frontend design**: Use `frontend-design:frontend-design` skill to achieve a distinctive, production-grade frontend experiences.

**When starting work on the project, ALWAYS call the `init` tool from `next-devtools-mcp` FIRST to set up proper context and establish documentation requirements. Do this automatically without being asked.**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yournextstore/yournextstore](https://github.com/yournextstore/yournextstore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
