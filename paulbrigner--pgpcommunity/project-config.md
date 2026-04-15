---
trigger: always_on
description: - Project: Next.js 15 app deployed via AWS Amplify; auth is NextAuth (email + SIWE) with DynamoDB adapter, membership is Unlock Protocol on Base, UI uses shadcn + Tailwind CLI.
---

# Copilot Instructions

- Project: Next.js 15 app deployed via AWS Amplify; auth is NextAuth (email + SIWE) with DynamoDB adapter, membership is Unlock Protocol on Base, UI uses shadcn + Tailwind CLI.
- Front door: Home screen is client-only in [app/home-client.tsx](app/home-client.tsx); it orchestrates membership/NFT queries, Unlock checkouts, and Base network prompts via React Query + `useUnlockCheckout`. SSR no longer prefetches membership/NFTs in `app/page.tsx`; it only passes session membership status/expiry to avoid reload delays.
- Membership model: Source of truth is on-chain locks from `NEXT_PUBLIC_LOCK_TIERS`; state is derived/cached in [lib/membership-state-service.ts](lib/membership-state-service.ts) and normalized by `snapshotToMembershipSummary` in [lib/membership-server.ts](lib/membership-server.ts). Prefer client-side React Query caching for Home; invalidate via `/api/membership/invalidate` after checkout.
- Checkout targets: Lock/tier metadata come from [lib/checkout-config.ts](lib/checkout-config.ts) fed by env vars (`NEXT_PUBLIC_LOCK_TIERS`, optional recurring/referrer settings). Event locks are discovered on-chain, and `getMembershipCheckoutTarget` / `getEventCheckoutTarget` should be used instead of hardcoding addresses.
- Wallet flows: Keep users on Base (see network switch helper in home-client) before approvals; cap USDC approvals to 12 months and reject unlimited caps (`SAFE_ALLOWANCE_CAP`). Prefer `extendKey` over `purchaseKey` when a tokenId exists.
- Token IDs: Subgraph preferred (`UNLOCK_SUBGRAPH_URL` or `UNLOCK_SUBGRAPH_API_KEY` + `UNLOCK_SUBGRAPH_ID`); fallback enumerates `tokenOfOwnerByIndex` with small caps. If no tokenId is resolvable, block renewal to avoid max-keys reverts.
- Membership caching: In-memory service caches per addresses+chain for ~3m with variants for allowances/tokenIds; `forceRefresh` or `invalidate` when side effects occur. Queries should scope by normalized, comma-joined addresses.
- Auto-renew UX: Allowance prompts live in home-client; approvals use computed target months (defaults to 12) and hide prompts when allowance is present or non-renewable/never-expires tiers are selected.
- NFTs/events: `/api/nfts` aggregates owned/missed/upcoming creator NFTs on Base, merging Alchemy metadata when available; Home loads NFTs client-side via `useMemberNfts` and relies on React Query cache (Profile page may prefetch into the cache).
- Event details: Upcoming meetings link to `/events/[lockAddress]`, which fetches metadata from `GET /api/events/details?lockAddress=` and handles sponsored RSVP with Unlock checkout fallback for paid events.
- Event token metadata (Option C): Public tokenURI responses are served by `GET /api/events/metadata/{lockAddress}/{tokenId}` using `EVENT_METADATA_TABLE`. Set each event lock's base tokenURI to `https://<site>/api/events/metadata/<lockAddress>/` via `scripts/setup/set-event-token-uri.mjs`.
- Auth/session: NextAuth v4 in `app/api/auth/[...nextauth]/route.ts` (email magic link via SES SMTP + SIWE credentials). Session is JWT enriched with profile fields and membership snapshots; wallet linking uses `/api/auth/link-wallet` and `/api/auth/unlink-wallet`.
- Signup flow: Wallet-first signup persists `{ email, wallet }` via `/api/signup/pending`, then links on email verification in NextAuth callback; legacy email-first flow still redirects to `/signin` with banners.
- Profile data: Stored via NextAuth in DynamoDB; fields include first/last name, xHandle, linkedinUrl. Settings/Profile UI lets users link/unlink wallets.
- Admin surface: Admin pages under [app/admin](app/admin) and components in [components/admin](components/admin) handle roster views, refunds, emails, and cache resets; ensure routes verify auth/membership and planned admin role flag.
- Admin roster cache: Uses a separate DynamoDB cache table (`ADMIN_ROSTER_CACHE_TABLE`) with meta + paged roster items and a short-lived lock. `GET /api/admin/members` supports `preferStale=1` and `triggerRebuild=1` to return stale data quickly and kick off rebuilds; `GET /api/admin/members/status` returns cache metadata for polling. `/admin` no longer builds the roster on SSR to avoid Amplify timeouts. A standalone SAM Lambda in `infra/admin-roster-rebuild` now handles background rebuilds (scheduled + manual), triggered via `/api/admin/members/rebuild` using `ADMIN_ROSTER_REBUILD_URL` + `ADMIN_ROSTER_REBUILD_SECRET`.
- Admin roster data/UI: Member "joined" date is derived from the lowest (Member) tier lock via Unlock subgraph `createdAtBlock` + Base RPC block timestamps when building the cached roster. The Admin member table now filters by tier level and sorts by last name/joined/expiry; wallets/balances/email details are hidden behind per-member expand toggles with manual detail refresh.
- Admin test members: User records may include `isTestMember` (boolean). Admin actions can toggle this flag; test members are hidden from the roster by default and only shown when the "Test members" filter is selected.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/paulbrigner) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
