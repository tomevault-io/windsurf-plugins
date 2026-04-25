---
trigger: always_on
description: Home Assistant HACS integration providing scoped guest access to door locks and garage covers via an iOS companion app.
---

# CLAUDE.md — HA Easy Control

Home Assistant HACS integration providing scoped guest access to door locks and garage covers via an iOS companion app.

- **Domain:** `easy_control` | **Version:** 0.4.0 | **Python:** 3.12
- **Runtime dep:** `segno==1.6.6` (QR generation, lazy-imported in `api.py`)
- **iOS companion app repo:** `../ha-easy-control` (Swift/SwiftUI, iOS 16+, MVVM)
- **Deep link scheme:** `easy-control://pair`

## How It Works

This is a **HACS custom component** that pairs with an iOS app to give guests scoped, time-limited access to specific Home Assistant entities (door locks, garage covers) without giving them a full HA account.

### End-to-End Flow
1. **Admin** calls `create_guest_pass` service → creates `PairingRecord` (in-memory, 5-min TTL) + QR deep link
2. **QR code** is shown via persistent notification or `/api/easy_control/qr` endpoint (non-consuming, repeatable until scan)
3. **iOS app** scans QR → calls `POST /pair/scanned` to acknowledge scan (one-time, disables QR display)
4. **iOS app** calls `POST /pair` with `pairing_code`, `device_id`, `device_public_key` → receives scoped JWT
5. **iOS app** requests nonce (`GET /action/nonce`) then executes action (`POST /action`) with optional Ed25519 proof
6. **Action mapping:** `door.open` → `lock.unlock`, `garage.open` → `cover.open_cover`

### QR Payload
Actions and configuration are transmitted to the iOS app via QR code containing a deep link:
```
easy-control://pair?pairing_code=...&base_url=...&entity_id=...&allowed_action=...&scan_ack_token=...
```

## Module Map

```
custom_components/easy_control/
  __init__.py          Setup: registers API views + services, reads config entry options
  api.py               6 HTTP views (all HomeAssistantView, requires_auth=False for public endpoints)
  config_flow.py       Single-instance config flow + options flow
  const.py             All CONF_*, DATA_*, DEFAULT_*, EVENT_* constants
  network.py           RFC1918 CIDR validation (is_remote_allowed, normalize_allowed_cidrs)
  pairing.py           PairingRecord dataclass + PairingStore (in-memory dict, auto-purge)
  proof.py             Ed25519 action proof: parse, verify, canonical signing input
  runtime_security.py  ActionNonceStore (single-use nonces) + FixedWindowRateLimiter
  services.py          5 HA service handlers (create/revoke/approve/reject)
  storage.py           HA Store persistence: signing keyring, token metadata, revocations
  token.py             GuestTokenManager: create/verify JWT-like tokens, GuestTokenPayload
  services.yaml        Service field definitions for HA UI
  strings.json         UI localization keys
```

## Security Model

- **JWT-like tokens** (HS256) with rotating signing keyring (`storage.py`)
- **Token claims:** `iss`, `aud` (localkey_ios), `jti`, `guest_id`, `entity_id`, `allowed_action`, `iat`, `nbf`, `exp`, `max_uses`, `token_version`, `device_id`, `cnf.jkt`
- **Multi-use tokens:** `max_uses=0` means unlimited until expiry/revocation; finite values enforce use count
- **Optional Ed25519 device binding** + proof of possession (nonce + signed action proof)
- **Revocation:** per-token JTI, per-guest_id, or global `token_version` increment
- **Rate limiting:** fixed-window per endpoint bucket (pair/action/qr/nonce)
- **Network policy:** optional RFC1918 local-only restriction
- **Constant-time comparison:** `hmac.compare_digest` for all token/code checks
- **Thread safety:** `asyncio.Lock` via `_get_storage_lock()` for storage mutations

## Allowed Entities & Actions

- `lock.*` entities → `door.open` action → `lock.unlock` service
- `cover.*` entities → `garage.open` action → `cover.open_cover` service

## Key Data Flow

- `hass.data[DOMAIN]` holds: `PairingStore`, `ActionNonceStore`, `FixedWindowRateLimiter`, per-entry config
- Persistent state via HA Store (key `easy_control.keys`): signing keyring, active_kid, token_version, use counts, revoked JTIs, issued token metadata
- Pairing store is **in-memory only** (not persisted); records auto-purge on 5-min expiry

## Action Proof Canonical Format

The Ed25519 proof uses a **newline-delimited** canonical signing input (NOT JSON-signed):
```
METHOD\npath\nts\nnonce\njti\ndevice_id\nbody_sha256
```
- Source of truth: `proof.py:build_proof_signing_input()`
- `body_sha256` is **hex-encoded** (not base64url)
- Headers: `X-Easy-Control-Proof` (base64url JSON), `X-Easy-Control-Proof-Signature` (base64url Ed25519 sig)

## API Response Contracts

**`POST /pair`** (pairing exchange):
```json
{"guest_token", "allowed_actions", "expires_at", "guest_id", "max_uses",
 "proof_required", "device_binding_required", "nonce_endpoint", "scan_ack_supported"}
```

**`POST /pair/scanned`** (scan acknowledge):
```json
{"success": true, "status": "acknowledged"|"already_acknowledged",
 "pairing_code", "qr_scanned_at"}
```

**`POST /action`** (action execution):
```json
{"success": true, "action", "entity_id", "remaining_uses", "used_count"}
```
`remaining_uses=-1` signals unlimited (`max_uses=0`)

**`POST /token/validate`** (token validation):
```json
{"guest_id", "allowed_actions", "entity_id", "expires_at", "remaining_uses"}
```

## iOS Companion App

- **Repo:** `../ha-easy-control`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dvselas) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
