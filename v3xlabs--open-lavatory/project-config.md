---
trigger: always_on
description: When sources conflict, follow this precedence:
---

# Repository Guidelines

## Source-of-Truth Order
When sources conflict, follow this precedence:

1. `packages/*/src` implementation
2. Package export surfaces (`packages/*/package.json` `exports` fields)
3. Root tooling config (`biome.json`, `eslint.config.mjs`, `tsconfig.base.json`)
4. Docs (`docs/`), especially `docs/specs/` for protocol intent
5. Examples (`examples/`) for usage patterns only, not strict style baseline

Notes:
- `docs/pages/api/*` is partially incomplete (`TODO` stubs).
- `docs/dist/` is generated output and should not be edited as source.
- `docs/specs/001_initial.md` is historical design intent from early development; canonical runtime behavior is defined by current `packages/*/src` implementation.

## Monorepo Topology
Workspace is a TypeScript monorepo (`pnpm`). Primary package graph:

- `@openlv/core`: core types/utilities (URL encoding, encryption, errors, shared types)
- `@openlv/signaling`: signaling abstraction + providers (`mqtt`, `ntfy`, `gundb`, dynamic loader)
- `@openlv/transport`: transport abstraction + `webrtc`
- `@openlv/session`: connection lifecycle orchestrator combining signaling + transport
- `@openlv/provider`: EIP-1193-style provider over sessions + persisted settings
- `@openlv/modal`: Preact modal + custom element wrapper for wallet UX
- `@openlv/connector`: Wagmi connector integrating provider + modal
- `@openlv/react-native`: React Native compatibility layer/polyfills around session APIs
- `@openlv/extension`: WIP browser extension implementation

Engineering baseline for style and architecture:
- Primary baseline: `packages/core`, `packages/signaling`, `packages/transport`, `packages/session`, `packages/provider`, `packages/connector`, `packages/modal`, `packages/react-native`
- Lower-confidence baseline (WIP/demo): `packages/extension`, `examples/`

## Protocol Model (Implementation-Oriented)
OpenLV establishes dApp-wallet JSON-RPC connectivity in phases:

1. Session creation
- dApp creates session parameters (`sessionId`, signaling protocol/server, handshake key material)
- URI generated via `encodeConnectionURL`

2. Signaling handshake
- Wallet parses URI (`decodeConnectionURL`)
- Peers discover each other over signaling layer
- Symmetric handshake + asymmetric key exchange occurs via signaling messages

3. Transport setup
- After signaling reaches encrypted state, transport (`webrtc`) is set up
- Offer/answer/candidate exchange flows through signaling path
- Transport transitions to connected and is used for payload transfer

4. JSON-RPC request/response
- Provider/session send request objects and await matched responses by message id

Important implementation details:
- Signaling protocols currently implemented: `mqtt`, `ntfy`, `gun`
- Transport currently implemented: `webrtc`
- Runtime protocol selection: `dynamicSignalingLayer(protocol)`

## Protocol/State Invariants
Use and preserve existing state/event conventions:

- State constants modeled as `as const` objects + derived union types
  - `SIGNAL_STATE`, `TRANSPORT_STATE`, `SESSION_STATE`, `PROVIDER_STATUS`
- State transition notifications emitted via typed event maps (`state_change`)
- Cross-layer communication uses `eventemitter3` with typed events
- Public API returns plain object contracts (factory-composed), not class-heavy APIs

Session lifecycle semantics:
- `createSession(...).connect()` initializes signaling and transport
- `waitForLink()` resolves when session reaches connected state
- `send(message, timeout)` sends request and resolves with correlated response payload

## URI and Handshake Notes
Documented intent exists in `docs/specs/001_initial.md` and `docs/pages/api/uri.mdx`; implementation is authoritative.

Current implementation characteristics (`packages/core/src/url/index.ts`):
- Scheme: `openlv://`
- Version host: `@1`
- Required params: `h`, `k`
- Optional params: `p`, `s`
- Session id validation: 16 URL-safe characters
- `h` validation: 16 lowercase hex chars
- `k` validation: 32 lowercase hex chars (implementation reality)

If editing URI behavior:
- Update `encodeConnectionURL` and `decodeConnectionURL` together
- Update/add tests in `packages/core/src/url/index.test.ts`
- Reconcile docs if behavior changes

## Build/Test/Lint Commands
Run from repository root unless package-scoped.

- `pnpm build`: recursive package build (root filters out docs/examples/extension)
- `pnpm test`: recursive tests (root filters out tests workspace + extension)
- `pnpm lint`
- `pnpm lint:fix`
- `pnpm changeset` for publishable package changes

Useful package-scoped execution:
- `pnpm --filter @openlv/<package> build`
- `pnpm --filter @openlv/<package> test`

## Coding Standards
Baseline rules carried from prior project guidance and aligned to this repo.

### Architectural style
- Prefer function-oriented and factory-oriented design
- Avoid classes except narrow runtime integration points
- Prefer explicit composition (`createX`, helper combinators) over inheritance
- Keep module boundaries explicit between signaling/transport/session/provider layers

### TypeScript style
- Prefer `type` over `interface` in general
- Keep types close to usage; avoid central type dump files
- Prefer `const fn = (...) => {}` for most function definitions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [v3xlabs/open-lavatory](https://github.com/v3xlabs/open-lavatory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
