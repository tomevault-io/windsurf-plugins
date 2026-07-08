---
trigger: always_on
description: Member Amenities module specifics — apply when touching amenities_* tables, supabase/functions/amenities-*, or amenities/attendant/leaderboard/post-health/floor-integrity frontend.
---


# Member Amenities module — implementation conventions

Source of truth: `Member Amenities System - PRD.md` Part B (B.1–B.8) and the Codebase Alignment Review. Where the conceptual sections (§7.2–§7.3) differ from Part B, **Part B wins**.

## Architecture spine
- **Mirror the member list; never live-call FlexWash per activation.** Every activation resolves against the server-side mirror `amenities_member`. The post never sees the CSV or the API.
- **Phase 1 (now, Margate pilot):** bulk mirror is loaded by **`amenities-import-members`** from the FlexWash "memberships On" CSV export (`id, name, email, phoneNumber, vehicles`). Sample: `data/FlexWash Member Export - import format sample (memberships On).csv`. **Presence in the file = active.** No status/expiry columns → `valid_until = null`.
- **Phase 2 (later):** `amenities-sync-members` pulls the read-only members API (only once FlexWash ships a higher rate limit). Reuses the same parse→normalize→guard→merge core.
- **Merge is key-preserving, keyed on `flexwash_customer_code`** (the stable identity anchor; `phone_e164` is a unique *lookup*, not identity). **Upsert, never delete-and-reinsert.** Never touch `amenities_member_tag` / `amenities_vehicle_tag` rows where `source='rt_os'`. RT OS is the system of record for all preference tags; tags are **not** written back to FlexWash.
- **Fail-safe:** stage first, then refuse the merge and keep the prior mirror if the file/pull is empty, implausibly small (< 50% of `prev_active_count`, a Settings threshold), or error-heavy. Deactivate only by absence in a *successful, sanity-checked* refresh. Log every load to `amenities_member_import`.

## Identity model (distinct on purpose)
- Floor attendants exist **only** as `amenities_attendant` rows (hashed punch code), **never** Supabase-auth users, **never** Connecteam users. "Logged in" == an open `amenities_attendant_session`.
- The leaderboard's "who was working" denominator is `amenities_attendant_session` **alone** — do **not** join `flexwash_employees`.
- Punch-code hashing happens **server-side** via `amenities-attendant-admin`; `punch_code_hash` is never sent to or stored in the browser and is not SELECT-able under RLS.
- Position (First/Second/Third) is **derived at runtime** from punch-in order among open sessions, not stored. Stall ownership comes from `amenities_lot_map` for the current headcount. `max_attendants` is a per-location Setting (default 3).

## Net-new patterns (no codebase precedent — build deliberately, design-review required)
1. **Per-device auth (B.7).** Device holds a per-device bearer key (not the anon key), scoped to `amenities-activate`, `amenities-member-cache`, `amenities-events-sync`, `amenities-heartbeat`, `amenities-help`. Key shown **once** at provisioning; store only `amenities_device.device_key_hash`. **Every** device-facing function checks `amenities_device.revoked` on every call. Shape = **validate-bearer-then-service-role-write** (same as the HMAC webhook). The device speaks raw HTTPS `POST /functions/v1/<fn>` — not `functions.invoke`.
2. **Tablet token & board reads — RESOLVED (June 2026), supersedes PRD §B.5 native Realtime on Lovable Cloud.** The PRD proposed Supabase Realtime on `amenities_service_card` + a Supabase-accepted custom tablet JWT (location-claim RLS keyed on `auth.jwt() ->> 'location_id'`). **This is NOT achievable on Lovable Cloud** (the project JWT secret is neither injected nor readable, custom-JWKS/third-party-issuer registration is not exposed, and floor attendants must never be Supabase-auth users — see `40-supabase-lovable-cloud-migrations.mdc` §12). So **do NOT** `ALTER PUBLICATION supabase_realtime ADD TABLE …` for the tablet and **do NOT** add tablet RLS keyed on a Supabase-verified JWT. The settled architecture:
   - **Our own token, our own verifier.** `amenities-attendant-session` mints an HS256 token with **`AMENITIES_TABLET_JWT_SECRET`** (a value WE control — *not* the project secret; project-prefixed because `SUPABASE_*` is reserved). Mint/verify both live in **`_shared/attendantAuth.ts`** (`mintTabletJwt` / `verifyTabletToken`). Claims: `att_role:'amenities_tablet'`, `location_id`, `session_id`, `attendant_id`; `role`/`aud` are `'authenticated'` but vestigial (Supabase never verifies this token).
   - **Board via session-validated edge function + polling.** `amenities-board` verifies the token → confirms the session is still open / not idle → service-role read scoped to the token's `location_id`; `amenities-serve` is the write. The dashboard uses React Query `refetchInterval` polling (the PRD's "correct on polling alone" path, here the **primary** mechanism). **Reads never extend the session; only attributed writes (serve/welcome/help) bump `last_activity_at`.**
   - **Shared floor logic in `_shared/`.** Punch-in ranking + lot-map ownership (`rankPositions`/`positionLabel`/`buildOwnership`) live in **`_shared/floorMap.ts`** so both `amenities-attendant-session` and `amenities-board` import them — never cross-import one function's `_lib.ts` from another (it fails at deploy; §12).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tderi33/spec-compliant-kit](https://github.com/tderi33/spec-compliant-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
