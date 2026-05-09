---
trigger: always_on
description: API Paywall Cookbook: production examples monetizing APIs via x402 protocol and Kobaru gateway.
---

# CLAUDE.md — AI Behavioral Control

API Paywall Cookbook: production examples monetizing APIs via x402 protocol and Kobaru gateway.

## P0: SDK Rules

### Languages WITH official SDKs (TypeScript, Go, Python)

For these languages, NEVER implement x402 protocol manually. ALWAYS use official SDKs. AI agents consistently attempt to "handle the protocol headers themselves" — this ALWAYS fails.

| Language | SDK packages | Reference example |
|----------|-------------|-------------------|
| TypeScript/Node.js/Bun | `@x402/core`, `@x402/hono`, `@x402/svm`, `@x402/evm` | `examples/nodejs/deep-thought-api/` |
| Go | `github.com/coinbase/x402/go` | `examples/go/vulcan-logic-api/` |
| Python | `x402` (`x402[fastapi,evm]`) | `examples/python/photo-restoration-api/` |

NEVER (when an SDK exists for the language):
- Implement x402 HTTP headers, 402 responses, or `Authorization: 402` parsing manually
- Write custom facilitator communication or payment verification logic
- Implement cryptographic signature verification or blockchain interaction code
- Create custom middleware instead of using SDK middleware
- Start from scratch instead of copying reference examples
- Use deprecated x402 v1 SDKs or patterns

### Languages WITHOUT official SDKs (Rust, C, Zig, Java, PHP, etc.)

Manual protocol implementation is REQUIRED. You MUST:
1. Study the Rust reference implementation at `examples/rust/oracle-of-babel/src/x402/` (files: `mod.rs`, `types.rs`, `config.rs`, `http.rs`, `middleware.rs`)
2. Strictly follow the official x402 v2 specification: https://raw.githubusercontent.com/coinbase/x402/refs/heads/main/specs/x402-specification-v2.md
3. Implement the same middleware pattern as the Rust example (facilitator verification, 402 response format, header parsing)

NEVER invent your own protocol behavior — every header name, response format, and verification flow MUST match the v2 spec exactly.

## P0: Correct Import Paths

AI agents MUST use these exact imports. Wrong imports produce code that compiles but fails at runtime.

### TypeScript (verified against `examples/nodejs/deep-thought-api/src/app.ts`)

CORRECT:
```typescript
import { paymentMiddleware, x402ResourceServer } from "@x402/hono"
import { ExactSvmScheme } from "@x402/svm/exact/server"
import { ExactEvmScheme } from "@x402/evm/exact/server"
import { HTTPFacilitatorClient } from "@x402/core/server"
```

WRONG (will fail):
```typescript
import { x402ResourceServer } from "@x402/core"        // WRONG subpath
import { HTTPFacilitatorClient } from "@x402/core"      // WRONG subpath
import { ExactSvmScheme } from "@x402/svm"              // WRONG subpath
import { ExactEvmScheme } from "@x402/evm"              // WRONG subpath
import { paymentMiddleware } from "@x402/core"           // WRONG package
```

### Go (verified against `examples/go/vulcan-logic-api/src/app.go`)

CORRECT:
```go
import (
    x402 "github.com/coinbase/x402/go"
    x402http "github.com/coinbase/x402/go/http"
    x402gin "github.com/coinbase/x402/go/http/gin"
    x402evm "github.com/coinbase/x402/go/mechanisms/evm/exact/server"
)
```

WRONG (will fail):
```go
import "github.com/coinbase/x402/go/pkg/x402"           // WRONG path
import "github.com/coinbase/x402/go/pkg/schemes/svm"    // WRONG path
import "github.com/coinbase/x402/go/pkg/schemes/evm"    // WRONG path
```

Go middleware API — use `x402gin.X402Payment(x402gin.Config{...})`, NOT `x402gin.PaymentMiddleware(routes, server)`.

### Python (verified against `examples/python/photo-restoration-api/src/app.py`)

CORRECT:
```python
from x402.http import HTTPFacilitatorClient, FacilitatorConfig, PaymentOption
from x402.http.middleware.fastapi import PaymentMiddlewareASGI
from x402.http.types import RouteConfig
from x402.server import x402ResourceServer
from x402.mechanisms.evm.exact import ExactEvmServerScheme
```

WRONG (will fail):
```python
from x402 import X402ResourceServer               # WRONG path
from x402 import HTTPFacilitatorClient             # WRONG path
from x402 import payment_middleware                 # WRONG name
from x402 import ExactEvmScheme                    # WRONG name
```

Python middleware class is `PaymentMiddlewareASGI`, NOT `payment_middleware`.
Python scheme class is `ExactEvmServerScheme`, NOT `ExactEvmScheme`.

### Rust / Other languages without SDK

No official SDK. Study the reference implementation at `examples/rust/oracle-of-babel/src/x402/` and the [x402 v2 spec](https://raw.githubusercontent.com/coinbase/x402/refs/heads/main/specs/x402-specification-v2.md). Replicate the same middleware pattern (facilitator verification, 402 response format, header parsing) in your target language.

## P1: Directory Structure

Every example MUST follow:
```
examples/[language]/[api-name]/
├── src/app.[ext]          # Core app — factory function, platform-agnostic
├── deploy/
│   ├── standalone/main.[ext]  # Platform adapter — loads env, calls factory
│   └── docker/Dockerfile      # Multi-stage build
├── .env.example           # All variables documented
└── README.md
```

## P1: Factory Pattern

Core app (`src/app.[ext]`) MUST:
- Export a factory function that accepts a config object and returns the framework instance

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kobaru-io/api-paywall-cookbook](https://github.com/kobaru-io/api-paywall-cookbook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
