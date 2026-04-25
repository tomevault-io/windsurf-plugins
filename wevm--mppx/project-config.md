---
trigger: always_on
description: TypeScript implementation of the "Payment" HTTP Authentication Scheme (402 Protocol).
---

# mppx

TypeScript implementation of the "Payment" HTTP Authentication Scheme (402 Protocol).

## Vision

mppx provides abstractions for the complete HTTP 402 payment flow — both client and server. The architecture has two layers:

### Core Abstractions

1. **`Mppx`** — Top-level payment handler. Groups related `Method`s and handles the HTTP 402 flow (challenge/credential parsing, header serialization, verification).

2. **`Method`** — A payment method definition. Each method has a `name` (e.g., `charge`, `session`), a `method` (e.g., `tempo`, `stripe`), and schemas for request validation and credential payloads.

```
┌────────────────────┐       ┌────────────────┐
│       Mppx         │ 1   * │     Method     │
│    (handler)       ├───────┤  (definition)  │
└────────────────────┘ has   └────────────────┘
│ payment            │       │ tempo/charge   │
│                    │       │ tempo/session  │
│                    │       │ stripe/charge  │
└────────────────────┘       └────────────────┘
```

```
Client (Mppx)                                       Server (Mppx)
   │                                                   │
   │  (1) GET /resource                                │
   ├──────────────────────────────────────────────────>│
   │                                                   │
   │             (2) handler.charge(request, { ... })  │
   │                   402 + WWW-Authenticate: Payment │
   │<──────────────────────────────────────────────────┤
   │                                                   │
   │  (3) handler.createCredential(response)           │
   │                                                   │
   │  (4) GET /resource                                │
   │      Authorization: Payment <credential>          │
   ├──────────────────────────────────────────────────>│
   │                                                   │
   │               (5) handler.charge(request)         │
   │                                                   │
   │               (6) 200 OK                          │
   │                   Payment-Receipt: <receipt>      │
   │<──────────────────────────────────────────────────┤
   │                                                   │
```

### Primitives

Low-level data structures that compose into the core abstractions:

- **`Challenge`** — Server-issued payment request (appears in `WWW-Authenticate` header). Contains `id`, `realm`, `method`, `intent`, `request`, and optional `expires`/`digest`.
- **`Credential`** — Client-submitted payment proof (appears in `Authorization` header). Contains `challenge` echo, `payload` (method-specific proof), and optional `source` (payer identity).
- **`Method`** — Payment method definition (e.g., `tempo/charge`, `stripe/charge`). Contains `method`, `name`, and validated `schema` (credential payload + request).
- **`Mppx`** — Top-level payment handler. Groups related `Method`s and handles the HTTP 402 flow.
- **`Receipt`** — Server-issued settlement confirmation (appears in `Payment-Receipt` header). Contains `status`, `method`, `timestamp`, and `reference`.
- **`Request`** — Method-specific payment parameters (e.g., `amount`, `currency`, `recipient`). Validated by the method's schema and serialized in the challenge.

### Method Architecture

Methods are flat structural types with `method`, `name`, and `schema`:

```ts
const tempoCharge = Method.from({
  method: 'tempo',
  name: 'charge',
  schema: {
    credential: {
      payload: z.object({ signature: z.string(), type: z.literal('transaction') }),
    },
    request: z.pipe(
      z.object({
        amount: z.amount(),
        chainId: z.optional(z.number()),
        currency: z.string(),
        decimals: z.number(),
        recipient: z.optional(z.string()),
        // ...
      }),
      z.transform(({ amount, decimals, chainId, ... }) => ({
        amount: parseUnits(amount, decimals).toString(),
        ...(chainId !== undefined ? { methodDetails: { chainId } } : {}),
      })),
    ),
  },
})
```

Methods are extended with client or server logic via `Method.toClient()` and `Method.toServer()`:

```ts
// Client-side: adds credential creation
const client = Method.toClient(Methods.charge, {
  async createCredential({ challenge }) { ... },
})

// Server-side: adds verification
const server = Method.toServer(Methods.charge, {
  async verify({ credential }) { ... },
})
```

## Spec Reference

Canonical specs live at [tempoxyz/payment-auth-spec](https://github.com/tempoxyz/payment-auth-spec).

### Spec Documents

| Layer         | Spec                                                                                                                                        | Description                                                             |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------- |
| **Core**      | [draft-httpauth-payment-00](https://github.com/tempoxyz/payment-auth-spec/blob/main/specs/core/draft-httpauth-payment-00.md)                | 402 flow, `WWW-Authenticate`/`Authorization` headers, `Payment-Receipt` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wevm/mppx](https://github.com/wevm/mppx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
