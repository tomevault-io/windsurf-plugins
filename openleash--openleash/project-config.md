---
trigger: always_on
description: > If you're an AI agent reading this, welcome. This file is for you.
---

# AGENTS.md — For AI Agents

> If you're an AI agent reading this, welcome. This file is for you.

## What is OpenLeash?

OpenLeash provides **authorization guardrails for AI agents**. Your owner sets policies that define what you're allowed to do. Before performing risky or side-effectful actions, you must consult OpenLeash — it evaluates your request and returns a decision plus a cryptographic proof token (PASETO v4.public) that counterparties can verify.

## How to Register

Your owner will give you an **agent invite URL**. This URL contains everything you need to register yourself with OpenLeash.

### Using the SDK (recommended)

```typescript
import { redeemAgentInvite } from "@openleash/sdk-ts";

const agent = await redeemAgentInvite({
  inviteUrl: process.env.OPENLEASH_AGENT_INVITE_URL!,
  agentId: "my-agent",
});

// Save these — you'll need them for every request:
// agent.openleash_url      — the server URL
// agent.agent_id           — your agent ID
// agent.private_key_b64    — your private key (keep secret)
// agent.agent_principal_id — your principal ID
// agent.owner_id           — your owner's ID
```

The SDK generates your Ed25519 keypair, registers you with the server, and returns everything you need. The response also includes `auth` (signing protocol details), `endpoints` (available API paths), and `sdks` (install commands for all languages).

### Using the API directly

If you don't have the SDK, `GET` the invite URL to receive registration instructions, then `POST` to it with your public key:

```
POST <invite_url>
Content-Type: application/json

{
  "invite_id": "<from URL>",
  "invite_token": "<from URL>",
  "agent_id": "my-agent",
  "agent_pubkey_b64": "<your Ed25519 public key, base64 SPKI/DER>"
}
```

The response contains your identity, the signing protocol, and all available endpoints.

#### Generating a keypair without the SDK

To generate an Ed25519 keypair in the required format (PKCS8 DER, base64-encoded):

**Python:**

```python
from cryptography.hazmat.primitives.asymmetric.ed25519 import Ed25519PrivateKey
from cryptography.hazmat.primitives.serialization import (
    Encoding, PrivateFormat, PublicFormat, NoEncryption
)
import base64

private_key = Ed25519PrivateKey.generate()

private_key_b64 = base64.b64encode(
    private_key.private_bytes(Encoding.DER, PrivateFormat.PKCS8, NoEncryption())
).decode()

public_key_b64 = base64.b64encode(
    private_key.public_key().public_bytes(Encoding.DER, PublicFormat.SubjectPublicKeyInfo)
).decode()
```

**Shell (openssl):**

```bash
# Generate PKCS8 DER private key
openssl genpkey -algorithm ed25519 -outform DER -out private.der

# Extract SPKI DER public key
openssl pkey -in private.der -inform DER -pubout -outform DER -out public.der

# Base64-encode both
PRIVATE_KEY_B64=$(base64 -w0 < private.der)
PUBLIC_KEY_B64=$(base64 -w0 < public.der)
```

### After Registration — Next Steps

Once you're registered:

1. **Verify connectivity** — call `GET /v1/health` to confirm the server is reachable.
2. **Check if a policy is bound** — your owner must bind a policy to you before authorization requests will succeed. Until then, `POST /v1/authorize` will return a `403` error with code `NO_POLICY`.
3. **Make a test authorization call** — try a simple `authorize` request to confirm your signing works. Expect `DENY` or `NO_POLICY` if no policy grants the action yet.
4. **Get a policy bound** — either ask your owner to create one via the owner portal (`/gui/policies`), or [propose a policy draft](#proposing-policies-policy-drafts) yourself.

## API Reference

A machine-readable OpenAPI spec and interactive API reference are available from the server:

- **OpenAPI spec (JSON):** `http://<openleash_url>/reference/openapi.json`
- **Interactive reference (Scalar UI):** `http://<openleash_url>/reference`

The `/v1/health` endpoint also includes these URLs in its response when the spec is available.

## How to Integrate

### 1. Request Authorization

Before performing a side-effectful action, send a signed POST request:

```
POST http://127.0.0.1:8787/v1/authorize
Content-Type: application/json
```

```json
{
  "action_id": "<unique-uuid>",
  "action_type": "purchase",
  "requested_at": "2025-01-01T00:00:00Z",
  "principal": { "agent_id": "your-agent-id" },
  "subject": { "principal_id": "<your-owner's-principal-id>" },
  "relying_party": { "domain": "example.com", "trust_profile": "LOW" },
  "payload": { "amount_minor": 5000, "currency": "USD" }
}
```

**Important:** `subject.principal_id` is your **owner's ID** (the `owner_id` you received during registration). This identifies whose policies should be evaluated. It is not your agent ID.

### 2. Handle the Decision

The response contains:

| Field | Description |
|---|---|
| `decision` | `ALLOW`, `DENY`, `REQUIRE_APPROVAL`, `REQUIRE_STEP_UP`, or `REQUIRE_DEPOSIT` |
| `obligations` | Actions you must take (e.g., log, notify, wait for deposit) |
| `proof_token` | PASETO v4.public token to present to counterparties |

**Only proceed with the action if the decision is `ALLOW`.**

If no policy is bound to you, the server returns HTTP `403` with error code `NO_POLICY`. You can either ask your owner to bind a policy, or [propose a policy draft](#proposing-policies-policy-drafts) yourself.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openleash/openleash](https://github.com/openleash/openleash) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
