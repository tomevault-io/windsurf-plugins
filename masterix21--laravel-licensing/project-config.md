---
trigger: always_on
description: - **You are** a senior Laravel developer specialized in **application security**.
---


# CLAUDE.md

## Role & working style
- **You are** a senior Laravel developer specialized in **application security**.
- **Mission**: deliver `laravel-licensing` with strong security-by-default, clear extensibility, and **offline-capable** verification.
- **Style**: concise, pragmatic, document decisions.
- **Definition of Done (DoD)**: tests green, security requirements met, CLI stable, docs updated.

---

## Project overview
Licensing package for Laravel 12/13 (PHP 8.3–8.5) with:
- **Polymorphic assignment** (`License → licensable`) to bind a license to any application model.
- **Activation keys** (128-bit entropy, hex format), **expirations/renewals**, and seat control via **LicenseUsage** (usage = one seat).
- **Offline verification** using public-key–signed tokens and a **two-level key hierarchy** (**root → signing**) for safe rotation & compromise handling.
- **License Scopes** for multi-product/software key isolation.
- **Trial management** with HMAC-SHA256 fingerprint hashing and conversion tracking.
- **Template-based licensing** with scope linkage.
- **Out of scope**: multi-tenant isolation, billing/invoicing, advanced entitlement management (hook points only).

---

## Architecture (high level)
- **Domain models** (overridable via config/contracts): `License`, `LicenseUsage`, `LicenseRenewal`.
- **Policies**: over-limit handling, grace periods, inactivity auto-revocation.
- **Crypto**: Ed25519 (default) for signatures; root CA issues short-lived signing keys; tokens carry `kid` and a chain to root.
- **Interfaces (contracts)** to allow project-level replacement: KeyStore, CertificateAuthority, TokenIssuer/Verifier, UsageRegistrar, FingerprintResolver, Notifier.
- **CLI** for key lifecycle (make/issue/rotate/revoke/export) and offline token issuance.
- **API (optional)** for validate/refresh/jwks/usages.
- **Jobs/Scheduler** for state transitions and notifications.

---

## Core entities (conceptual)
### License
- Fields: `id(ULID)`, `key_hash`, `status[pending|active|grace|expired|suspended|cancelled]`, `activated_at`, `expires_at`, `max_usages`, `meta(json)`.
- Relations: `licensable (morphTo)`, `usages (hasMany)`, `renewals (hasMany)`.
- Indexes: `expires_at`, `status`, `key_hash (unique)`, `licensable_type+licensable_id`.

### LicenseUsage
- Meaning: one **seat consumption** (device/VM/service/user/session).
- Fields: `id`, `license_id`, `usage_fingerprint`, `status[active|revoked]`, `registered_at`, `last_seen_at`, `revoked_at`, `client_type`, `name`, `ip?`, `user_agent?`, `meta(json)`.
- Uniqueness (default): **per-license** → unique(`license_id`,`usage_fingerprint`). Configurable to **global**.
- Indexes: unique composite above, plus `revoked_at`, `last_seen_at`.

### LicenseRenewal
- Fields: `id`, `license_id`, `period_start`, `period_end`, `amount_cents?`, `currency?`, `notes?`.

---

## States & lifecycle
- States: `pending → active → grace → expired`; side paths: `suspended`, `cancelled`.
- Events: `LicenseActivated`, `LicenseExpiringSoon`, `LicenseExpired`, `LicenseRenewed`, `UsageRegistered`, `UsageRevoked`, `UsageLimitReached`.
- Scheduler (daily): time-based transitions, emit notifications/webhooks.

---

## Configuration (defaults are secure & overridable)
`config/licensing.php` keys:
- `models`: `license`, `license_usage`, `license_renewal` (class names to override).
- `morph_map`: aliases for `licensable` types (hide app class names).
- `policies`:
  - `over_limit`: `reject` (default) | `auto_replace_oldest`.
  - `grace_days`: `14`.
  - `usage_inactivity_auto_revoke_days`: `null` (off) or integer.
  - `unique_usage_scope`: `license` (default) | `global`.
- `offline_token`:
  - `enabled`: `true`.
  - `format`: `paseto` (default) | `jws`.
  - `ttl_days`: `7`.
  - `force_online_after_days`: `14`.
  - `clock_skew_seconds`: `60`.
- `crypto`:
  - `algorithm`: `ed25519` (default) | `ES256`.
  - `keystore.driver`: `files` | `database` | `custom`.
  - `keystore.path`: `storage/app/licensing/keys`.
  - `keystore.passphrase_env`: `LICENSING_KEY_PASSPHRASE`.
- `publishing`:
  - `jwks_url`: nullable (for JWS clients).
  - `public_bundle_path`: path to bundle with root public and chain (PEM/JSON).
- `rate_limit`:
  - `validate_per_minute`: `60`.
  - `token_per_minute`: `20`.
  - `register_per_minute`: `30`.
- `notifications`: toggle per event; choose mail/queue/webhook via Notifier contract.
- `audit`:
  - `enabled`: `true`.
  - `store`: `database` | `file`.

All timestamps stored in **UTC**.

---

## Security requirements
- Activation keys: store **HMAC-SHA256 hash** only; constant-time comparisons; 128-bit entropy keys generated with `random_bytes()`.
- License key format: `PREFIX-XXXXXXXX-XXXXXXXX-XXXXXXXX-XXXXXXXX` (8 hex chars per segment).
- Offline tokens: signed only; client holds **public** material; no private keys client-side.
- Two-level hierarchy:
  - **Root (pub/priv)** = trust anchor, **never** signs tokens; used to sign **signing-key certificates**.
  - **Signing key (pub/priv)** = signs offline tokens; short-lived; includes `kid`; distributed with a **chain** up to root.
  - KIDs generated with `bin2hex(random_bytes(16))` — cryptographically unpredictable.
- Rotation & compromise:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/masterix21) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
