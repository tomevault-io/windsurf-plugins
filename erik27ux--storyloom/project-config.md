---
trigger: always_on
description: These must be triggered manually at the right moments. Remind the user proactively.
---

# Storyloom — Claude Code Reference

## ⚠️ Security Checkpoints — DO NOT FORGET
These must be triggered manually at the right moments. Remind the user proactively.

1. **Before testing with real users / soft launch** → re-run full security review (`/security-review` or manual file audit). Confirm the Supabase SQL migration (`supabase_security_migration.sql`) has been run in production.
2. **Before adding RevenueCat / any payment system** → full security review with focus on: subscription tier write paths, webhook authentication, receipt validation, and ensuring `updateSubscriptionTier()` in AuthManager is replaced by a backend webhook (never called from client in production).
3. **Before App Store submission** → remove `#if DEBUG` dev tier overrides in `AuthManager.swift` (`devTierOverrides` dict), replace App Store placeholder ID `id000000000` in `join.html` and `index.html`, and run security review.
4. **After any Supabase schema change** → re-verify RLS policies cover the new tables/columns.
5. **When Apple Developer Program enrollment is complete ($99/year)** → upgrade push notifications from local to APNs:
   - Enable Push Notifications capability in Xcode → Signing & Capabilities (one checkbox)
   - Download an APNs Auth Key (.p8) from Apple Developer portal → Keys section
   - Run the Supabase DB migration to add `push_token text` column to `profiles` table (SQL in `NotificationManager.swift` comments)
   - Write a Supabase Edge Function that triggers on DB inserts (comments, questions, story publishes) and sends APNs pushes using the .p8 key
   - The `NotificationManager.uploadTokenToSupabase()` and all client-side code is already complete — no app changes needed beyond enabling the capability

---

## 🚀 Pre-Launch Checklist — Step-by-Step

### PHASE 1 — Apple Developer Program ($99/yr)
**When:** As soon as enrolled at developer.apple.com

**Xcode — one-time setup:**
- [ ] Add **Associated Domains** capability → `applinks:storyloom.live`
  - (AASA file already deployed at `storyloom.live/.well-known/apple-app-site-association` — no web change needed)
  - ⚠️ **Removed for simulator testing** — personal teams don't support this capability. Re-add it once enrolled in Apple Developer Program.
- [ ] Add **Push Notifications** capability (one checkbox in Signing & Capabilities)

**Apple Developer portal:**
- [ ] Download an **APNs Auth Key** (.p8) → Keys section of developer.apple.com
  - Save it somewhere safe — it cannot be re-downloaded

**Supabase — run this SQL migration:**
```sql
ALTER TABLE profiles ADD COLUMN push_token text;
```
  - (Full SQL in `NotificationManager.swift` comments)

**Supabase — write a new Edge Function:**
- Trigger: on INSERT into `story_entries`, `story_questions` (answer updates), or `comments`
- Action: call APNs using the .p8 key to send a push to the relevant user's `push_token`
- The client-side code (`NotificationManager.uploadTokenToSupabase()`) is already complete — no app changes needed

---

### PHASE 2 — RevenueCat Integration
**When:** Adding paid subscription tiers

**Security review first** — run `/security-review` with focus on:
- Subscription tier write paths
- Webhook authentication
- Receipt validation

**Files to update:**
- `AuthManager.swift` — find `updateSubscriptionTier()`:
  - Currently called from client-side (OK for dev/testing)
  - **Must be replaced by a RevenueCat webhook** before shipping — client must NEVER be able to write its own subscription tier in production
- `AuthManager.swift` — find `devTierOverrides` dict:
  - Already gated with `#if DEBUG` — ensure it stays that way until explicitly removed pre-launch

**RevenueCat setup steps:**
- [ ] Create RevenueCat project, link to App Store Connect
- [ ] Add RevenueCat SDK to Xcode project
- [ ] Configure entitlements/products in RevenueCat dashboard matching existing tier names (`free`, `family`)
- [ ] Write a Supabase Edge Function as the RevenueCat webhook endpoint (updates `profiles.subscription_tier` on purchase/renewal/cancellation)
- [ ] Verify webhook signature validation in the Edge Function
- [ ] Run Supabase RLS audit — confirm `subscription_tier` column is NOT writable by the user themselves (currently protected but verify after any schema changes)

---

### PHASE 3 — App Store Submission
**When:** Ready to submit

**Security review** — run `/security-review` (full pass, not just subscription paths)

**Files to update before submitting:**

1. `AuthManager.swift`
   - Remove the `devTierOverrides` dictionary entirely (or the whole `#if DEBUG` block that sets tier overrides)
   - Search for: `devTierOverrides`

2. `Storyloom web/join.html`
   - The "Coming soon to the App Store" badge needs to become a real App Store link
   - Replace the `.coming-soon` div with an `<a href="https://apps.apple.com/app/storyloom/id<REAL_ID>">` link
   - Also update the meta/OG tags if added later

3. `Storyloom web/index.html`
   - Same App Store link replacement as `join.html`

4. `Storyloom web/privacy.html`
   - Verify "Last updated" date is current
   - Add any new data processors introduced by RevenueCat

**App Store Connect setup:**
- [ ] Set Privacy Policy URL → `https://storyloom.live/privacy`
- [ ] Set Support URL → `https://storyloom.live` (or a contact page)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Erik27UX/StoryLoom](https://github.com/Erik27UX/StoryLoom) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
